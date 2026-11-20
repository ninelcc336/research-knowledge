---
title: "CosyVoice 3 与 IndexTTS 2.5 本地部署及 8 GB 显存评估"
date: 2026-11-20
category: "AI语音"
tags:
  - 语音合成
  - 声音克隆
  - 模型部署
  - 情绪控制
  - 商业许可
  - Windows
---

# CosyVoice 3 与 IndexTTS 2.5 本地部署及 8 GB 显存评估

调研日期为 2026-11-20。范围是 Windows 11 本机的官方权重、单卡推理、WebUI 和短句验收。已核查 GPU 与 WSL2 可见性，并在 Windows 上用 `uv sync --dry-run` 验证 IndexTTS 的锁文件能选中 Python 3.10、CUDA 版 PyTorch 和 WebUI 依赖。本次完成资料与环境核查，模型推理峰值与生成速度待本机跑样片确认。

## 本机能否运行

本机为 **RTX 4060 Ti 8 GB、约 32 GB 系统内存、Windows 11 Pro**，NVIDIA 驱动版本为 `617.42`。Windows 读到显存总量 8188 MiB、当时占用 933 MiB。已有 Ubuntu 22.04 WSL2，运行 `nvidia-smi` 能识别同一张卡，当时可用约 7035 MiB。原生 Windows 已有 `uv`、Git 和 Python 3.10。

| 模型 | 在这台机器上的判断 | 第一轮配置 |
| --- | --- | --- |
| CosyVoice 3 | **预计可用**。有条件用官方 0.5B 权重做单人、单句克隆。本机峰值待测 | WSL2，PyTorch 路径，用 Python API 开启 `fp16=True`，单句串行 |
| IndexTTS 2.5 | **8 GB 可用于低显存配音，余量较窄**。上游有低显存路径和同容量 GPU 的单次推理记录 | 原生 Windows，官方 WebUI，BF16，保留默认的 QwenEmotion 跳过行为，单句串行 |

- **CosyVoice 3 的 0.5B 是语言模型规模口径。** 官方加载 `llm.pt`、`flow.pt`、`hift.pt`，这三个磁盘文件合计约 3.20 GiB，另初始化说话人和语音 tokenizer 的 ONNX session。推理时还需要缓存、激活和 ONNX 运行开销。源码的 `fp16=True` 主要通过 autocast 生效，加载流程仍会把检查点张量搬到 GPU，启动峰值也要测。官方文档未给这套配置的 8 GB 峰值。 [模型文件树](https://huggingface.co/FunAudioLLM/Fun-CosyVoice3-0.5B-2512/tree/29e01c4e8d000f4bcd70751be16fa94bf3d85a18) · [模型加载和 autocast](https://github.com/QwenAudio/CosyVoice/blob/074ca6dc9e80a2f424f1f74b48bdd7d3fea531cc/cosyvoice/cli/model.py) · [ONNX 前端](https://github.com/QwenAudio/CosyVoice/blob/074ca6dc9e80a2f424f1f74b48bdd7d3fea531cc/cosyvoice/cli/frontend.py)
- **IndexTTS 2.5 的官方 WebUI 在显卡总容量低于 10 GiB 时尝试用 BF16，并默认跳过 QwenEmotion。** 参考音色、独立情绪音频和情绪向量仍可用。8 GB 本身会触发此逻辑，可用显存仍取决于桌面及其他进程占用。推理源码也会为低显存卡拆分长文本。主分支合入的 PR 报告在 RTX 5060 8 GB 上做过单次 FP32 推理，另有同为 8 GB 的 RTX 4060 Windows 短句记录，但后者为注明由 AI 生成的社区 Issue，使用了额外加速组件，且报告出现了显存分页。两条记录用于判断可行性，峰值和速度以本机为准。 [WebUI 低显存逻辑](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/webui.py#L115) · [8 GB 贡献者实测](https://github.com/index-tts/index-tts/pull/813) · [RTX 4060 社区记录](https://github.com/index-tts/index-tts/issues/835)

单次运行一个引擎。语速、参考音频长度、长句、并发以及其他程序的显存占用都会改变结果。当前判断针对普通话单人克隆和逐句配音。

## CosyVoice 3 在 WSL2 部署

使用阿里官方 `FunAudioLLM/Fun-CosyVoice3-0.5B-2512` 的基础权重。代码和该权重的模型卡均标为 **Apache-2.0**。官方依赖采用 Python 3.10、PyTorch 2.3.1，其 Linux 依赖表使用 CUDA 版 ONNX Runtime。以下步骤使用 Conda，需先在 WSL 内装好 Linux 版 Miniconda 并启用终端中的 `conda`。WSL 已能看到 Windows 驱动提供的 CUDA 设备。 [官方安装说明](https://github.com/QwenAudio/CosyVoice/blob/074ca6dc9e80a2f424f1f74b48bdd7d3fea531cc/README.md#install) · [依赖表](https://github.com/QwenAudio/CosyVoice/blob/074ca6dc9e80a2f424f1f74b48bdd7d3fea531cc/requirements.txt) · [Miniconda 安装](https://www.anaconda.com/docs/getting-started/miniconda/install) · [NVIDIA WSL CUDA 指南](https://docs.nvidia.com/cuda/wsl-user-guide/index.html) · [模型卡](https://huggingface.co/FunAudioLLM/Fun-CosyVoice3-0.5B-2512)

在 PowerShell 进入现有发行版。

```powershell
wsl -d Ubuntu-22.04
```

在 Ubuntu 中安装。项目放在 Linux 家目录。下面的下载命令选用 Hugging Face 固定版本，跳过备用 RL 权重、TensorRT 导出图和批量 tokenizer，模型目录约 **5.05 GiB**。完整快照约 9.08 GiB。默认文字前端可用 WeText，`ttsfrd` 可以后装。 [模型文件树](https://huggingface.co/FunAudioLLM/Fun-CosyVoice3-0.5B-2512/tree/29e01c4e8d000f4bcd70751be16fa94bf3d85a18) · [加载器](https://github.com/QwenAudio/CosyVoice/blob/074ca6dc9e80a2f424f1f74b48bdd7d3fea531cc/cosyvoice/cli/cosyvoice.py#L190)

```bash
cd ~
sudo apt-get update
sudo apt-get install -y git build-essential sox libsox-dev libsndfile1
git clone https://github.com/QwenAudio/CosyVoice.git
cd CosyVoice
git checkout 074ca6dc9e80a2f424f1f74b48bdd7d3fea531cc
git submodule update --init --recursive

conda create -n cosyvoice3 python=3.10 -y
conda activate cosyvoice3
python -m pip install -r requirements.txt
python -c "import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available()); print(torch.cuda.get_device_name(0))"

python - <<'PY'
from huggingface_hub import snapshot_download

snapshot_download(
    "FunAudioLLM/Fun-CosyVoice3-0.5B-2512",
    revision="29e01c4e8d000f4bcd70751be16fa94bf3d85a18",
    local_dir="pretrained_models/Fun-CosyVoice3-0.5B",
    ignore_patterns=[
        "llm.rl.pt",
        "flow.decoder.estimator.fp32.onnx",
        "speech_tokenizer_v3.batch.onnx",
    ],
)
PY
```

如果 Hugging Face 下载缓慢，可把下载段替换为官方 ModelScope 下载。 [ModelScope 模型页](https://modelscope.cn/models/FunAudioLLM/Fun-CosyVoice3-0.5B-2512)

```bash
python -c "from modelscope import snapshot_download; snapshot_download('FunAudioLLM/Fun-CosyVoice3-0.5B-2512', local_dir='pretrained_models/Fun-CosyVoice3-0.5B')"
```

**8 GB 先跑一次短句。** 在仓库根目录使用自带的参考音频及其逐字稿。CosyVoice 3 的克隆提示词带 `<|endofprompt|>`。换成本人的录音时，提供与录音一致的逐字稿，参考音频选择约 3–8 秒、采样率至少 16 kHz 的干净片段，官方前端上限 30 秒。输出采样率 24 kHz。 [官方示例](https://github.com/QwenAudio/CosyVoice/blob/074ca6dc9e80a2f424f1f74b48bdd7d3fea531cc/example.py) · [官方模型配置](https://huggingface.co/FunAudioLLM/Fun-CosyVoice3-0.5B-2512/blob/main/cosyvoice3.yaml)

```bash
python - <<'PY'
import sys

sys.path.append("third_party/Matcha-TTS")
import torchaudio
from cosyvoice.cli.cosyvoice import AutoModel

model = AutoModel(
    model_dir="pretrained_models/Fun-CosyVoice3-0.5B",
    fp16=True,
)
reference = "You are a helpful assistant.<|endofprompt|>希望你以后能够做的比我还好呦。"
results = model.inference_zero_shot(
    "这是一段本地语音合成测试，今天我们先试听一句。",
    reference,
    "asset/zero_shot_prompt.wav",
    stream=False,
)
for index, result in enumerate(results):
    torchaudio.save(f"cosyvoice3-{index}.wav", result["tts_speech"], model.sample_rate)
PY
```

想用浏览器时，执行 `python webui.py --port 50000 --model_dir pretrained_models/Fun-CosyVoice3-0.5B`，从 Windows 打开 `http://127.0.0.1:50000`，选择“3s极速复刻”。当前官方 WebUI 直接实例化 `AutoModel(model_dir=...)`，默认为全精度且没有 `--fp16` 启动参数。要在 WebUI 沿用上述混合精度，可在本地将这一处改为 `AutoModel(model_dir=args.model_dir, fp16=True)`。方言或自然语言指令使用官方示例中的 `inference_instruct2`。 [官方 WebUI](https://github.com/QwenAudio/CosyVoice/blob/074ca6dc9e80a2f424f1f74b48bdd7d3fea531cc/webui.py#L154) · [指令示例](https://github.com/QwenAudio/CosyVoice/blob/074ca6dc9e80a2f424f1f74b48bdd7d3fea531cc/example.py)

## IndexTTS 2.5 在原生 Windows 部署

使用 `IndexTeam/IndexTTS-2.5` 权重和本次核查的官方主分支提交 `d9e41aa`，该提交包含参考音频处理和输出末尾修复。官方支持 Python 3.10/3.11，`uv` 锁定 PyTorch 2.8 的 CUDA 12.8 构建。只安装 WebUI 扩展，PyTorch 常规推理先用现有 NVIDIA 驱动。DeepSpeed、编译版 CUDA kernel、`accel` 和 `torch.compile` 留到基本推理通过后再评估。 [官方安装说明](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/README.md) · [依赖和 CUDA 源](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/pyproject.toml)

PowerShell 中选一个空间充足的父目录，再执行。

```powershell
git clone https://github.com/index-tts/index-tts.git
cd index-tts
git checkout d9e41aac89fd00b3d71497fddb287b7f24613712
uv sync --locked --python 3.10 --extra webui
uv run --no-sync python -c "import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available()); print(torch.cuda.get_device_name(0), torch.cuda.is_bf16_supported())"

uvx --from huggingface-hub hf download IndexTeam/IndexTTS-2.5 --revision c39ce5ba981572cb187443877ff559dfb246ce63 --local-dir checkpoints
uv run --no-sync webui.py --version 2.5 --model_dir checkpoints --host 127.0.0.1 --port 7860
```

模型快照约 **5.11 GiB**。按首启下载逻辑，辅助模型还会占约 5 GiB，主要是 W2V-BERT、BigVGAN、CAMPPlus 和 MaskGCT 下载文件，另有 Python 环境与工具缓存。可以为每套环境和权重预留约 20 GiB 可用磁盘作为安装预算。若 Hugging Face 下载慢，可用 `uvx modelscope download --model IndexTeam/IndexTTS-2.5 --local_dir checkpoints` 下载主权重。首启下载日志若卡在 Hugging Face，可在 PowerShell 设置 `$env:HF_ENDPOINT = "https://hf-mirror.com"` 后重启，或按启动日志定位缺少的辅助文件。 [主权重文件树](https://huggingface.co/IndexTeam/IndexTTS-2.5/tree/c39ce5ba981572cb187443877ff559dfb246ce63) · [首启下载代码](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/indextts/utils/model_download.py) · [W2V-BERT 文件树](https://huggingface.co/facebook/w2v-bert-2.0/tree/main)

打开 `http://127.0.0.1:7860`。上传 3–8 秒的说话人参考音频，语言选中文，先生成一句 20–40 字的台词。可以再换一段独立情绪参考音频测试。模型会把参考音频裁到约 15 秒，生成采样率为 22.05 kHz。看到低显存模式与跳过 QwenEmotion 的日志属于正常行为。程序化调用时用 `from indextts.infer_v2_5 import IndexTTS2`，显式设置 `use_bf16=True, use_cuda_kernel=False, use_qwen_emo=False`。 [官方推理实现](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/indextts/infer_v2_5.py)

如果使用文字描述来控制情绪，可在短句通过后单独用 `--qwen_emo` 试跑。这个开关会额外加载约 0.6B 参数的情绪模型，磁盘中的 FP16 权重约 1.11 GiB。若可用显存不足，继续用情绪向量或参考音频。在同一台机器运行时优先保留单卡单进程。 [情绪模型文件](https://huggingface.co/IndexTeam/IndexTTS-2.5/tree/main/qwen0.6bemo4-merge) · [WebUI 情绪开关](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/webui.py)

IndexTTS 提供官方代码与可本地下载的权重，采用 **bilibili Model Use License Agreement**。协议允许按条款免费使用，达到上月月活超过 1 亿或上年度收入超过人民币 1 亿元的适用门槛时需申请商业许可。协议还约束分发、衍生品和用模型改进其他 AI 模型。 [2.5 权重许可证](https://huggingface.co/IndexTeam/IndexTTS-2.5/blob/main/LICENSE) · [中文协议](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/LICENSE_ZH.txt)

## 如何确认这台机器的余量

两款引擎也可由 Windows 桌面版 VoiceStudio 集中安装和切换。它的 IndexTTS 适配器会主动加载文字情绪模型，因此这一路径的显存需求要单独验收。 [安装和切换方法](./2026-11-20-VoiceStudio项目.md#本机-rtx-4060-ti-8-gb-的模型管理) · [中文配音技巧](./2026-11-20-VoiceStudio项目.md#三个本地模型的中文配音技巧)

在另一个终端分别记录空闲、初始化、首次出音以及同音色第二次出音的显存。用一段 20–40 字的短句先出 WAV，再用平时的 1–2 分钟稿件逐句生成。检查是否有 CUDA OOM、输出是否有漏读，以及生成耗时是否满足自己的配音节奏。

```powershell
nvidia-smi --query-gpu=timestamp,memory.used,memory.free,utilization.gpu --format=csv -lms 200
```

遇到 OOM 时先关闭另一模型和占显存的应用，缩短参考和目标句。IndexTTS 可从 WebUI 高级选项将 `num_beams` 从 3 降到 1 后重新试听，采样结果可能变化。CosyVoice 先用上述混合精度短句入口，并核对是初始化阶段、ONNX provider 还是合成阶段报错。Windows 上若 CUDA 检测失败，先看诊断命令中的 `torch.version.cuda` 和 `torch.cuda.is_available()`。WSL 内驱动由 Windows 提供，ONNX 的 CUDA provider 仍需安装环境里的 CUDA/cuDNN 库能够被找到。 [IndexTTS WebUI 采样设置](https://github.com/index-tts/index-tts/blob/d9e41aac89fd00b3d71497fddb287b7f24613712/webui.py#L924) · [ONNX Runtime CUDA 要求](https://onnxruntime.ai/docs/execution-providers/CUDA-ExecutionProvider.html#requirements) · [NVIDIA WSL CUDA 指南](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)

中文情绪对白可先用 IndexTTS 的音色与情绪分别控制。偏中文方言、发音可修正、流式输入输出或希望采用 Apache-2.0 权重时，先试 CosyVoice。更广的能力和场景对比见 [文本转语音方案与选型对比](./2026-11-16-文本转语音方案与选型对比.md) 和 [IndexTTS、Qwen3-TTS 与 VoxCPM2 对比](./2026-10-20-IndexTTS-Qwen3-TTS与VoxCPM2对比.md)。
