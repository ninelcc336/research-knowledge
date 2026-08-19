# VibeVoice 项目调研

## 1. 调研范围

- 项目：[microsoft/VibeVoice](https://github.com/microsoft/VibeVoice)
- 关联仓库：[microsoft/VibeASR.cpp](https://github.com/microsoft/VibeASR.cpp)
- 调研时间：2026-08-19
- 默认分支：`main`
- 主仓库最近一次提交：`94da20d`，2026-07-24
- 主仓库动态状态：约 52,894 Star、5,969 Fork、183 个开放 Issue
- 代码许可证：MIT

本文依据官方仓库 README、当前文档和代码树、GitHub API、公开 Issue/PR、Hugging Face 模型卡、VibeVoice TTS/ASR 技术报告以及 VibeASR.cpp 技术报告整理。Star、Fork、Issue 等动态数据只代表调研时刻的状态。

## 2. 一句话结论

VibeVoice 是微软推出的长上下文语音 AI 模型家族，技术重点是用 7.5Hz 连续语音 Tokenizer 和 LLM/扩散头处理长音频、多人对话和语音上下文。当前最具实际价值的是 VibeVoice-ASR；原始多说话人 TTS 因滥用风险撤下了官方推理代码和试玩入口，Realtime TTS 则更适合英文实时语音助手原型，不宜直接视为成熟的通用中文 TTS 服务。

## 3. 当前模型家族

| 模型 | 定位 | 官方能力 | 实际判断 |
| --- | --- | --- | --- |
| VibeVoice-ASR-7B | 长音频语音识别 | 单次最长约 60 分钟；识别、说话人、时间戳一体化；50+ 语言；热词和自定义上下文 | 当前最值得优先评估，但显存门槛高 |
| VibeVoice-Realtime-0.5B | 实时文本转语音 | 单说话人、8K 上下文、约 10 分钟长音频、约 200-300ms 首段延迟 | 适合英文实时助手和流式原型 |
| VibeVoice-TTS-1.5B | 长篇多人 TTS | 最长约 90 分钟、最多 4 位说话人、对话式表达 | 论文能力突出，但官方推理入口已禁用 |
| VibeVoice-Large | 长篇多人 TTS 大模型 | 约 45 分钟上下文 | 当前官方标为 Disabled |
| VibeVoice-ASR-BitNet | CPU 端 ASR | 1.58GB 量化模型，CPU 实时多语言识别 | 没有大显卡时的优先路线 |

## 4. 技术架构

### 4.1 连续语音 Tokenizer

VibeVoice 使用声学 Tokenizer 和语义 Tokenizer，将语音表示压缩到 7.5Hz。技术报告同时使用了两种描述：相对于 24kHz 原始采样帧率约为 3200 倍压缩；相对于 EnCodec，数据压缩效率提升约 80 倍。

低帧率减少了长音频进入 LLM 后的序列长度，使 64K 上下文处理长对话成为可能，同时保留语音的音色、语义和韵律信息。

### 4.2 LLM 与 next-token diffusion

- LLM：基于 Qwen2.5，负责文本语境、对话流程和角色信息理解。
- Diffusion Head：根据 LLM 隐状态生成连续声学表示。
- 解码器：将连续语音表示恢复成音频。

Realtime 版本为了降低延迟，移除了语义 Tokenizer，主要依赖声学 Tokenizer，并采用窗口化、交错式的增量生成方式。

## 5. VibeVoice-ASR-7B

### 5.1 核心能力

- 最长约 60 分钟音频单次处理，官方强调不需要常规切片。
- 一次生成 `Who / When / What`：说话人、开始/结束时间、文本内容。
- 50+ 语言，支持自动语言识别和跨语言 code-switching。
- 支持自定义热词和上下文，适合人名、产品名、技术术语等专业场景。
- 提供普通 Transformers 推理、Gradio Demo、vLLM OpenAI 兼容 API 和 LoRA 微调代码。

### 5.2 官方论文指标

ASR 技术报告在 AISHELL-4、AMI-IHM、AMI-SDM、AliMeeting 和 MLC-Challenge 等数据集上报告了如下平均结果：

| 指标 | 平均值 | 含义 |
| --- | ---: | --- |
| DER | 3.42 | 说话人归属、漏检和误检综合错误率 |
| cpWER | 14.81 | 忽略局部时间对齐、关注说话人和文本准确率 |
| tcpWER | 15.66 | 同时考虑说话人、文本和时间对齐 |
| WER | 12.07 | 不考虑说话人和时间戳的文本错误率 |

论文称其在多个长音频、多说话人基准上优于 Gemini-2.5-Pro 和 Gemini-3-Pro。需要注意，论文中的 Gemini 对比使用了 240 秒切片，而 VibeVoice-ASR 使用单次长音频推理，比较口径并不完全相同。

### 5.3 服务化能力

仓库提供 vLLM 插件，可通过 OpenAI 兼容的 `/v1/chat/completions` 接口提供转录服务，支持：

- 单 GPU 推理；
- Tensor Parallel，将一个模型拆到多张 GPU；
- Data Parallel，运行多个模型副本提高吞吐；
- 长音频请求、热词和自动恢复脚本。

LoRA 微调代码位于 `finetuning-asr/`，训练数据需要音频、说话人、时间戳、文本和可选自定义上下文。

## 6. 显存和性能门槛

“60 分钟单次处理”并不代表 24GB 显卡可以稳定完成。GitHub Issue 中的社区实测显示：

- RTX 4090 24GB，默认 SDPA：30 分钟音频约 22GB 显存并成功；50 分钟 OOM；
- L20 48GB：1 小时音频约 34GB 显存；
- 模型加载本身可能占用约 22GB 显存；
- 减小 `max_new_tokens`、使用 Flash Attention、单样本推理可以降低额外开销，但不能消除长音频的显存增长。

因此建议：

- 24GB 显卡：适合短音频或切片处理，不要默认承诺 60 分钟单次处理；
- 48GB 显卡：更适合 30-60 分钟长音频；
- 80GB 级别 GPU：更接近官方长音频能力的目标环境。

上述显存数字来自用户 Issue，不是微软统一发布的硬件规格，应在目标音频、语言和注意力实现下重新压测。

## 7. VibeVoice-ASR-BitNet 与 CPU 路线

[VibeASR.cpp](https://github.com/microsoft/VibeASR.cpp) 是官方 CPU 推理运行时，使用异构量化：

- VAE Tokenizer 使用 I8_S；
- Qwen2.5-1.5B 解码器使用 I2_S BitNet 权重；
- 模型体积从 4.62GB 降到 1.58GB；
- 官方声称相较同等模型规模的 Whisper.cpp 快 1.6-2.3 倍；
- 在 AMD EPYC、Apple M4 和 Intel i7-13700 测试中，3 个线程左右即可达到 RTF < 1。

它将完整 ASR-7B 的语言模型替换为 1.5B 量化模型，因此并非无损压缩。官方表格显示，在部分多说话人数据集上 WER 增加明显，例如 AISHELL-4 从 FP16 基线的 19.83 增至 27.45；它的优势是本地化、低内存和实时速度，而不是全面超过 7B 模型。

Windows 部署需要 GCC/Clang，官方明确不支持 MSVC，推荐 MinGW-w64。基本依赖包括 Python 3.9+、CMake 3.14+ 和约 2GB 磁盘空间。

## 8. VibeVoice-Realtime-0.5B

### 8.1 能力

- 0.5B 参数量；
- 单说话人；
- 8K 上下文，约 10 分钟音频生成；
- 官方文档给出约 200ms 首段音频，根 README 给出约 300ms，实际还会叠加网络和播放缓冲；
- NVIDIA T4 和 Apple M4 Pro 在官方测试中可以达到实时表现；
- 提供英文预置音色以及德、法、意、日、韩、荷、波、葡、西九种实验音色。

### 8.2 限制

- 英文是主要目标语言，其他语言属于实验能力，官方不承诺中文稳定输出；
- 只支持单说话人，不适合多人播客；
- 不适合代码、公式、特殊符号和极短文本；
- 文档的 TODO 仍包括真正的流式 Token 输入，说明“低首帧延迟”和成熟的边生成边接收任意新 Token 不是完全等价的能力。

## 9. VibeVoice-TTS 当前状态

原始 TTS 在 2025-08-25 发布，技术报告主张：

- 64K 上下文；
- 最长约 90 分钟；
- 最多 4 个说话人；
- 英文、中文长对话及跨语言表现；
- 具有自然停顿、情绪、角色轮换和偶发歌唱等涌现能力。

论文在约 1 小时的长对话评测中使用 24 位标注者。报告的代表性结果包括：VibeVoice-1.5B 的 Whisper WER 为 1.11，VibeVoice-7B 的说话人相似度 SIM 为 0.692。

但 2025-09-05 微软表示发现模型被用于与项目初衷不一致的场景，因此移除了 TTS 代码。当前仓库仍保留 TTS 文档、公共建模模块和权重链接，但安装与官方试玩入口已禁用。中文文档还明确提到中文发音不稳定、中文数据量较少、建议使用英文标点；这些内容只能作为历史研究资料，不能视为当前可直接部署的官方产品。

## 10. 工程成熟度与已知问题

当前开放 Issue 主要集中在以下方面：

- 长音频显存不足和 OOM；
- ASR 重复输出或生成循环；
- 小幅音频增益变化导致说话人分离结果变化；
- vLLM 版本兼容和音频预处理问题；
- Playground 偶发没有可用界面；
- Realtime 语音预设加载问题。

安全方面，仓库曾存在 `.pt` 预设通过不安全 `torch.load` 触发任意代码执行的风险。主分支已经加入 `weights_only=True` 和安全反序列化相关修改，但 [Issue #392](https://github.com/microsoft/VibeVoice/issues/392) 报告了 PyTorch 新版本下的加载回归，[PR #397](https://github.com/microsoft/VibeVoice/pull/397) 提议使用受限 Unpickler，截至调研时仍未合并。

使用第三方语音预设时，应将其视为不可信输入，不应恢复到 `weights_only=False` 的旧加载方式。

## 11. 许可证与合规边界

- GitHub 仓库代码使用 MIT License。
- 模型权重的 Hugging Face 模型卡有额外使用限制，不能仅依据仓库 MIT License 判断所有权重用途。
- 禁止未经明确、可记录同意的声音冒充或声音克隆。
- 不应用于深度伪造、误导性录音、诈骗、认证绕过或实时低延迟变声。
- 非英文/中文 TTS 输入可能不可用；Realtime 的稳定目标主要是英文。
- 官方建议仅用于研究和开发，不建议未经进一步测试直接用于商业或真实生产场景。

## 12. 选型建议

| 需求 | 建议 | 原因 |
| --- | --- | --- |
| 会议、播客、访谈转录 | VibeVoice-ASR-7B | 原生输出说话人、时间戳和文本 |
| 24GB 以下显卡或本地 CPU | VibeASR.cpp | 1.58GB、CPU 实时、部署成本低 |
| 英文实时语音助手 | VibeVoice-Realtime-0.5B | 参数量小、首段延迟低、适合 LLM 边生成边播报原型 |
| 中文实时 TTS | 不建议直接选 VibeVoice-Realtime | 官方未将中文列为稳定支持语言 |
| 长篇多人播客 TTS | 仅作研究评估 | 原始 TTS 代码和试玩已禁用，商业可用性不足 |
| 商业语音产品 | 先做授权、安全和准确率审查 | 权重有额外限制，官方也定位为研究开发用途 |

## 13. 主要来源

- [VibeVoice 官方仓库](https://github.com/microsoft/VibeVoice)
- [VibeVoice TTS 技术报告](https://arxiv.org/abs/2508.19205)
- [VibeVoice-ASR 技术报告](https://arxiv.org/abs/2601.18184)
- [VibeVoice-ASR-BitNet 技术报告](https://arxiv.org/abs/2607.21075)
- [VibeVoice ASR 文档](https://github.com/microsoft/VibeVoice/blob/main/docs/vibevoice-asr.md)
- [VibeVoice Realtime 文档](https://github.com/microsoft/VibeVoice/blob/main/docs/vibevoice-realtime-0.5b.md)
- [VibeVoice vLLM ASR 部署文档](https://github.com/microsoft/VibeVoice/blob/main/docs/vibevoice-vllm-asr.md)
- [VibeVoice-TTS 模型卡](https://huggingface.co/microsoft/VibeVoice-1.5B)
- [VibeVoice-Realtime-0.5B 模型卡](https://huggingface.co/microsoft/VibeVoice-Realtime-0.5B)
- [VibeASR.cpp 官方仓库](https://github.com/microsoft/VibeASR.cpp)
- [显存与长音频 Issue #367](https://github.com/microsoft/VibeVoice/issues/367)
- [CUDA OOM Issue #210](https://github.com/microsoft/VibeVoice/issues/210)
- [重复转录 Issue #415](https://github.com/microsoft/VibeVoice/issues/415)
- [语音预设加载 Issue #392](https://github.com/microsoft/VibeVoice/issues/392)

