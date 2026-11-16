---
title: "VoiceStudio 项目调研"
date: 2026-11-16
category: "AI语音"
tags:
  - 语音合成
  - 声音克隆
  - 视频配音
  - 本地部署
  - 多引擎
  - 情绪控制
  - MCP
  - 模型许可
---

# VoiceStudio 项目调研

调研日期：2026-11-16。研究对象为 [debpalash/VoiceStudio](https://github.com/debpalash/VoiceStudio)。GitHub 存在同名项目，本次根据搜索结果与本库旧文确认研究这一主仓库，而非中文翻译分支或其他同名工具。本次读取 GitHub API、官方文档、关键源码、发布记录、问题记录和上游模型卡，未安装应用、下载模型、运行语音推理或试听生成结果。

源码快照固定为 `6fe6f4dd62b477ab5fbf4d94500e3c0298d05232`，提交时间为 2026-11-16 13:06:14 UTC。读取时最新稳定版为 **v0.5.8**（11 月 12 日发布），最新已发布应用预览版为 **v0.5.9-beta.3**（11 月 16 日发布）；主分支已提交 beta.5 的发布准备，不能据此认定 beta.5 安装包已发布。下文将稳定版、后续 Beta 和主分支区分。[S1][S2]

本次更新：整合 2026-10-26 的项目概览与 2026-11-16 的版本核查，保留声音工作流、转录、远程部署和 Agent 接入说明，更新 Windows 加速路径、版本与默认模型许可。本文只保留一份当前调研；历史版本可从 Git 中查看。合并整理未重新联网核查，不改变上述资料快照与验证范围。

## 1. 结论与使用建议

**VoiceStudio 值得作为本地语音制作工作台试用。它的价值在于把模型管理、声音库、转录、配音、长文制作和自动化放进同一个应用；最终音质、克隆相似度、语言和情绪能力仍由具体引擎决定。** “本地 ElevenLabs 替代品”是项目定位，尚不能作为音质、易用性或服务水平相当的证据。

针对中文口播和视频制作，建议如下：

| 需求 | 建议试用方式 | 关键判断 |
| --- | --- | --- |
| 用自己的声音读中文稿 | VoiceStudio + VoxCPM2，对照 IndexTTS 2.5 | 先比较清晰度、相似度、多音字、中英混读与长句稳定性 |
| 有情绪的中文角色对白 | 优先试 IndexTTS 2.5，或 VoxCPM2/CosyVoice 3 的文字表演指令 | 不同引擎控制方式不同，不能通用复制情绪标签 |
| 没有录音、想描述一种新音色 | 试 VoxCPM2；Apple Silicon 可另试 Qwen3-TTS VoiceDesign | 默认 OmniVoice 的描述映射主要是固定属性，不能等同于任意自然语言设计 |
| 原视频翻译后重新配音 | 使用 Dubbing，先做 1–3 分钟片段 | 必须检查转录、翻译、说话人、时长与背景混音 |
| AMD/Intel Windows 电脑试本地生成 | 先检查 OmniVoice GGUF/Vulkan；商业用途另选许可合适的模型 | Vulkan 是特定原生引擎路径，不代表全部 PyTorch 引擎都用上显卡 |
| 程序批量生成旁白 | 优先本地 HTTP API；Agent 交互可用 MCP | 输出文件路径便于后续剪辑，避免把大段音频 Base64 放进 Agent 上下文 |
| 想免维护，马上稳定出稿 | 继续把云端 TTS 作为对照 | 本地方案仍有首次依赖、模型下载、磁盘与显卡兼容成本 |

**商业内容制作最需要先确认默认模型许可：VoiceStudio 应用允许按 AGPL 条款商用，但默认 OmniVoice 预训练权重为 CC-BY-NC，不能直接当作商业配音默认引擎。购买 VoiceStudio Pro 也不替代上游权重许可。** VoxCPM2 的上游模型卡明确声明 Apache-2.0、允许商用，是值得优先核对具体依赖与素材权利的候选；这不是对整个工作流的统一授权。[S12][S13][S14]

## 2. 仓库与版本现状

读取时 GitHub API 显示约 **11.17 万 Stars、1.24 万 Forks**，仓库未归档，当天仍有推送。热度说明关注度，不能代替可用性验证。应用仍处于 `0.5.x`，更新频繁，也持续处理安装、驱动、模型加载和长任务问题。[S1][S2][S18]

| 核查项 | 当前证据 | 使用影响 |
| --- | --- | --- |
| v0.5.8 稳定版 | 已有 Windows x64/ARM64、macOS、Linux 和手机构建附件 | 普通试用优先从稳定版开始；ARM64 和手机仍有各自实验/预览状态 |
| OmniVoice GGUF/Vulkan | 已进入 v0.5.8，支持 Windows/Linux 的原生 GPU 发现与量化选择 | 低显存及非 NVIDIA 用户有更多路径 |
| GGUF 音色设计、非语言标签、解码步数 | v0.5.8 引擎文档仍写“不支持音色设计”；beta.2 发布记录与主分支已补充 | 需要这些能力时核对 Beta 版本，不能套用到稳定版 |
| 配音崩溃后的已完成分段恢复 | beta.2/主分支有新增实现和适用条件 | 并非所有引擎、模型来源都能复用；不要理解为任意任务恢复 |
| Dubbing 直接导出 TXT | beta.3 发布记录新增 | v0.5.8 不应照最新版文档期待这一入口 |
| MCP 权限和长录音超时 | beta.3 包含多项修复 | 远程共享或长转录任务需要特别核对实际安装版本 |

2026-10-26 的初次调研对应 v0.5.6/v0.5.7 Beta 时期；旧文对 Windows AMD/Intel 加速路径的概括、许可库存数量和发布版本已由本次核查结果替换。

## 3. 它具体能完成哪些工作

### 3.1 文字转语音与声音库

可以上传参考录音建立声音，输入新文本合成，保存声音档案并在配音、长文和 API 中复用。应用克隆入口接受最长 75 秒参考片段，指南推荐 5–15 秒干净人声；默认 OmniVoice 的实际建议更偏向 3–10 秒，其超过 20 秒的长录音会选取约 15 秒语音窗口，VoxCPM2 则通常取裁剪后前 30 秒。**入口允许的长度不等于模型使用的长度。** 参考转写必须对应实际使用的录音片段。[S4][S5]

跨语言克隆可以让同一个音色读另一种语言，但选择输出语言不会自动翻译输入文本。新稿需要先写成目标语言。参考中的背景音乐、混响、噪声和表演方式也会影响克隆，不能只追求录音长度。

### 3.2 音色设计与情绪控制

默认设计界面会将描述映射为性别、年龄、音高、风格、口音等属性，并保存种子；VoxCPM2 可在无参考音频时采用原生描述设计，有参考时也可用文字控制表达。IndexTTS 2.5 提供八维情绪权重和自然语言情绪描述；其情绪参考录音还存在 API 与界面能力差异。[S5][S6][S7]

通用 `[pause]`、`[pause 500ms]` 等停顿由应用拼接静音实现。笑声、叹气、呼吸等标签要按引擎检查；不能把 ElevenLabs 的 `[excited]`、`[whispers]` 整段原样搬来，未识别标签可能被读出来。语音引擎有无“表演方向”能力，可以通过 `GET /engines/tts` 的 `expression_caps` 查询。[S7]

### 3.3 视频配音与字幕

主要流程是：

`导入音视频/字幕 → 人声与背景处理 → 转录及说话人识别 → 翻译并校对 → 分配声音 → 分段合成 → 时长适配与混音 → 导出`

支持源文和译文逐段编辑、给说话人绑定声音、改动后局部重生成，以及 WAV/MP3、MP4、多语言音轨、SRT/VTT/ASS、分段或分轨 ZIP 等导出。可保留分离后的背景声，调节混音音量，也能烧录字幕。Smart Fit 会在共享视频时间线上处理重叠说话人和时长适配。[S8]

**时间适配与人物嘴部重绘应分开理解。** 文档中虽然出现 Lip sync 选项名，但明确不承诺音素级口型同步；核查到的配音路径包括语音变速、片段摆放和视频时长调整。这不足以证明它提供 MuseTalk/Wav2Lip 类嘴部画面生成，不能据此承诺外国人物说中文时嘴形准确。[S8]

翻译既可以选择 Argos/NLLB 等本地组件，也可以配置在线翻译、LLM 或已安装的 Agent CLI。云端翻译会改变纯本地数据路径；“本地优先”不代表任何配置下都不联网。翻译文档部分安装说明仍提到旧 MSI/只读环境，与当前 Electron 首次引导安装运行时的说明不一致；应以对应版本的实际界面和运行时状态为准，不能把旧安装限制推广到当前所有安装包。[S8][S9]

### 3.4 长文、有声书与工作流

Stories/Audiobook 用于长篇文本、分角色和章节制作；应用提供缓存、分段及恢复等机制，但仍需检查角色一致性、漏句、重复和章节衔接。适合把已经审定的稿件转为声音，不代表自动完成整本书的编辑和质检。

工作流画布在 v0.5.8 文档中已存在，可执行输入、Speak、Convert、Transcribe、Translate、Normalize、Condition、End。支持条件分支、串行处理和完成步骤的检查点；文档限制每次最多 50 个输入、单稿 20,000 字符、单音频 64 MiB。**Agent、Call 节点仍是本地草稿，包含它们的流程不能运行**；不能把画布展示当成已实现完整语音 Agent 或电话机器人。源码 `compileWorkflow` 与串行执行实现也能对应这些约束。[S10]

### 3.5 独立转录与听写

除配音链路中的 ASR，应用也提供独立转录和悬浮听写入口。引擎目录列出 WhisperX、Faster-Whisper、MLX Whisper、PyTorch Whisper、Parakeet、FunASR 和 sherpa-onnx 等不同路径。词级时间戳、说话人分离、实时输出和语言覆盖要分别确认，不能把其中一个引擎的能力套给全部转录任务。[S4][S11]

## 4. 中文创作应怎样选择引擎

| 引擎/路径 | 已核查的用途 | 部署与控制边界 |
| --- | --- | --- |
| 默认 OmniVoice | 广语言零样本克隆、24 kHz 单声道、固定属性音色设计 | PyTorch 独显建议约 6 GB VRAM 起步；权重非商业许可 |
| OmniVoice GGUF | 同系模型的原生量化推理，适合低显存、CPU、AMD/Intel Vulkan | v0.5.8 已有 Vulkan；音色设计等新增能力要核对后续 Beta；量化不消除原权重许可 |
| VoxCPM2 | 中文等 30 种语言、48 kHz 输出、描述音色设计、可控克隆 | 可在独立环境安装；CUDA/MPS/CPU 和部分 Linux ROCm 路径按硬件选择；48 kHz 本身不证明听感最好 |
| IndexTTS 2.5 | 中英日西阿五语克隆，情绪向量、描述和参考控制 | 独立 Python 环境；安装前检查至少约 12 GB 可用空间；自定义模型许可 |
| CosyVoice 3 | 中文及多语言克隆、文字表达指令 | 与 CosyVoice 3 GGUF 是不同路径；GGUF 原生运行时目前需自行构建，并非安装应用就全部可用 |
| Qwen3-TTS | Apple Silicon 的 MLX Base 克隆/VoiceDesign；另有 CUDA Base 适配器 | MLX 目录有明确平台限制；CUDA 路径需要本地 0.6B Base 检查点、`faster-qwen3-tts` 和手动配置，不能理解为 Windows 全功能一键接入 |

“646 种语言”明确属于 **OmniVoice 语言目录**，并非所有 TTS、ASR、翻译、说话人和情绪功能的共同覆盖。例如 VoxCPM2 文档列 30 语，IndexTTS 2.5 列 5 语。MCP 文档转录工具旁也写了 646，不能因此认定所选 Whisper、Parakeet 等 ASR 都支持这个数量；实际应按 ASR 引擎核对。[S4][S5][S6][S11]

本次未做声音盲测，不能给出“OmniVoice/VoxCPM2/IndexTTS 谁最像本人”的排名。建议用同一份本人参考录音和同一段稿件，逐句比较；也可对照本库[文本转语音选型](./2026-11-16-文本转语音方案与选型对比.md)中的云端方案。

VoiceStudio 研究的是模型之上的应用与集成工作流；模型本身的比较可继续看 [IndexTTS、Qwen3-TTS 与 VoxCPM2 对比](./2026-10-20-IndexTTS-Qwen3-TTS与VoxCPM2对比.md)。需要剪辑、人物口型重绘和混剪产线时，可衔接[视频口型与混剪方案](../AI内容创作/2026-09-04-AI混剪视频与口型语气同步开源方案.md)。

## 5. Windows 部署与硬件成本

普通 Windows 用户优先下载 `VoiceStudio-0.5.8-win-x64.exe`。当前桌面应用是 **Electron + React + Python/FastAPI**，旧 Tauri/MSI 已退出维护。安装包首次启动会引导准备 Python、FFmpeg、依赖和所需权重，普通安装不要求先配齐源码编译工具。[S2][S3]

| 机器情况 | 实际建议 |
| --- | --- |
| Windows 10/11 x64 + NVIDIA | 默认优先 CUDA，检查显卡架构与驱动；旧 GTX 不一定被新 PyTorch wheel 支持 |
| Windows + AMD/Intel | 默认 PyTorch 可能使用 CPU；优先查看特定 GGUF/Vulkan 路径，不能仅凭设置页显示显卡认定全部引擎加速 |
| Windows 11 + 指定 AMD GPU | 有实验性 ROCm 安装，须按架构、驱动及运行时配套；部分 RDNA 1/2 已有测试记录，但不能推广到所有型号 |
| Windows + Intel Arc | XPU 路径仍实验，需对应 PyTorch 构建；独立环境的其他模型不自动继承 |
| Windows ARM64 | 原生 UI，后端为模拟执行的 x64 Python，官方文档标为 CPU、实验性；不能期待 Snapdragon NPU 加速 |
| Apple Silicon | 可使用 MPS/MLX，模型各有差异；Intel Mac 安装包主要作为远程后端客户端 |

Windows 指南给出约 10 GB 初始空闲空间要求，但它不是“安装任意模型组合”的总容量上限。多个隔离引擎环境、模型、缓存、转录和配音临时文件会继续增长；建议把运行环境和模型位置提前选到空间充足的盘。没有必要为调研直接安装整套开发工具。

官方 `benchmarks.md` 的结果表目前仍是 **No verified rows yet**，所以本报告不填造“某显卡几秒生成一分钟”的数据。GGUF 文档有约 659 MB/945 MB 等量化内存估值，也不应当作包含 Tokenizer、推理缓存、ASR 和并行工作负载的整机峰值。[S3][S5][S17]

## 6. 技术架构与自动化接口

从维护中的目录和源码可见，Electron 主进程管理窗口、运行时、后端生命周期和更新，React renderer 提供工作区；FastAPI 后端处理声音档案、任务、模型调度、媒体加工和 HTTP/WebSocket；不同模型通过适配器或独立 sidecar 进程及环境运行，媒体导出使用 FFmpeg，持久化涉及 SQLite/Alembic。[S15]

这种做法能隔离不同引擎的 Python/Torch 依赖，代价是更多下载、独立环境和设备兼容问题。安装 VoiceStudio 不等于所有模型已安装或都能在当前 GPU 上工作。

**本机与远程运行是两种部署方式。** 本机方式由应用准备运行环境、按需下载模型，再使用本机 CPU/GPU 推理；远程方式是在另一台电脑或云 GPU 服务器部署 VoiceStudio 后端和所需模型，再让客户端连接。项目还提供 Docker/服务端部署指南；并非把任意模型文件上传服务器就能直接调用，第三方服务也需要对应接口适配。模型与依赖就绪后，本地生成可离线运行；选择在线翻译或远程服务时，相应环节需要网络。[S1][S3][S11][S19]

源码开发与普通安装包使用也应区分：Windows 开发指南另列 Git、Bun、uv、Node.js、Rust/Cargo 和 C++ Build Tools；仅使用安装包不需要先装齐这些开发工具。项目还维护 iOS/Android 原生客户端，当前列为预览版；手机运行和配对电脑的能力应按移动端指南单独核对。[S3][S19]

自动化入口主要有三类：

- **CLI**：安装程序提供 `voicestudio` 命令，适合脚本和终端工作流。具体子命令应查当前安装版本的 `--help`。
- **HTTP/WebSocket**：本地后端通常使用 3900 端口，提供 `/generate`、OpenAI 兼容语音/转录路由和实时转录等接口；具体模型流式能力不能从 API 形状推断。
- **MCP**：运行中的后端挂载 `http://127.0.0.1:3900/mcp/`，源码与文档可核对 `generate_speech`、`clone_voice`、`describe_voice`、`design_voice`、`transcribe`、声音列表和健康检查等工具。[S11]

MCP 还支持按客户端标识绑定声音：调用时显式指定的 `profile_id` 优先，其次是该 Agent 的声音绑定，最后使用默认声音。这样多个 Agent 可以共用后端并选择各自声音，无需为每个 Agent 复制一套模型。[S11]

Agent 制作视频旁白时，建议设置 `OMNIVOICE_MCP_OUTPUT_MODE=files` 和共享可见目录 `OMNIVOICE_MCP_BASE_PATH`，取得音频文件路径交给剪辑/渲染工具。它提供语音工具，不会自动把视频脚本、画面、字幕、口型和最终审片全部做完。共享后端时，按照对应版本的 PIN/API key 与远程权限说明配置；beta.3 发布记录确有 MCP/分享权限修复，部署判断应结合该事实。[S2][S11]

局域网 HTTP 会明文传输 PIN 和任务内容；在共享或不受信任网络使用 HTTPS 或加密隧道。通过本机反向代理转发时还须由代理负责认证，不能以为开了 HTTPS 或应用分享 PIN 就自动保护了代理入口。[S11]

## 7. 免费版、Pro、Cloud 与许可

### 7.1 产品费用

README 写明本地创作功能免费、使用自有硬件、不要求账号。Pro 是可选的闭源构建，介绍包含商业应用许可、单用户最多三台同时使用设备和远程 GPU 分派等；其具体销售价格及权益应以购买时官方页面和合同为准。本次没有购买或实测 Pro。[S1][S12]

README 对 Cloud 的描述仍为 **coming soon / not available yet / early access**。仓库已有账号、同步和 Cloud 设置代码，不足以证明托管 TTS/配音 API 已正式开放；本报告不提供未经核实的 Cloud 单价或可用性承诺。[S1]

### 7.2 应用代码与模型分别判断

| 层次 | 本次核查结果 | 对使用的影响 |
| --- | --- | --- |
| VoiceStudio 自有应用代码 | AGPL-3.0-only；许可说明明确包含商业和内部业务使用 | 不能写成“商用必须买 Pro”；分发、修改和网络提供服务按 AGPL 条款判断 |
| 默认 OmniVoice | 上游区分 Apache-2.0 代码与 CC-BY-NC 预训练权重，Tokenizer 另有条款 | 代码开源不等于权重可商用；不能直接用于付费客户配音 |
| OmniVoice GGUF | 转换权重保留其来源的非商业限制 | 换量化运行时不会获得商业授权 |
| VoxCPM2 | 上游模型卡写明 Apache-2.0、free for commercial use | 可优先检查具体模型、依赖和参考声音后用于商业方案评估 |
| IndexTTS 2.5 | bilibili Model Use License；超过特定规模需单独许可 | 用户或关联方上一自然月 MAU 超 1 亿、或上一自然年营收超 10 亿元人民币，须取得单独书面许可；另有用途和下游义务 |
| 声音、配音文本、组件及输出 | 有独立权利与条件 | 应用许可和 Pro 均不替代这些来源的许可 |

上述条件以各上游原协议及实际所用模型版本为准。[S6][S12][S13][S14]

另发现一处可复核的文档滞后：`docs/licensing/model-inventory.md` 写“63 个模型仓库、7 类动态资产”，但本次固定提交的 `backend/config/model_licenses.json` 实际包含 **73 条模型记录、7 类动态资产**；`commercial_use: true` 的记录数为 **0**。这里 false 同时包括已知非商业和未审完，**不能解释成 73 个模型全部禁止商用**。这体现的是项目自身尚未给出整套模型的商业审查完成承诺。[S16]

## 8. 维护质量与当前未决问题

仓库有 Python、renderer、桌面和打包等多类检查，也有独立环境与错误恢复机制；这些是工程投入的证据，不代表所有真实模型、显卡和长素材组合都已验证。本次抽查的问题包括：

- **#4200**：用户报告 v0.5.8 在 Windows AMD 独显 + Ryzen 核显组合上 ROCm 校验崩溃，附有复现和 DLL 测试。它是特定组合的报告，不能推广成“AMD 都不能用”，也不能把报告者的替换 DLL 做法当官方通用修复。
- **#4199**：CosyVoice 3 的参考条件生成被报告在不同种子下加词或漏句。记录明确说维护者尚未独立听验/模型复现，模型无关路由测试通过也不能证明输出内容正确。
- **#4222**：Docker Web UI 的下载按钮禁用报告仍开放。它提醒服务端 UI/导出也需单独验收，不能仅因桌面包可启动便假定一致。

以上是读取时的问题状态与证据层级，不是本次独立复现。建议以版本、引擎、权重、硬件、驱动和实际错误定位问题，避免将模型问题、桌面壳问题和旧 Tauri 问题混为一谈。[S18]

## 9. 最小试用与验收方式

1. 安装稳定版，记录版本、GPU/驱动、内存、模型目录，先只装一个 TTS 引擎及必要 ASR。商业试做优先评估许可合适的候选，避免默认 OmniVoice 直接进入付费生产。
2. 取一段 5–15 秒本人干净录音，并保存逐字对应转写。用约 200–300 字真实中文稿，包含人名、多音字、英文、数字、百分比、长短句，分别生成三次。
3. 检查是否漏句、重复、额外发声、音色漂移、音量异常和长尾静音，记录冷启动、热生成耗时及显存/内存；不要只听第一句。
4. 如需情绪，再比较 IndexTTS 2.5 的描述/向量与 VoxCPM2 的指令，不预设控制强度跨引擎等效。
5. 如需视频，使用 1–3 分钟、包含两名说话人和背景音乐的片段，检查分人、翻译长度、重叠对白、重生成和导出，确认能继续进入现有剪辑工具。
6. 如需自动化，用 API/MCP 生成文件、交给下游渲染器；验证连续任务与失败恢复后再扩大量，不把 UI 工作流草稿节点计入交付能力。

**当前建议：想在自己电脑集中管理中文配音流程，可以试 VoiceStudio；想判断中文声音质量，先选定 VoxCPM2/IndexTTS 等具体引擎做同稿对照。它适合作为语音制作环节，尚无证据支撑“下载即可替代商业语音服务与整套视频后期”。**

## 10. 主要来源

仓库文档和源码链接尽量固定到本次提交；Release 和 Issue 链接用于查看其各自版本和后续状态。上游模型卡为本次读取的公开页面，未来可能变化。

- [S1：主仓库与 README](https://github.com/debpalash/VoiceStudio)；[本次提交](https://github.com/debpalash/VoiceStudio/commit/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232)。仓库统计来自 GitHub API。
- [S2：v0.5.8 稳定版](https://github.com/debpalash/VoiceStudio/releases/tag/v0.5.8)、[beta.2](https://github.com/debpalash/VoiceStudio/releases/tag/v0.5.9-beta.2)、[beta.3](https://github.com/debpalash/VoiceStudio/releases/tag/v0.5.9-beta.3)。
- [S3：Windows 指南](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/install/windows.md)、[Windows ROCm](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/install/windows-rocm.md)。
- [S4：引擎总表与参考录音策略](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/engines/README.md)、[语言目录范围](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/languages.md)。
- [S5：OmniVoice](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/engines/omnivoice.md)、[GGUF 主分支](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/engines/omnivoice-gguf.md)、[GGUF v0.5.8](https://github.com/debpalash/VoiceStudio/blob/v0.5.8/docs/engines/omnivoice-gguf.md)、[VoxCPM2](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/engines/voxcpm2.md)。
- [S6：IndexTTS 2.5 集成](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/engines/indextts.md)、[上游模型许可证](https://huggingface.co/IndexTeam/IndexTTS-2.5/blob/main/LICENSE)。
- [S7：表达能力与标签](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/expressive-speech.md)、[设计界面](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/electron-voice-design.md)。
- [S8：配音工作区及验证边界](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/electron-dubbing.md)、[导出](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/dubbing/export.md)、[时长适配源码](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/backend/api/routers/dub_generate.py)。
- [S9：翻译引擎](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/dubbing/translation-engines.md)、[隐私与可选遥测](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/electron-privacy.md)。
- [S10：v0.5.8 工作流说明](https://github.com/debpalash/VoiceStudio/blob/v0.5.8/docs/electron-workflows.md)、[执行实现](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/electron/src/renderer/src/features/workflows/workflow-runtime.ts)。
- [S11：MCP](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/mcp.md)、[MCP 实现](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/backend/mcp_server.py)、[本地语音协议](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/speech-platform.md)。
- [S12：应用许可与 Pro 的范围](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/LICENSE-NOTICE.md)。
- [S13：OmniVoice 上游模型卡](https://huggingface.co/k2-fsa/OmniVoice)。
- [S14：VoxCPM2 上游模型卡](https://huggingface.co/openbmb/VoxCPM2)。
- [S15：架构目录](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/STRUCTURE.md)、[React/Electron 依赖](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/electron/package.json)、[MLX/Qwen 路径](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/engines/mlx-audio.md)、[CosyVoice GGUF](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/engines/cosyvoice-gguf.md)。
- [S16：许可库存文档](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/licensing/model-inventory.md)、[实际 JSON 记录](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/backend/config/model_licenses.json)。
- [S17：官方性能基准表](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/benchmarks.md)。
- [S18：AMD 双 GPU 问题 #4200](https://github.com/debpalash/VoiceStudio/issues/4200)、[CosyVoice 加漏词报告 #4199](https://github.com/debpalash/VoiceStudio/issues/4199)、[Docker 下载按钮 #4222](https://github.com/debpalash/VoiceStudio/issues/4222)。
- [S19：Docker/服务端部署指南](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/install/docker.md)、[移动端状态与配对方式](https://github.com/debpalash/VoiceStudio/blob/6fe6f4dd62b477ab5fbf4d94500e3c0298d05232/docs/mobile.md)。Docker 入口沿用 2026-10-26 已核查内容；移动端指南也在 2026-11-16 读取，本文未实测安装或配对。
