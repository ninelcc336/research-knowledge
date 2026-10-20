---
title: "IndexTTS、Qwen3-TTS 与 VoxCPM2 对比调研"
date: 2026-10-20
category: "AI语音"
tags:
  - 语音合成
  - 声音克隆
  - 音色设计
  - 多语言
  - 流式生成
  - 情绪控制
  - 模型部署
  - 选型对比
---

# IndexTTS、Qwen3-TTS 与 VoxCPM2 对比调研

调研日期：2026-10-20。研究对象为 bilibili IndexTTS、阿里 Qwen3-TTS、OpenBMB VoxCPM2，比较声音克隆、情绪控制、音色设计、多语言、流式生成与本地部署。

## 1. 先看结论

**中文短视频、角色对白和需要反复调整情绪的配音，优先试 IndexTTS-2.5；通用播报、预设音色和语音服务，优先试 Qwen3-TTS；需要更多语言、48kHz 输出，以及在同一模型里完成音色设计和克隆，重点试 VoxCPM2。** 这是按功能与部署条件给出的试用顺序，不是三者音质的绝对排名。

三者都是把文字生成语音的模型，也都能做零样本声音克隆。它们不直接完成视频口型同步、字幕强制对齐、完整播客编排或多角色时间线剪辑；这些仍需要上层程序。

本次最影响判断的发现是：

- **IndexTTS 的当前主版本已经是 2.5。** 它于 2026-08-10 官宣，GitHub `v2.5.0` Release 发布于 2026-08-13，新增多语言、原生语速控制和更快推理。不能只用 IndexTTS-2 的旧资料评价整个项目。[S1][S2]
- **Qwen3-TTS 是一组分工不同的模型。** `Base` 负责参考音频克隆；`CustomVoice` 使用预设音色；`VoiceDesign` 根据文字设计新音色。不能下载任意一个权重就期待获得全部功能。[S6]
- **VoxCPM2 的可控克隆和高保真克隆存在取舍。** 前者能用文字改变表达风格；后者主要延续参考语音的节奏和情绪，官方明确说明该模式忽略控制指令。[S13]
- **“支持流式”必须具体到实现。** Qwen 基础 Python 接口仍返回完整波形；当前 vLLM-Omni 已有在线 PCM 分块与 WebSocket。IndexTTS 当前 vLLM 路径标为非分块响应，不能因为接受 `stream=true` 就当作低延迟音频分块生成。[S7][S17]
- **长文本仍应分句或分段。** 仓库已有 VoxCPM2 调研中的长文本问题，截至本次核查仍未被上游宣布彻底解决；2026 年 9 月的一项独立研究也观察到了 Qwen3-TTS 和 VoxCPM2 随文本变长而出现的漏读、幻觉和音色失稳。[S15][S20]

本次没有下载大模型、执行本机语音生成或进行听音盲测。下面的指标均注明资料出处；安装与 API 示例属于文档、源码核对结果，不是本机性能承诺。

## 2. 版本基线与仓库已有信息

### 2.1 本次核查的版本

| 项目 | 当前比较对象 | 发布与维护快照 | 本次读取的主分支提交 |
| --- | --- | --- | --- |
| IndexTTS | IndexTTS-2.5；必要时对照 IndexTTS-2 | 2.5 于 2026-08 官宣与发布；主分支最近提交为 2026-09-29 | `d9e41aa` |
| Qwen3-TTS | 已公开的 12Hz、0.6B/1.7B 系列 | README 公告为 2026-01-22；本次主分支最近提交为 2026-03-17 | `022e286` |
| VoxCPM | VoxCPM2；最新 GitHub Release 为 `2.0.3` | 2.0.3 发布于 2026-05-11；主分支最近提交为 2026-10-08 | `cce58a5` |

以上来自 GitHub 当前 README、Release 和提交 API。模型版本、Python 包版本和主分支提交不是同一概念：例如 IndexTTS 当前 `pyproject.toml` 的包版本仍写 `2.0.0`，不能据此认定加载的是 IndexTTS-2 权重；Qwen 技术报告中的 25Hz/VoiceEditing 也不能直接视为已按官方清单公开下载的模型。[S1][S4][S6]

涉及在线服务时，另核查了 vLLM-Omni `main` 提交 `01f807f`，其时间为 2026-10-20。该实现更新快，文中关于在线服务的结论只适用于此次读取的文档与源码状态，不代表任意旧版 wheel 都已包含相同功能。[S17]

### 2.2 如何使用仓库旧调研

本次结合了三篇已有资料：

- [VoxCPM 项目调研报告](./2026-08-19-VoxCPM.md)：保留其中对 VoxCPM2 三种模式、长文本和微调的研究基础，并重新核查当前 README、Release、使用指南和相关 Issue。语言数、采样率与基础性能参考仍能对应当前官方说明。
- [MiniMax T2A HTTP 语音合成 API 接入与计费调研](./2026-10-10-MiniMax-T2A-HTTP-API.md)：作为托管 API 的另一条路线。本文三款本地权重的许可与运行成本，不能套用 MiniMax 网页或 API 的额度规则。
- [AI混剪视频与口型、语气、语速同步开源方案调研](../AI内容创作/2026-09-04-AI混剪视频与口型语气同步开源方案.md)：沿用“语音、时间对齐、口型、剪辑分开处理”的工程思路。

原有调研正文保持原样，新结论集中记录在本文。

## 3. 核心能力对比

| 维度 | IndexTTS-2.5 | Qwen3-TTS 12Hz 系列 | VoxCPM2 |
| --- | --- | --- | --- |
| 开发团队 | bilibili Index Speech Team | 阿里 Qwen 团队 | OpenBMB |
| 模型规模口径 | 约 0.8B GPT backbone | 0.6B / 1.7B | 2B backbone |
| 官方支持语言 | 5 种：中、英、日、西、阿 | 10 种：中、英、日、韩、德、法、俄、葡、西、意 | 30 种，另列中文方言能力 |
| 原生输出采样率 | 22.05kHz | 24kHz | 48kHz |
| 参考音频克隆 | 支持，公开基础接口不要求填写参考转写 | `Base` 支持；完整 ICL 克隆使用参考音频和转写 | 可控克隆不需要转写；高保真续写需要转写 |
| 无参考音频设计新音色 | 当前官方主流程依赖说话人参考音频 | 独立的 1.7B `VoiceDesign` 模型 | 同一模型直接使用文字描述 |
| 预设音色 | 官方主接口没有内置预设音色体系 | `CustomVoice` 有 9 个预设音色 | 主打设计与参考克隆 |
| 情绪控制 | 独立情绪参考音频、8 维向量、文字转情绪 | 1.7B `CustomVoice`、`VoiceDesign` 支持指令；`Base` 不承诺同样的指令控制 | 音色设计、可控克隆支持括号指令；高保真模式忽略指令 |
| 语速控制 | 原生 `duration_factor`，0.5–2.0 | 指令与参考风格影响语速；服务层 `speed` 依部署实现，流式有约束 | 自然语言控制或继承参考节奏 |
| 显式发音控制 | 中文拼音、英文 CMU 音素、日文假名 | 未在本次官方已发布接口中发现等价的音素标注承诺 | 未在本次官方接口中发现等价的音素标注承诺 |
| 微调 | 主仓库不是一套完整官方训练教程；社区另有方案 | 官方提供单说话人 SFT 示例 | 官方提供 SFT / LoRA 工作流 |
| 许可 | bilibili 自定义模型许可 | Apache-2.0 | Apache-2.0 |

来源：[S1][S3][S6][S7][S9][S11][S13][S17]。参数量不是整套服务的显存占用，部分项目表格只计算 backbone；采样率也不能单独代表自然度、噪声水平或音色还原能力。

## 4. IndexTTS：适合明确控制表演的配音

### 4.1 2.5 相对 2 的实际变化

IndexTTS 原本就强调把说话人音色与情绪分开。2.5 延续这一方向，主要增加多语言、降低语义序列开销、改进声学生成与发音控制，并开放原生语速参数。对中文视频创作者，比较有价值的是能够保留角色声音，同时独立试验情绪和快慢，而不必重新寻找一段情绪、音色都合适的参考音频。[S1][S5]

当前 `duration_factor > 1` 会生成更长、更慢的语音，`< 1` 会生成更短、更快的语音。它是模型内部的时长缩放，不是最终 WAV 的播放变速；vLLM API 的 `speed` 方向相反，通过 `duration_factor = 1 / speed` 映射。[S1][S17]

**语速可控不等于能把任意台词精确压进指定秒数。** IndexTTS-2 的公告仍注明精确时长控制功能未在该版本开放；2.5 公开的倍率也不是“目标时长为 4.000 秒”的硬约束。用于镜头配音时仍要根据实际音频时长调整文案、停顿或倍率，再做时间对齐。[S1]

### 4.2 情绪与发音控制的特点

主要有三种情绪来源：独立情绪音频、8 维情绪向量、文字情绪描述。8 维顺序为 `happy, angry, sad, afraid, disgusted, melancholic, surprised, calm`；`emo_alpha` 调整情绪影响强度。

2.5 用文字推断情绪时，应在构造模型时显式启用 `use_qwen_emo=True`，再使用 `use_emo_text=True`。不启用前者会报错。这里的 Qwen 情绪解释组件是 IndexTTS 的辅助组件，不能与 Qwen3-TTS 语音生成模型混为一谈。[S1]

拼音、CMU 音素和日文假名控制适合处理多音字、人名、品牌名和需要固定读法的文本。这是它与另外两款相比比较明确的工作流优势，但也应检查具体词表与生成结果，不能假定任意标注都有效。

### 4.3 边界与部署差异

当前标准实现有 `stream_return` 分段返回路径，但它不等于一套已经验证的双向低延迟交互服务。vLLM-Omni 当前总表把 IndexTTS-2/2.5 标为 `stream=true response, non-chunk`，本报告不据此推荐其作为最低首包延迟的语音助手方案。[S3][S17]

vLLM-Omni 当前 IndexTTS-2.5 路径采用普通采样，不复现官方默认 `num_beams=3` 的 beam search；服务端文档建议比较时让上游也使用 `num_beams=1`。因此，迁移引擎后需要重新检查声音质量，不能只测速度。[S17]

主分支 2026-09-29 的更新在最终拼接音频末尾添加淡出以缓解结束时的咔哒声；提交说明同时指出它不覆盖流式最终拼接之外的路径，也没有解决所有尾部高频噪声。不能把该更新理解成所有音频异常都已修复。

## 5. Qwen3-TTS：先选对权重，再讨论功能

### 5.1 五个已发布模型如何选择

| 权重后缀 | 主要用途 | 选择建议 |
| --- | --- | --- |
| `12Hz-1.7B-CustomVoice` | 9 个预设音色，加自然语言风格指令 | 无需克隆、需要固定播报音色时先试 |
| `12Hz-0.6B-CustomVoice` | 9 个预设音色 | 资源较紧时试；源码会忽略该小模型的 `instruct` |
| `12Hz-1.7B-VoiceDesign` | 根据文字设计音色和表达风格 | 需要原创角色声音时试 |
| `12Hz-1.7B-Base` | 参考音频声音克隆 | 对目标说话人做完整克隆验证 |
| `12Hz-0.6B-Base` | 较小的参考克隆模型 | 与 1.7B 对照资源、速度和质量 |

`CustomVoice` 的 “Custom” 容易让人误解为“上传任意声音”。它实际提供 Vivian、Serena、Uncle_Fu、Dylan、Eric、Ryan、Aiden、Ono_Anna、Sohee 等预设说话人；上传参考音频克隆应选 `Base`。[S6]

型号中的 12Hz 描述语音 token 速率这一层级，实际 tokenizer 为 12.5Hz；它与最终波形的 24kHz 采样率是不同概念。[S8]

`Base` 默认 ICL 模式使用参考音频与准确转写。开启 `x_vector_only_mode=True` 可以不填转写，但官方说明克隆质量可能降低。`Base` 在已发布功能表中没有指令控制勾选，不能把 `CustomVoice` 的情绪指令能力原样承诺给克隆模式。[S6][S7]

### 5.2 固定原创角色声音的实用流程

官方提供了“Voice Design then Clone”路线：先用 `VoiceDesign` 生成满意的短样音，再把样音及其转写交给 `Base` 创建 `voice_clone_prompt`，后续复用该 prompt 生成台词。

这比每句都重新描述同一个角色更适合持续创作，也能避免反复计算参考特征。但它没有消除克隆模式的表达限制；设计阶段的风格控制，不等于后续每条 Base 克隆请求都能接受同样的独立情绪控制。[S6]

### 5.3 流式能力要分三层

1. **模型与论文层**：12Hz 系列为流式而设计，论文给出优化后的低首包延迟。
2. **官方 `qwen-tts` Python 包层**：本次源码中 `generate_voice_clone`、`generate_voice_design`、`generate_custom_voice` 返回整段波形。`non_streaming_mode=False` 的注释明确说明目前是在模拟流式文本条件，不会因此启用真正的流式输入或流式生成。
3. **vLLM-Omni 服务层**：当前实现已支持 `/v1/audio/speech` 的 PCM 分块输出与 `/v1/audio/speech/stream` WebSocket。Qwen 主 README 中“目前仅离线推理”的文字已经落后于该部署项目的当前状态。[S6][S7][S17]

当前 vLLM-Omni 的 PCM 流式需要 `stream_format="audio"`、`response_format="pcm"` 和启用 `async_chunk`，且流式不支持 `speed`。WebSocket 默认缓存文本，在 `input.done` 后以连续请求合成；想在整条输入结束前逐句合成，需要选择 sentence 分割方式。它仍不能简单等同于任意字符一到达就立即生成声音。[S17]

### 5.4 微调与已知反馈

官方训练示例当前围绕单说话人 SFT，JSONL 包含 `audio`、`text`、`ref_audio`；建议所有样本使用同一参考音频。它不是官方已经完成的多说话人、跨平台完整训练产品。[S9]

截至本次，Issue #341 仍开放，报告部分 ICL 克隆会先复述参考音频末尾内容，再进入目标台词；Issue #371 报告微调损失与推理对齐方面的疑点。这些是社区问题报告，本次未独立复现，不能当作每个模型、每次生成必然发生的错误；但足以把“开头串入参考台词”和“微调后试听回归”加入验收项目。[S10]

## 6. VoxCPM2：同一模型覆盖设计、可控克隆和续写

VoxCPM2 使用连续语音潜变量和扩散自回归生成，官方称 tokenizer-free。这里指避免传统离散音频码本作为核心语音生成接口，不是没有文本 tokenizer，也不是没有任何内部量化结构。实际选型价值主要在于同一个模型提供三种声音生成方式，而不在于这一术语本身。[S11]

三种模式应分开试：

- **Voice Design**：在文本前用括号描述年龄、性别、音色、情绪等，不需要参考音频。
- **Controllable Cloning**：使用 `reference_wav_path` 提取音色，并以括号指令控制语速、情绪和表达，无需参考转写。
- **Ultimate / Hi-Fi Cloning**：提供 `prompt_wav_path` 与准确的 `prompt_text`，需要时同时提供独立 reference；侧重保留参考的音色、节奏和风格。官方说明此模式忽略括号控制指令。[S13]

如果目标是“同一个人，今天平静、下一句愤怒”，应优先测试可控克隆。如果目标是“尽量像这段参考录音的原始声音和讲述方式”，再测试高保真模式。不能同时把最高音色相似度与任意风格重写当作必然可得。

官方支持 30 种语言，包含另外两款未统一覆盖的泰语、越南语、印尼语等。不同语言质量并不一致；语言数是覆盖范围，不是每种语言相同的质量保证。原生 48kHz 适合高采样率音频链路，但不能因此推断它必然比 22.05kHz 或 24kHz 的模型更自然。[S11]

当前 Python API 已有 `generate_streaming()`；2.0.3 Release 优化了流式 VAE 解码，并改进训练清单校验、可选 `ref_audio`、设备处理与 LoRA 加载。当前主分支还包含之后的界面、部署和训练相关更新，所以固定 2.0.3 版本与直接安装主分支不能视为完全相同状态。[S11][S12]

官方承认可控生成在不同运行之间可能有差别，并建议必要时生成 1–3 次选择结果。连续内容生产应复用经过筛选的参考声音或缓存，不宜每句都重新随机设计音色。[S11]

## 7. 效果和速度：能从现有证据得出什么

### 7.1 同一张官方比较表里的三者

IndexTTS-2.5 当前 README 给出 CV3-Eval 零样本比较，以下只摘录中文和英文，保留该表的 WER/SS 命名与数值：

| 模型 | 中文 WER↓ | 中文 SS↑ | 英文 WER↓ | 英文 SS↑ |
| --- | ---: | ---: | ---: | ---: |
| IndexTTS-2.5 | 4.36 | 77.10 | 5.12 | 68.06 |
| Qwen3-TTS 1.7B | 3.27 | 73.02 | 5.06 | 67.17 |
| VoxCPM2 | 3.88 | 74.99 | 5.13 | 71.57 |

这张表支持的有限结论是：**在 Index 团队公布的这一评测口径下，Qwen 中文文字准确性更好，Index 中文说话人相似度更高，VoxCPM2 英文说话人相似度更高。** 三者各有优势，没有全面胜出者。SS 是说话人相似度，不是主观自然度评分。[S1]

该表由模型开发团队公布，不能视为完全独立的统一盲测；Qwen 行也没有在表名中进一步明确 Base 子型号。Index README 另有 `IndexTTS2.5-RL` 行，本文未将其指标替代普通 `IndexTeam/IndexTTS-2.5` 权重的结果。

仓库旧 VoxCPM 调研引用的是另一套 Seed-TTS-eval 等结果，测试集、转写器和推理设置不同。不能从各项目 README 各挑一个最好数字拼成总榜，也不能用同名基准下不同团队的数值计算精确提升率。

### 7.2 官方性能数字与口径

| 项目 | 官方参考数据 | 必须同时知道的条件 |
| --- | --- | --- |
| IndexTTS-2.5 | RTX 4090、BF16、`kv_cache=True`，整体 RTF 0.2065；对应 IndexTTS-2 FP16 为 0.3257 | 同仓库参考测试；不是并发 API 的首包延迟 |
| Qwen3-TTS 12Hz | 0.6B 首包 97ms、RTF 0.288；1.7B 首包 101ms、RTF 0.313 | 论文内部 vLLM V0 引擎、并发 1，并优化 tokenizer 解码；不是普通 Python 包的本机成绩 |
| VoxCPM2 | RTX 4090 标准 PyTorch RTF 约 0.30；Nano-vLLM 路径约 0.13 | 官方配置与引擎不同；未据此统一推导首包和高并发能力 |

来源：[S1][S8][S11]。RTF 是生成耗时除以音频时长，越低越快。RTF 0.2 表示按该测试口径，10 秒音频约需 2 秒生成；它没有说明第一个音频块何时能播放。

这里尤其不能把 Qwen 的 97ms 误写成“1.7B 在普通消费卡上的实际首包”，也不能把 Vox 的加速引擎与 Index 标准实现直接排序。真实服务还涉及冷启动、参考音频处理、队列、并发、音频封装和播放器缓存。

### 7.3 2026 年 9 月新增的长文本证据

独立预印本 *Taming Long-form Text-to-Speech* 针对多个文本长度和 10 个随机种子评估 Qwen3-TTS 与 VoxCPM2，并研究 LACI 推理纠错方法。[S20]

论文报告的原始基线中，Qwen3-TTS-0.6B 在短于 500 词时的 worst-of-N WER 为 5.4%，在长于 1500 词时达到 35.2%；1.7B 在最长分桶中达到 47.8%。VoxCPM2 在至少 1300 词的分桶中，平均/最差种子 WER 为 18.1%/35.0%。这些结果依赖该文的数据和采样设置；“词”也不能直接换算为中文字符数。

上述长文本实验使用 Qwen 的 CustomVoice 模型，不能把数值直接套用到 Base 参考克隆模式。

这里的 worst-of-N 衡量多个种子中的不利结果，不能与平均 WER 混用。这篇论文没有在本次引用的实验中评测 IndexTTS-2.5，不能推断 Index 因而更稳定，也不能把 LACI 的改进当作当前官方安装包已自带的能力。本次未复现论文。

针对中文，VoxCPM Issue #372 报告约 300 字内容在生成后半段失真，涉及标准实现及加速引擎；维护者承认长文本问题尚未彻底解决。该 Issue 截至本次仍开放，10 月有下游分句缓解方案的讨论。**300 字是一个案例，不是所有机器或所有声音的统一失效阈值。** [S15]

## 8. 本地部署与 Windows 试用

### 8.1 安装路线

三个项目应使用独立环境。Index 当前依赖约束较细，直接把三者装进现有同一个 Python 环境，容易产生 PyTorch、Transformers 等版本冲突。

| 项目 | 官方起步方式 | Windows 实用建议 |
| --- | --- | --- |
| IndexTTS-2.5 | 克隆官方仓库，`uv sync`，下载 `IndexTeam/IndexTTS-2.5`，运行 WebUI | 当前 Python 约束为 `>=3.10,<3.12`；先使用 `uv sync --extra webui`，再按需要加速功能；官方提醒 Windows 的 DeepSpeed 安装可能困难 |
| Qwen3-TTS | 独立环境 `pip install -U qwen-tts`，运行 `qwen-tts-demo` 或 Python API | 官方推荐 Python 3.12；FlashAttention 2 是加速选项，不能把它当作所有 Windows 机器的必备条件 |
| VoxCPM2 | 独立环境 `pip install voxcpm`，使用 Python API；源码另有 WebUI | 官方要求 Python `>=3.10,<3.13`；先验证普通推理，必要时关闭 `optimize` 排查编译问题 |

来源：[S1][S4][S6][S11]。Index 的 README 推荐通过 `uv` 锁定依赖；本表给出的仅 WebUI extra 是减少初次安装组件的选择，不等价于启用所有加速。

Index 当前 vLLM recipe 列出 vLLM 0.27.0+、vLLM-Omni nightly，并以 H200 环境验证；还注明低显存配置没有被验证。不能把这一生产 recipe 直接当成 8GB Windows 显卡的一键安装教程。[S18]

### 8.2 显存如何判断

VoxCPM2 官方模型表标注约 8GB 基础推理显存。本次未找到能覆盖 Index、Qwen 全部功能与部署方式的统一官方最低显存数字，因此不把社区整合包所写的“4GB/5GB/6GB 可用”升级成通用结论。

如果现有显存有限，可以先试 Qwen 0.6B、较短文本和单路生成；这只是验证顺序。长参考音频、额外情绪模型、KV cache、声码器、编译和并发都会改变峰值占用。准备采购时，应先用目标权重和真实文本测峰值显存与延迟，再定硬件。

VoxCPM 新近 Issue #419 报告，在有 Triton 但没有 C 编译器的 Linux runtime 镜像中，默认 `optimize=True` 会在首次生成时失败；报告者给出的临时路径为补充编译器或 `optimize=False`。这是特定环境反馈，本次未复现，不是 Windows 上必然发生的错误。[S16]

### 8.3 三个最小 API 入口

以下分别在对应项目环境中运行，并提前准备权重与参考音频。示例用于说明功能边界，未在本机执行。

**IndexTTS-2.5：固定音色、单独描述情绪。**

```python
from indextts.infer_v2_5 import IndexTTS2

tts = IndexTTS2(
    cfg_path="checkpoints/config.yaml",
    model_dir="checkpoints",
    use_bf16=True,
    use_qwen_emo=True,
)
tts.infer(
    spk_audio_prompt="speaker.wav",
    text="原来，那个一直在等你的人，就是我。",
    lang="ZH",
    use_emo_text=True,
    emo_text="先克制地讲述，最后带一点释然。",
    emo_alpha=0.6,
    duration_factor=1.0,
    output_path="index.wav",
)
```

文字情绪会转成模型支持的情绪条件，不保证精确执行句内每个表演阶段。需要逐句不同情绪时应拆成多个请求。

**Qwen3-TTS：预设音色与风格指令。**

```python
import torch
import soundfile as sf
from qwen_tts import Qwen3TTSModel

model = Qwen3TTSModel.from_pretrained(
    "Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice",
    device_map="cuda:0",
    dtype=torch.bfloat16,
)
wavs, sr = model.generate_custom_voice(
    text="原来，那个一直在等你的人，就是我。",
    language="Chinese",
    speaker="Serena",
    instruct="语气克制、温柔，语速稍慢。",
)
sf.write("qwen.wav", wavs[0], sr)
```

需要克隆时改用 `Base` 和 `generate_voice_clone`，不能只给这个 CustomVoice 示例增加参考音频字段。

**VoxCPM2：参考音色与可控风格。**

```python
import soundfile as sf
from voxcpm import VoxCPM

model = VoxCPM.from_pretrained(
    "openbmb/VoxCPM2",
    load_denoiser=False,
    optimize=False,
)
wav = model.generate(
    text="(语气克制、温柔，语速稍慢)原来，那个一直在等你的人，就是我。",
    reference_wav_path="speaker.wav",
    cfg_value=2.0,
    inference_timesteps=10,
    seed=42,
)
sf.write("vox.wav", wav, model.tts_model.sample_rate)
```

这里关闭优化便于先验证普通路径，性能不代表官方优化配置。BF16 示例也要求硬件支持相应精度，应按实际设备调整。

## 9. 许可差异：Index 不能套用另外两款的规则

Qwen3-TTS 官方仓库、模型说明及 VoxCPM 官方仓库均声明 Apache-2.0。IndexTTS 当前仓库和 2.5 模型卡使用 **bilibili Model Use License Agreement**，不是 Apache-2.0 或 MIT。[S4][S19]

Index 当前许可存在一个具体的不一致：英文 2.2 条的年度收入门槛为 **RMB 1 billion（10 亿元）**，中文 `LICENSE_ZH.txt` 2.2 条则为 **1 亿元人民币**；两者的月活门槛都为超过 1 亿。英文第 9 条规定冲突时中文为准，因此不能仅依据英文门槛评估额外商业许可要求。

Index 的 3.4(c) 还限制用模型或衍生品改进其他 AI 模型，除 Index 自身、其衍生品及非商业 AI 模型等例外。协议对衍生品的定义涉及模型输出；如果计划把它批量生成的语音拿去训练另一个商业 TTS，这一点直接影响技术路线。普通配音使用与训练其他模型应分别阅读对应条款。

这里列的是本次读取的实际条款差异，不把“可以下载权重”概括为三者授权完全相同。

## 10. 社区资料提供了哪些补充

本次除官方资料外，还检索了 Reddit 的公开索引内容，并通过 B站搜索 API 查看中文使用场景。

Reddit 关于 IndexTTS-2.5 发布的帖子中，有用户报告英文克隆的原始口音保持不如 2.0，偏向美式口音。这是单个用户体验，不能据此认定全面退步，但说明比较 2 与 2.5 时应保留同一段带口音参考音频测试。[S21]

B站搜索能看到 IndexTTS-2.5 的配音、微调、字幕配音和 ComfyUI 整合方案，说明中文创作工具生态活跃；但“智能角色配音”“SRT 配音”“低显存”等可能来自整合包的切句、卸载或量化方案，不能都归为上游模型原生能力。示例包括 Hewkick 的评测与 T8star-Aix 的逐句情感控制工具介绍。[S22]

本次没有获取这些视频的完整字幕，也没有据此做听音判断；Reddit 资料为搜索索引抓取到的帖子内容，未完整遍历讨论。社区材料只用于发现待测试项，不作为音质排名依据。

## 11. 按用途选择与验证

### 11.1 推荐试用顺序

| 目标 | 优先候选 | 对照与关键判断 |
| --- | --- | --- |
| 中文短视频、情绪对白、角色配音 | IndexTTS-2.5 | 用 Qwen 1.7B CustomVoice 建立播报基线；重点听情绪是否自然、发音能否稳定修正 |
| 不想准备参考音频，只想固定声音播报 | Qwen 1.7B CustomVoice | 9 个预设音色最直接；若声音不合适，再用 VoiceDesign 生成角色样音 |
| 原创角色音色，之后长期复用 | Qwen VoiceDesign → Base；或 VoxCPM2 设计 → 固定参考 | 测跨句、跨批次一致性；不要每句重新随机设计 |
| 参考克隆后还要改变情绪 | IndexTTS-2.5；VoxCPM2 可控克隆 | 相似度和表达能力一起验收；不要拿 Vox 高保真模式测试指令能力 |
| 更多语言，尤其泰语、越南语等 | VoxCPM2 | 按目标语言单独测；Qwen 和 Index 当前语言清单不能直接覆盖这些需求 |
| 低延迟语音服务 | Qwen3-TTS + 当前 vLLM-Omni；VoxCPM2 流式路径 | 实测首个可播放 PCM、并发队列与打断；Index 当前 vLLM 非分块路径需区别看待 |
| 长篇文章、有声书 | 三者均采用分段与检查 | 不能根据上下文长度或一次 demo 就承诺整章稳定生成 |
| 资源受限 | 先用 Qwen 0.6B 验证 | 与实际可用的小模型/量化方案对比；不保证单凭参数小就能满足延迟 |

对于仓库近期涉及的短视频与情绪内容，建议先做 **IndexTTS-2.5 与 Qwen3-TTS-1.7B 的中文样片对照**；有多语言或高保真克隆需求时，再把 VoxCPM2 纳入同一批文本。若主要是复制特定声音，Qwen 对照项应选 `Base`，而不是拿预设 CustomVoice 与参考克隆比较。

### 11.2 一轮足够有区分度的试验

建议准备 20–30 条真实台词，包含极短句、多音字、数字日期、英文品牌、中英混读、情绪转折和逐步加长的段落。克隆比较使用相同、干净且可使用的参考录音，并保持参考转写一致；预设音色或音色设计另设实验组。

每条至少生成 3 次，记录成功率与重试次数，不只挑最好的一条。分开统计文本漏读/错读、声音相似度、情绪自然度、跨句一致性、冷/热启动首包、整段 RTF、峰值显存和并发等待。盲听时统一响度与播放条件，保留原始输出供检查。

长文本同时测试整段与分句方案；分句长度根据实际声音与标点调整，不把某个固定字数当成模型定律。分段可以降低失稳概率，但可能增加停顿与音色跳变，需要一起听。

### 11.3 接入视频制作时的顺序

先规范台词中的数字、专有名词与读法，再按句生成和筛选音频；固定音频后做字词时间对齐、字幕、镜头与动画。需要角色口型时，再进入独立口型模型或动画控制流程。

例如一个镜头只能容纳 4 秒台词，先检查实际生成时长，再改写文案或小幅调整 Index `duration_factor`；不要一开始就把 TTS 宣传中的“可控时长”当作精确剪辑时间线。对于声音驱动的 Remotion 短片，先确定音频节奏，通常更容易安排视觉转折。

## 12. 来源与证据范围

以下资料核查日期均为 2026-10-20。GitHub 的 README、源码、Release 和 Issue 通过 `gh` 读取；模型卡、论文与部署页面结合 Exa 检索核对。通用网页读取通道本次连接失败，部分模型卡和论文依据检索返回的正文片段，因此涉及精确复现时仍应回到对应全文。未声称完整运行任何第三方基准。

### 官方项目、模型与代码

- **[S1]** [IndexTTS 当前 README，固定提交](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/README.md)：版本、语言、情绪与语速、发音、基准和 RTF。
- **[S2]** [IndexTTS-2.5 Release](https://github.com/index-tts/index-tts/releases/tag/v2.5.0)：GitHub 发布日期。
- **[S3]** [IndexTTS-2.5 推理实现，固定提交](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/indextts/infer_v2_5.py)：22.05kHz、时长缩放、分段返回等。
- **[S4]** [Index LICENSE](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/LICENSE)、[中文协议](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/LICENSE_ZH.txt)、[依赖约束](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/pyproject.toml)。
- **[S5]** [IndexTTS 2.5 Technical Report，v5](https://arxiv.org/html/2601.03888v5)、[2.5 模型卡](https://huggingface.co/IndexTeam/IndexTTS-2.5)：架构及模型口径；论文旧版本与当前语言清单可能不同。
- **[S6]** [Qwen3-TTS README，固定提交](https://github.com/QwenLM/Qwen3-TTS/blob/022e286b98fbec7e1e916cb940cdf532cd9f488e/README.md)：五个已发布模型、预设声音、参考克隆、设计后克隆与安装。
- **[S7]** [Qwen Python 接口，固定提交](https://github.com/QwenLM/Qwen3-TTS/blob/022e286b98fbec7e1e916cb940cdf532cd9f488e/qwen_tts/inference/qwen3_tts_model.py)：流式参数注释、返回值、0.6B 指令支持边界。
- **[S8]** [Qwen3-TTS Technical Report](https://arxiv.org/html/2601.15621)：效率章节与 Table 2。
- **[S9]** [Qwen 官方微调说明](https://github.com/QwenLM/Qwen3-TTS/blob/022e286b98fbec7e1e916cb940cdf532cd9f488e/finetuning/README.md)：单说话人 SFT 和数据格式。
- **[S10]** [Qwen Issue #341：克隆复述参考尾句](https://github.com/QwenLM/Qwen3-TTS/issues/341)、[Issue #371：训练与生成对齐反馈](https://github.com/QwenLM/Qwen3-TTS/issues/371)：用户报告，非本次复现。
- **[S11]** [VoxCPM README，固定提交](https://github.com/OpenBMB/VoxCPM/blob/cce58a5b59303c9bd63d12f23afe6a49a4a80c59/README.md)：模式、语言、采样率、性能、显存与生成波动。
- **[S12]** [VoxCPM 2.0.3 Release](https://github.com/OpenBMB/VoxCPM/releases/tag/2.0.3)：2026-05-11 更新内容。
- **[S13]** [VoxCPM 使用指南](https://voxcpm.readthedocs.io/en/latest/usage_guide.html)：控制参数、Hi-Fi 模式忽略指令与长文本建议。
- **[S14]** [VoxCPM2 Technical Report](https://arxiv.org/abs/2606.06928)、[VoxCPM2 模型卡](https://huggingface.co/openbmb/VoxCPM2)。
- **[S15]** [VoxCPM Issue #372：长文本声音失真](https://github.com/OpenBMB/VoxCPM/issues/372)：问题、维护者回应和后续分段讨论。
- **[S16]** [VoxCPM Issue #419：缺少 C 编译器时 optimize 路径失败](https://github.com/OpenBMB/VoxCPM/issues/419)：2026-10-16 环境反馈。
- **[S17]** [vLLM-Omni TTS 在线服务说明，固定提交](https://github.com/vllm-project/vllm-omni/blob/01f807f6844bd063cd70dafc952bedf0ff3b6328/examples/online_serving/text_to_speech/README.md)、[对应在线文档](https://docs.vllm.ai/projects/vllm-omni/en/latest/user_guide/examples/online_serving/text_to_speech/)：三者接入、Index 非分块与采样差异、Qwen PCM/WebSocket。
- **[S18]** [IndexTTS-2.5 vLLM recipe](https://recipes.vllm.ai/IndexTeam/IndexTTS-2.5)：部署要求、原生速度映射与验证范围。
- **[S19]** [Qwen LICENSE](https://github.com/QwenLM/Qwen3-TTS/blob/022e286b98fbec7e1e916cb940cdf532cd9f488e/LICENSE)、[Qwen Base 模型卡](https://huggingface.co/Qwen/Qwen3-TTS-12Hz-1.7B-Base)、[VoxCPM LICENSE](https://github.com/OpenBMB/VoxCPM/blob/cce58a5b59303c9bd63d12f23afe6a49a4a80c59/LICENSE)：Apache-2.0 声明。

### 独立研究与社区线索

- **[S20]** [Taming Long-form Text-to-Speech，arXiv:2609.16989](https://arxiv.org/html/2609.16989)：2026 年 9 月预印本；长文本、多随机种子与 LACI，非本次复现。
- **[S21]** [Reddit：IndexTTS 2.5 is out!](https://www.reddit.com/r/TextToSpeech/comments/1vlo1dw/indextts_25_is_out/)：2026-08-11 发布讨论及口音反馈，仅作待验证线索。
- **[S22]** [Hewkick：IndexTTS2.5 评测](https://www.bilibili.com/video/BV1ZK8m6BELi/)、[T8star-Aix：逐句情感控制工具介绍](https://www.bilibili.com/video/BV1mptg6LEDA/)：本次读取搜索元数据与简介，未获取完整字幕或进行听音评测。
