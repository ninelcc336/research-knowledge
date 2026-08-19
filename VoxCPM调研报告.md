# VoxCPM 项目调研报告

## 1. 调研范围

- 项目：[OpenBMB/VoxCPM](https://github.com/OpenBMB/VoxCPM)
- 当前重点版本：`VoxCPM2`
- 调研时间：2026-08-19
- 默认分支：`main`
- 仓库动态状态：约 35,840 Star、4,100 Fork、110 个开放 Issue
- 最新稳定版：`v2.0.3`，2026-05-11 发布
- 主分支最近一次提交：2026-08-12
- 代码和官方仓库声明的模型权重许可证：Apache-2.0

本文依据官方 README/中文 README、源码树、`pyproject.toml`、官方文档、GitHub Release、公开 Issue、VoxCPM 技术报告和 VoxCPM2 技术报告整理。Star、Fork、Issue、提交时间等动态数据只代表调研时刻的状态。

## 2. 一句话结论

VoxCPM2 是一个面向多语言 TTS、文本音色设计和声音克隆的开源模型，核心使用“连续语音潜变量 + 分层语义/声学建模 + 局部扩散自回归”架构，在音色保持、韵律表现和多语言覆盖上很有竞争力；但长文本稳定性、可控克隆一致性、微调显存和不同语言之间的质量差异，仍然决定了它更适合先做针对性验证，再进入生产。

## 3. 模型家族与版本选择

| 模型 | 定位 | 参数量 | 音频输出 | 语言 | 克隆能力 | 官方单卡参考性能 |
| --- | --- | ---: | ---: | --- | --- | --- |
| VoxCPM-0.5B | 早期轻量版本 | 约 0.5–0.6B | 16kHz | 中文、英文 | 续写式克隆 | RTX 4090 RTF 约 0.17，显存约 5GB |
| VoxCPM1.5 | 1.x 稳定版本 | 约 0.6–0.8B | 44.1kHz | 中文、英文 | 续写式克隆 | RTX 4090 RTF 约 0.15，显存约 6GB |
| VoxCPM2 | 当前推荐版本 | 2B | 48kHz | 30 种语言 | 独立参考音频、续写、组合克隆 | RTX 4090 RTF 约 0.30，显存约 8GB |

参数量在官方不同页面中存在“backbone 参数量”和“总参数量”的口径差异，因此表中使用约数。官方 README 明确建议新项目使用 VoxCPM2；如果只需要中英文、较低显存和旧版工作流兼容性，可以评估 VoxCPM1.5。

## 4. 核心能力

### 4.1 文本转语音

直接输入普通文本即可生成语音，不需要额外语言标签。VoxCPM2 支持：

- 阿拉伯语、缅甸语、中文、丹麦语、荷兰语、英语、芬兰语、法语、德语、希腊语、希伯来语、印地语、印尼语、意大利语、日语；
- 高棉语、韩语、老挝语、马来语、挪威语、波兰语、葡萄牙语、俄语、西班牙语、斯瓦希里语、瑞典语、菲律宾语、泰语、土耳其语、越南语。

官方还列出 9 种中文方言：四川话、粤语、吴语、东北话、河南话、陕西话、山东话、天津话和闽南话。方言场景应使用自然、地道的方言词汇和表达，不能只把标准普通话文本加上“方言”标签。

### 4.2 Voice Design：文本设计音色

不提供参考音频，仅在目标文本前加上括号中的自然语言描述，例如：

```python
wav = model.generate(
    text="(年轻女性，温柔甜美，语速稍慢)欢迎使用 VoxCPM2。",
    cfg_value=2.0,
    inference_timesteps=10,
)
```

可描述性别、年龄、音高、音色、情绪、语速和说话风格。该模式每次生成可能得到不同音色，不适合需要跨请求严格固定说话人的场景。

### 4.3 Controllable Voice Cloning：可控声音克隆

提供参考音频，不需要参考文本。模型从参考音频提取音色，再通过括号指令控制情绪、语速和表达风格：

```python
wav = model.generate(
    text="(语气轻快、稍微快一些)这是一个可控声音克隆示例。",
    reference_wav_path="speaker.wav",
)
```

这是功能最方便的克隆模式，但也是当前 Issue 中反馈稳定性较多的模式。同一参考音频和同一文本多次生成，音量、语速或音色可能有差异。

### 4.4 Ultimate Cloning：高保真续写式克隆

同时提供参考音频和其准确转写文本，模型以参考语音为上下文进行延续式生成：

```python
wav = model.generate(
    text="这是参考语音之后要继续说的内容。",
    prompt_wav_path="speaker.wav",
    prompt_text="speaker.wav 中实际说出的完整文字。",
    reference_wav_path="speaker.wav",
)
```

该模式通常具有更高的音色、节奏、情绪和说话风格相似度，但对参考音频和转写文本严格匹配的要求更高。官方使用指南说明，高保真模式下括号中的控制指令会被忽略。

### 4.5 流式生成

Python API 提供 `generate_streaming()`，逐个返回音频块。项目在 VoxCPM 2.0.3 中优化了流式 VAE 解码，使用有状态的解码器减少重复重叠解码开销。实际交互产品仍建议按句切分后分别流式生成，而不是把一个不断增长的超长文本持续塞给模型。

## 5. 技术架构

### 5.1 总体流程

```text
文本 / 参考音频
        |
        v
Local Encoder（LocEnc，音频 patch 编码）
        |
        v
Text-Semantic LM（TSLM，文本语义与韵律规划）
        |
        v
Residual Acoustic LM（RALM，声学细节建模）
        |
        v
Local DiT（LocDiT，Conditional Flow Matching 扩散解码）
        |
        v
AudioVAE / AudioVAE V2
        |
        v
波形
```

### 5.2 Tokenizer-free 的含义

VoxCPM 不依赖传统 TTS 常见的离散音频 tokenizer，把语音编码到 AudioVAE 的连续潜空间，并在这个空间里生成语音表征。这样可以减少离散码本带来的细节损失，并保留更连续的音色和韵律信息。

不过，VoxCPM2 并非完全没有离散约束。TSLM 内部使用 FSQ（Finite Scalar Quantization）形成半离散瓶颈，让语义/韵律规划更稳定；RALM 和 LocDiT 继续负责连续声学细节。

### 5.3 VoxCPM2 相对 1.x 的关键变化

| 组件 | VoxCPM 1.x | VoxCPM2 | 影响 |
| --- | --- | --- | --- |
| RALM 融合 | 语义表示与音频特征相加 | concat + projection | 允许模型学习更灵活的语义-声学融合 |
| LocDiT 条件 | 单个融合条件 token | 多 token 条件前缀 | 减少早期信息压缩，提升表达和可控性 |
| 参考音频 | 主要是 prompt continuation | 独立 reference audio 通道 | 能把“谁在说”和“从哪里续写”分开 |
| AudioVAE | 对称编解码 | 16kHz 编码、48kHz 解码 | 保持 6.25Hz 的语言模型序列效率，同时输出高采样率音频 |
| FSQ latent dim | 256 | 512 | 提高大规模模型的表征容量 |
| 最大序列长度 | 4096 | 8192 | 为更长文本提供更大上下文空间，但不等于长文本稳定性已经解决 |

VoxCPM2 使用 MiniCPM-4 作为语言模型基础，并通过统一的序列组织方式把零样本、参考克隆、续写和组合克隆放进同一套模型与训练目标中。

## 6. 官方效果与性能

### 6.1 Seed-TTS-eval

官方 README 给出的 VoxCPM2 结果如下：

| 测试集 | 文本准确率 | 说话人相似度 SIM |
| --- | ---: | ---: |
| 英文 test-EN | WER 1.84% | 75.3 |
| 中文 test-ZH | CER 0.97% | 79.5 |
| 中文困难集 test-ZH-Hard | CER 8.13% | 75.3 |

WER/CER 越低越好，SIM 越高越好。WER/CER 主要衡量“是否说对了文本”，SIM 主要衡量“是否像参考说话人”，二者不能单独作为自然度或总体体验的替代指标。

### 6.2 CV3-eval 与内部多语言测试

官方 CV3-eval 表中，VoxCPM2 的代表性 WER/CER 为：中文 `3.65`、英文 `5.00`、困难中文 `8.55`、困难英文 `8.48`、日语 `5.96`、韩语 `5.69`、德语 `4.77`、西班牙语 `3.80`、法语 `9.85`、意大利语 `4.25`、俄语 `5.21`。

项目还进行了一项内部测试：30 种语言、每种 500 条样本，使用 Gemini 3.1 Flash Lite API 转写，平均 WER/CER 为 `1.68%`。这是项目内部结果，评测转写器、数据分布和对比基线与公共基准不同，不能直接等同于第三方独立评测。

### 6.3 实时因子和显存

官方模型对比表给出的口径：

| 运行方式 | VoxCPM2 RTX 4090 |
| --- | ---: |
| 标准 PyTorch 推理 RTF | 约 0.30 |
| Nano-vLLM 推理 RTF | 约 0.13 |
| 官方表格标注显存 | 约 8GB |

RTF（Real-Time Factor）越低越快，例如 RTF 0.30 表示理论上生成 1 秒音频约需要 0.30 秒计算时间。实际延迟还会受到模型首次加载、`torch.compile` warm-up、参考音频编码、VAE 解码、文本长度、扩散步数和并发数影响。

## 7. 安装与最小使用方式

### 7.1 环境要求

- Python：建议 3.10–3.12；项目当前文档和 README 要求 `<3.13`。
- PyTorch：`>=2.5.0`。
- NVIDIA CUDA：GPU 加速建议 CUDA `>=12.0`。
- CPU 和 Apple Silicon MPS 可以运行，但速度和兼容性需要实测。
- 首次使用会从 Hugging Face 下载模型；国内网络环境可以先从 ModelScope 下载到本地。

安装：

```bash
pip install voxcpm
```

基础 Python API：

```python
from voxcpm import VoxCPM
import soundfile as sf

model = VoxCPM.from_pretrained(
    "openbmb/VoxCPM2",
    load_denoiser=False,
)

wav = model.generate(
    text="VoxCPM2 是一个多语言语音合成模型。",
    cfg_value=2.0,
    inference_timesteps=10,
    seed=42,
)

sf.write("demo.wav", wav, model.tts_model.sample_rate)
```

保存音频时应读取 `model.tts_model.sample_rate`，不要把 1.x 的 `44100` 硬编码到 VoxCPM2。VoxCPM2 的输出采样率是 48000Hz。

### 7.2 CLI

```bash
# 文本音色设计
voxcpm design \
  --text "欢迎使用 VoxCPM2。" \
  --control "年轻女性，温柔、清晰、语速稍慢" \
  --output out.wav

# 参考音频克隆
voxcpm clone \
  --text "这是一个声音克隆示例。" \
  --reference-audio speaker.wav \
  --output clone.wav

# 高保真续写式克隆
voxcpm clone \
  --text "这是后续需要合成的内容。" \
  --prompt-audio speaker.wav \
  --prompt-text "speaker.wav 中的准确转写文本" \
  --reference-audio speaker.wav \
  --output hifi_clone.wav
```

Web Demo 需要源代码仓库，并且首次使用参考音频转写时还会下载 SenseVoice-Small ASR 模型：

```bash
git clone https://github.com/OpenBMB/VoxCPM.git
cd VoxCPM
pip install -e .
python app.py --port 8808
```

如果 `torch.compile` 在 CPU、MPS 或特殊 CUDA 环境下不兼容，可以使用 `optimize=False` 或 CLI 的 `--no-optimize`。

## 8. 部署路线

### 8.1 标准 PyTorch

适合单机验证、低并发或需要直接使用全部 Python API 的场景。默认会尝试 `torch.compile` 并进行 warm-up；调试时可以关闭优化。

### 8.2 Nano-vLLM-VoxCPM

[Nano-vLLM-VoxCPM](https://github.com/a710128/nanovllm-voxcpm) 是面向 VoxCPM 的高吞吐推理引擎，提供并发请求、异步 API 和 FastAPI 服务。官方 README 给出的 RTX 4090 RTF 约为 `0.13`，但该指标通常来自特定单请求或固定配置，生产环境仍应压测并发、首包延迟和长文本场景。

### 8.3 vLLM-Omni

[vLLM-Omni](https://github.com/vllm-project/vllm-omni) 是官方 vLLM 项目的全模态扩展，提供：

- PagedAttention 和连续批处理；
- 多请求并发和多 GPU 部署；
- OpenAI 兼容的 `/v1/audio/speech` 接口；
- 流式音频输出。

官方文档建议从源码安装，当前安装命令示例包含 `vllm==0.19.0`。Linux GPU 环境更适合作为生产服务基础。GitHub Issue #374 中维护者说明，官方示例的 `max_num_seqs` 默认值较保守（8），10 个并发请求出现分批等待属于连续批处理行为；可以依据显存和延迟目标调高 `max_num_seqs`、`max_num_batched_tokens` 和 GPU 利用率，并移除仅适合调试的 `--enforce-eager`。

### 8.4 C++/端侧生态

README 目前列出多个社区运行时：

- `llama.cpp-omni`：支持 VoxCPM2 GGUF，可运行在 CPU、Metal、CUDA、Vulkan；README 标注 Apple M4 Pro/Metal 上 Q8_0 RTF 约 1.76。
- `VoxCPM.cpp`：基于 ggml/GGUF，但官方文档当前明确列出的版本支持主要是 VoxCPM 1.0/1.5，不能把它与 VoxCPM2 兼容性混同。
- `VoxCPM-ONNX`、`VoxCPMANE`、`voxcpm_rs`：分别面向 ONNX、Apple Neural Engine 和 Rust 等方向。

这些项目不是 OpenBMB 主仓库的同一维护边界，部署前应检查具体版本、权重转换、功能覆盖和许可证。

## 9. 显存、微调与数据准备

### 9.1 推理显存

官方表格标注 VoxCPM2 约 8GB 显存，但这只是基础推理参考。长文本、较长参考音频、VAE 编解码、denoiser、`torch.compile` 缓存和并发服务都会增加实际占用。建议：

- 8GB 级别：只把它作为短文本、单路、关闭额外组件的最低验证起点。
- 12–16GB：更适合单路常规使用，并留出参考音频和编译开销。
- 24GB：更适合作为稳定单卡推理或低并发服务起点。
- 高并发：依据 KV cache 和实际请求长度压测，不能按“模型约 8GB”直接估算并发容量。

### 9.2 LoRA 微调

项目支持 LoRA 微调，官方 V2 配置示例的关键参数为：

```yaml
batch_size: 2
grad_accum_steps: 8
learning_rate: 0.0001
max_batch_tokens: 8192
lora:
  enable_lm: true
  enable_dit: true
  enable_proj: false
  r: 32
  alpha: 32
  dropout: 0.0
```

LoRA 通常是单说话人、音色、风格适配的首选。官方 FAQ 说明 LoRA 显存大约是全量微调的一半，社区维护者还特别强调 `enable_dit: true` 对声音质量很重要。

### 9.3 全量微调

全量微调使用更小的学习率，官方示例为 `1e-5`，显存和 checkpoint 体积都明显高于 LoRA。Issue #335 中，用户在 4 张 24GB RTX 3090 上仍遇到 OOM；维护者说明单卡显存不足时，多卡总显存不能简单替代单卡显存，降低 batch/长度并增加梯度累积可以缓解，但全量微调仍应准备 40GB 级别单卡显存作为更现实的起点。

这不是一个适用于所有数据和配置的硬性最低值，实际峰值取决于样本长度、`batch_size`、`max_batch_tokens`、混合精度、梯度检查点和 DDP 缓冲。

### 9.4 数据与过拟合

训练清单是 JSONL，每行至少包含：

```json
{"audio":"data/example.wav","text":"对应的音频转写文本"}
```

可选字段包括 `duration`、`dataset_id` 和 VoxCPM 2.0.3 新增支持的 `ref_audio`。当使用 `ref_audio` 时，训练管线会将参考音频作为上下文，只对目标音频片段计算损失。

官方微调文档提醒：小数据集可能在几百步内过拟合，单说话人通常 1–3 个 epoch 就应重点检查；一旦模型开始忽略输入文本、对不同文本生成近似同一段声音，应回退到更早 checkpoint。新语言微调建议使用大量目标语言数据，并混入部分中英文数据以减轻遗忘。

## 10. 已知限制和真实反馈

### 10.1 长文本失真

官方使用指南明确将长文本列为最容易触发问题的场景，包括：逐渐加速、啸叫或噪声、显存增长、无法停止。官方建议按短句或段落切分后分段生成，再拼接波形。

GitHub Issue #372 中，用户报告约 300 字中文文本在约 20 秒后出现严重失真，并称标准 PyTorch、Nano-vLLM 和 vLLM-Omni 都能复现；维护者回应“长文本确实会有失真问题，目前一直在优化，但还没有彻底解决”。因此，8192 最大序列长度不能理解为“生产级长篇生成已经稳定”。

### 10.2 短文本与克隆一致性

- Issue #361 报告克隆模式下 4 字及以下短句效果异常。
- Issue #362 报告同一参考音频和相同文案多次生成时，音量、语速和音色可能变化。
- Issue #362 的跟进反馈指出，带参考音频和转写文本的 Ultimate Cloning 通常比只提供参考音频的可控克隆更稳定。

### 10.3 参考音频质量

官方文档建议参考音频使用 5–30 秒、干净、少噪声的片段。`denoise=True` 处理的是 prompt/reference 音频，不是生成结果本身；denoiser 运行在 16kHz 管线，可能轻微改变音色，干净参考音频不一定适合开启。

### 10.4 多语言质量并不均匀

“支持 30 种语言”描述的是模型覆盖范围，不代表每种语言的韵律、发音、文本准确率和音色克隆都达到中文/英文水平。官方内部结果中，不同语言的 WER/CER 差异明显；实际项目应按目标语言单独建立测试集。

### 10.5 运行环境和版本风险

- 项目当前文档要求 Python `<3.13`；Issue #340 中 Python 3.14 因 `funasr` 依赖缺少预编译 wheel 而安装失败。
- MPS 运行在近期版本中有改进，但 Issue #337 仍反映过 Mac M 芯片 MPS 体验问题，建议用最新版本并关闭优化进行排查。
- VoxCPM2 将 `from_pretrained()` 默认模型从 1.5 改为 2.0，并将默认输出从 44.1kHz 改为 48kHz；升级旧代码时必须显式固定模型 ID，并使用模型自身的采样率。
- `v2.0.3` 新增 manifest 校验、设备选择、安全 LoRA 加载和流式解码改进，建议不要长期停留在早期 2.0.x 版本。

## 11. 许可证、安全与合规

仓库 LICENSE 为 Apache-2.0，允许商业使用、修改和再分发，但再分发时需要保留许可证、版权/专利/归属声明，并对修改文件做显著说明；Apache-2.0 不授予项目商标使用权。

声音克隆存在明显滥用风险。项目 README 明确禁止冒充、欺诈和虚假信息，并建议标记 AI 生成内容。实际产品至少应具备：

- 参考音频所有者的明确授权和可审计记录；
- 禁止公众人物、客服身份和认证环节的无授权克隆策略；
- 生成内容标识、日志、滥用检测和投诉/下架机制；
- 对第三方依赖、模型权重、音频数据和上游部署组件分别做许可证审查。

“Apache-2.0 / 商用友好”只说明仓库和官方发布物的授权条件较宽松，不代表可以无条件使用任意参考音频，也不替代目标地区的隐私、肖像、著作权和 AI 内容监管评估。

## 12. 选型建议

### 12.1 适合优先评估的场景

- 需要私有化部署的多语言 TTS；
- 需要用自然语言设计新音色；
- 需要参考音频克隆并控制风格；
- 需要 48kHz 输出并且能接受本地 GPU 运维；
- 需要 LoRA 适配特定说话人或领域风格；
- 需要在同一模型中同时覆盖普通 TTS、参考克隆和续写式克隆。

### 12.2 不应直接承诺的场景

- 不经分段的长篇有声书、几十秒以上连续播报；
- 需要每次生成完全相同音色、音量和语速的确定性生产链路；
- 只有 CPU 或很小显存、但又要求 48kHz 和低延迟；
- 需要对所有 30 种语言提供同等质量 SLA；
- 未经授权的名人或他人声音复制。

### 12.3 推荐 PoC 验证流程

1. 先固定 `VoxCPM2` 版本、Python、PyTorch、CUDA 和模型权重哈希。
2. 为每个目标语言准备 20–50 条真实业务文本，覆盖数字、专有名词、长句、短句和标点。
3. 准备 5–30 秒、已获授权且干净的参考音频，分别测试可控克隆和 Ultimate Cloning。
4. 将长文本按句或短段生成，比较整段生成和分段拼接的失真、停顿与音色一致性。
5. 用固定 seed 重复生成，同时保留多 seed 结果，评估可重复性。
6. 同时测量文本准确率、说话人相似度、主观自然度、首包延迟、RTF、峰值显存和并发吞吐。
7. 只有在标准 PyTorch 路径稳定后，再迁移到 Nano-vLLM 或 vLLM-Omni，并重新压测并发行为。

总体建议是：VoxCPM2 可以作为开源多语言声音生成的重点候选，但不要只依据官方 demo、单个 SIM 指标或“约 8GB 显存”做上线决策。对于长文本和商业生产，至少应与 CosyVoice3、Qwen3-TTS、F5-TTS 或 IndexTTS2 在自己的数据集上进行对照测试。

## 13. 主要来源

### 官方仓库与文档

- [GitHub 主仓库](https://github.com/OpenBMB/VoxCPM)
- [中文 README](https://github.com/OpenBMB/VoxCPM/blob/main/README_zh.md)
- [官方文档](https://voxcpm.readthedocs.io/en/latest/)
- [快速开始](https://voxcpm.readthedocs.io/en/latest/quickstart.html)
- [使用指南](https://voxcpm.readthedocs.io/en/latest/usage_guide.html)
- [架构说明](https://voxcpm.readthedocs.io/en/latest/models/architecture.html)
- [VoxCPM2 模型说明](https://voxcpm.readthedocs.io/en/latest/models/voxcpm2.html)
- [微调指南](https://voxcpm.readthedocs.io/en/latest/finetuning/finetune.html)
- [微调 FAQ](https://voxcpm.readthedocs.io/en/latest/finetuning/faq.html)
- [安装说明](https://voxcpm.readthedocs.io/en/latest/installation.html)
- [vLLM-Omni 部署](https://voxcpm.readthedocs.io/en/latest/deployment/vllm_omni.html)
- [最新 Release：v2.0.3](https://github.com/OpenBMB/VoxCPM/releases/tag/2.0.3)
- [Apache-2.0 LICENSE](https://github.com/OpenBMB/VoxCPM/blob/main/LICENSE)

### 技术报告

- [VoxCPM2 Technical Report，arXiv:2606.06928](https://arxiv.org/abs/2606.06928)
- [VoxCPM Technical Report，arXiv:2509.24650](https://arxiv.org/abs/2509.24650)

### 运行时与模型权重

- [Hugging Face：VoxCPM2](https://huggingface.co/openbmb/VoxCPM2)
- [ModelScope：VoxCPM2](https://modelscope.cn/models/OpenBMB/VoxCPM2)
- [Nano-vLLM-VoxCPM](https://github.com/a710128/nanovllm-voxcpm)
- [vLLM-Omni](https://github.com/vllm-project/vllm-omni)
- [llama.cpp-omni](https://github.com/tc-mb/llama.cpp-omni)

### 公开 Issue 与实际反馈

- [Issue #372：长文本声音失真](https://github.com/OpenBMB/VoxCPM/issues/372)
- [Issue #374：vLLM-Omni 并发性能观察](https://github.com/OpenBMB/VoxCPM/issues/374)
- [Issue #335：VoxCPM2 微调显存不足](https://github.com/OpenBMB/VoxCPM/issues/335)
- [Issue #361：克隆模式短句异常](https://github.com/OpenBMB/VoxCPM/issues/361)
- [Issue #362：音量与克隆音色不准确](https://github.com/OpenBMB/VoxCPM/issues/362)
- [Issue #337：Mac MPS 加速反馈](https://github.com/OpenBMB/VoxCPM/issues/337)
- [Issue #340：Python 3.14 安装失败](https://github.com/OpenBMB/VoxCPM/issues/340)
