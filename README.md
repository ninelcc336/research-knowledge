# 调研知识库

本仓库用于持续沉淀经过资料核查的调研结果。调研按主要研究领域归档；当前分类只反映已有内容，不构成封闭的分类清单。新增调研应优先复用合适分类，没有合适分类时再按内容创建新的语义分类。

## 调研目录

### Agent工程

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

- [AI生成产品的无限画布开源技术调研](./AI内容创作/2026-08-19-AI生成无限画布开源技术.md)（2026-08-19）：分 React、Vue、原生 Web 三个模块比较无限画布、节点工作流、协作编辑和图片/视频生成后端，并给出产品落地组合建议。
- [VOZEB PRO 项目调研](./AI内容创作/2026-08-18-VOZEB-PRO.md)（2026-08-18）：分析集成多模态生成、创作 Agent、素材管理和 SaaS 能力的内容生产平台。

### AI语音

- [VibeVoice 项目调研](./AI语音/2026-08-19-VibeVoice.md)（2026-08-19）：评估 VibeVoice 在长音频识别、多说话人语音和实时语音生成方面的能力与限制。
- [VoxCPM 项目调研报告](./AI语音/2026-08-19-VoxCPM.md)（2026-08-19）：评估 VoxCPM2 的多语言语音合成、音色设计、声音克隆及部署条件。

### 计算机视觉

- [Tesseract.js 项目调研](./计算机视觉/2026-08-21-Tesseract-js项目.md)（2026-08-21）：分析 Tesseract.js 的 WebAssembly OCR 架构、浏览器与 Node.js 用法、Worker 并发、性能边界及 PDF/手写/复杂文档限制。

### 设计工具

- [开源 Logo、产品图标工具及 Agent Skills 调研](./设计工具/2026-08-20-开源Logo与产品图标工具及Agent-Skills.md)（2026-08-20）：比较 Inkscape、Penpot、SVG-Edit、open-assets 等开源工具，以及 Logo、App Icon 和品牌一致性相关的 Agent Skills 与图标素材库。

### 图像处理

- [JPG、PNG 图片分层处理与开源方案](./图像处理/2026-08-23-JPG-PNG图片分层处理与开源方案.md)（2026-08-23）：说明 JPG/PNG 与图层工程文件的区别，比较 GIMP、Krita、OpenRaster、ImageMagick、Pillow、SAM、IOPaint、LaMa 等方案，并给出分层、加水印、抠图、移除和添加元素的组合架构。

## 归档约定

- 分类目录按调研内容动态生成，每篇调研只归入一个主分类，交叉领域使用标签表达。
- 文件名采用 `YYYY-MM-DD-主题.md`，日期取自正文明确记录的调研日期。
- 每篇调研使用统一的 YAML 元数据：`title`、`date`、`category` 和 `tags`。
- 索引按分类分组，组内按调研日期倒序排列；新增或移动调研后必须同步更新本页。
- Agent 的详细归档和内容保护规则见 [AGENTS.md](./AGENTS.md)。
