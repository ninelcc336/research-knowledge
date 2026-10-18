# 调研知识库

本仓库用于持续沉淀经过资料核查的调研结果。调研按主要研究领域归档；当前分类只反映已有内容，不构成封闭的分类清单。新增调研应优先复用合适分类，没有合适分类时再按内容创建新的语义分类。

## 调研目录

### Agent工程

- [Agent Browser、Playwright CLI/Skills 与 Playwright MCP 对比调研](./Agent工程/2026-10-17-Agent-Browser与Playwright-MCP对比.md)（2026-10-17）：核查 Playwright CLI、CLI Skills、Test Agents 和 MCP，并与 Agent Browser 比较通用网页操作、Playwright 测试闭环、上下文成本和客户端接入方式。
- [OpenDesign、OpenPencil 与 pen.dev 对比报告](./Agent工程/2026-08-22-OpenDesign-OpenPencil-pen.dev对比.md)（2026-08-22）：从定位、文件格式、Agent/MCP/CLI、扩展部署、选型场景和组合工作流比较三种工具，并明确它们分别适合代码媒体交付、Figma `.fig` 自动化和 `.pen` 设计协作。
- [nexu-io/open-design 项目调研](./Agent工程/2026-08-22-OpenDesign项目.md)（2026-08-22）：分析本地优先的 Agent 设计工作台、Skills/模板/设计系统/插件四层模型、MCP 与多 CLI 运行时、文件预览导出、隐私安全边界和自托管风险。
- [OpenPencil 开源设计编辑器与 Agent 工具链调研](./Agent工程/2026-08-22-OpenPencil.md)（2026-08-22）：分析 OpenPencil 的开源定位、`.fig`/`.pen` 文件支持、编辑器架构、AI/MCP、CLI、Vue SDK、协作方式、Figma 兼容性和落地风险。
- [emilkowalski/skills 调研](./Agent工程/2026-08-21-emilkowalski-skills.md)（2026-08-21）：分析面向 UI 设计工程、Web/Expo 动画、动效审查、原型探索和前端依赖选择的 Agent Skills 集合，并评估其工作流、安装方式与适用边界。
- [nextlevelbuilder/ui-ux-pro-max-skill 调研](./Agent工程/2026-08-21-ui-ux-pro-max-skill.md)（2026-08-21）：分析面向多种 AI 编程助手的 UI/UX 设计智能 Skill，涵盖结构化设计目录、BM25 检索、设计系统生成、CLI 安装、质量门禁、版本漂移与视觉回归风险。
- [Nutlope/hallmark 设计 Skill 调研](./Agent工程/2026-08-21-Nutlope-hallmark设计Skill.md)（2026-08-21）：分析面向 Claude Code、Cursor 和 Codex 的反 AI 模板设计 Skill，涵盖宏结构、主题路由、57+1（含 38a）项审查门禁、安装方式、能力边界与当前文档风险。
- [Agent 项目级记忆方案调研](./Agent工程/2026-08-20-Agent项目级记忆方案.md)（2026-08-20）：比较 Codex/ChatGPT、Claude Code、Pi、OpenCode 和 WorkBuddy 的项目规则、自动记忆、会话状态、作用域与跨 Agent 落地方案。
- [Agent 连接数据库方案调研](./Agent工程/2026-08-20-Agent连接数据库方案.md)（2026-08-20）：比较直接驱动、MCP 数据库工具、Text-to-SQL 框架、语义层和厂商 MCP，并给出生产权限、审批、隔离与审计基线。
- [pen.dev VS Code 插件、Windows 桌面端、MCP 与 CLI 区别调研](./Agent工程/2026-08-20-pen.dev客户端与集成方式对比.md)（2026-08-20）：区分 IDE 插件、Windows 桌面应用、MCP 协议层和 CLI 客户端的职责、能力、认证方式与适用场景，并标注官方 Windows 可用性页面之间的冲突。
- [Codex 在 VS Code 中的使用能力调研](./Agent工程/2026-08-20-Codex在VS-Code中的使用能力.md)（2026-08-20）：确认 Codex 提供 VS Code 官方扩展，并支持编辑器上下文、修改审查和较大任务委托。
- [适合 Agent 协作前端设计的工具调研](./Agent工程/2026-08-18-Agent协作前端设计工具.md)（2026-08-18）：比较适合 Agent 参与原型、界面设计和前端交付的工具及协作流程。
- [mattpocock/skills 调研](./Agent工程/2026-08-01-mattpocock-skills.md)（2026-08-01）：分析面向 Claude Code、Codex 等编程代理的工程技能集合、调用方式与适用流程。
- [Waza 仓库调研](./Agent工程/2026-08-01-Waza.md)（2026-08-01）：介绍可安装到 AI 编程代理中的工程工作流技能包及其使用边界。

### AI内容创作

- [目标抖音视频的 Code Agent 制作技术调研](./AI内容创作/2026-10-17-目标抖音视频的Code-Agent制作技术调研.md)（2026-10-17）：单独核查“claude，我妈问我在干嘛”视频的画面层、Code Agent 职责边界、Remotion/3D/剪辑产线、时间线工程和 Windows 最小试做方案。
- [Vibe知识大赏短视频制作流程与风格拆解](./AI内容创作/2026-10-17-Vibe知识大赏短视频制作流程与风格拆解.md)（2026-10-17）：归纳原生代码动画、AI 素材剪辑和混合制作三条产线，拆解“先演后命名”、字幕与可见动作对应、分镜和渲染 QA，并注明中文平台案例的证据边界。
- [Claude Opus 5.5 代码视频与 Agent Skill 仓库调研](./AI内容创作/2026-10-15-Claude-Opus-5-5代码视频与Agent-Skill仓库.md)（2026-10-15）：核查四个已定位仓库的 703 条提示词、Lemo 制片 Skill、59 个案例和 117 项工具索引，区分代码渲染与混合媒体流程，并分析安装、Windows 兼容性和复现边界；单数同名目标待确认。
- [JianYing Editor Skill 仓库与使用方式调研](./AI内容创作/2026-10-13-Jianying-Editor-Skill仓库调研.md)（2026-10-13）：核查同名仓库、草稿生成原理、Codex/通用项目安装、Windows 与 macOS 差异、最小验证流程、自动导出限制和多模态视频理解边界，建议先做可编辑粗剪再逐步增加自动化。
- [AI混剪视频与口型、语气、语速同步开源方案调研](./AI内容创作/2026-09-04-AI混剪视频与口型语气同步开源方案.md)（2026-09-04）：评估直播高光发现、词级对齐、音色/语气控制、口型重绘和多片段自然拼接，结论是采用 Clips Kitty/FunClip、WhisperX/FunASR、CosyVoice/OpenVoice、MuseTalk 与 FFmpeg 的组合式管线。
- [AI生成产品的无限画布开源技术调研](./AI内容创作/2026-08-19-AI生成无限画布开源技术.md)（2026-08-19）：分 React、Vue、原生 Web 三个模块比较无限画布、节点工作流、协作编辑和图片/视频生成后端，并给出产品落地组合建议。
- [VOZEB PRO 项目调研](./AI内容创作/2026-08-18-VOZEB-PRO.md)（2026-08-18）：分析集成多模态生成、创作 Agent、素材管理和 SaaS 能力的内容生产平台。

### AI语音

- [MiniMax T2A HTTP 语音合成 API 接入与计费调研](./AI语音/2026-10-10-MiniMax-T2A-HTTP-API.md)（2026-10-10）：核查 HTTP TTS 的接入、流式响应、Skill/CLI/MCP 集成与 API 计费，区分 Audio 网页声贝、开放平台余额、语音资源包和 M Plan 额度。
- [VibeVoice 项目调研](./AI语音/2026-08-19-VibeVoice.md)（2026-08-19）：评估 VibeVoice 在长音频识别、多说话人语音和实时语音生成方面的能力与限制。
- [VoxCPM 项目调研报告](./AI语音/2026-08-19-VoxCPM.md)（2026-08-19）：评估 VoxCPM2 的多语言语音合成、音色设计、声音克隆及部署条件。

### 计算机视觉

- [Tesseract.js 项目调研](./计算机视觉/2026-08-21-Tesseract-js项目.md)（2026-08-21）：分析 Tesseract.js 的 WebAssembly OCR 架构、浏览器与 Node.js 用法、Worker 并发、性能边界及 PDF/手写/复杂文档限制。

### 设计工具

- [Morphicons 官网与 SVG 图标变形库调研](./设计工具/2026-09-17-Morphicons.md)（2026-09-17）：确认 morphicons.com 是描边 SVG 图标万能变形库的 Playground，适合 Lucide/Tabler 等状态切换微动效，不支持 Font Awesome 等填充图标。
- [开源 Logo、产品图标工具及 Agent Skills 调研](./设计工具/2026-08-20-开源Logo与产品图标工具及Agent-Skills.md)（2026-08-20）：比较 Inkscape、Penpot、SVG-Edit、open-assets 等开源工具，以及 Logo、App Icon 和品牌一致性相关的 Agent Skills 与图标素材库。

### 图像处理

- [JPG、PNG 图片分层处理与开源方案](./图像处理/2026-08-23-JPG-PNG图片分层处理与开源方案.md)（2026-08-23）：说明 JPG/PNG 与图层工程文件的区别，比较 GIMP、Krita、OpenRaster、ImageMagick、Pillow、SAM、IOPaint、LaMa 等方案，并给出分层、加水印、抠图、移除和添加元素的组合架构。

### 模拟器

- [热门好用模拟器项目比较](./模拟器/2026-09-02-热门好用模拟器项目比较.md)（2026-09-02）：按 Android 开发测试、Linux Android 运行环境、游戏主机和跨架构仿真比较 Android Emulator、Genymotion、Waydroid、RetroArch、Dolphin、PCSX2、PPSSPP、RPCS3、melonDS 与 QEMU，并给出目标导向的选型建议。

### 网站部署

- [国内静态网站托管与视频播放方案调研](./网站部署/2026-10-17-国内静态网站托管与视频播放方案.md)（2026-10-17）：核查 Gitee Pages 与 CODING 现状，比较国内 COS/OSS、海外 Pages、服务器和视频分发方案，给出域名备案、播放成本及文档视频站的实施建议。

### 桌面应用

- [Electron、Vue3 与 Python 桌面应用更新与热更新方案调研](./桌面应用/2026-08-24-Electron-Vue3-Python桌面应用更新与热更新方案.md)（2026-08-24）：比较完整安装包、Electron 差分更新、Vue renderer 组件更新和 Python 服务独立更新，并给出签名、回滚、阿里云/CDN 成本核算和分阶段落地方案。

## 归档约定

- 分类目录按调研内容动态生成，每篇调研只归入一个主分类，交叉领域使用标签表达。
- 文件名采用 `YYYY-MM-DD-主题.md`，日期取自正文明确记录的调研日期。
- 每篇调研使用统一的 YAML 元数据：`title`、`date`、`category` 和 `tags`。
- 索引按分类分组，组内按调研日期倒序排列；新增或移动调研后必须同步更新本页。
- Agent 的详细归档和内容保护规则见 [AGENTS.md](./AGENTS.md)。
