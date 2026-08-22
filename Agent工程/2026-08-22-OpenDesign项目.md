---
title: "nexu-io/open-design 项目调研"
date: 2026-08-22
category: "Agent工程"
tags:
  - OpenDesign
  - Coding Agents
  - Agent Skills
  - MCP
  - 本地优先
  - BYOK
  - 设计系统
  - Electron
---

# nexu-io/open-design 项目调研

调研日期：2026-08-22。本文以 [nexu-io/open-design](https://github.com/nexu-io/open-design) 的 `main` 分支、README、QUICKSTART、架构/协议文档、源码入口、最新 Release 和 Issues 为依据。仓库变化很快，文中的数量和版本均是本次核验时的快照，不应视为长期 API 承诺。

## 结论先行

1. **OpenDesign 不是传统的画布型 Figma 克隆，而是“Agent 运行时 + 设计工作流 + 文件预览/导出”的产品。** 它让 Claude Code、Codex、Cursor、OpenCode、DeepSeek Harness 等已有 CLI 生成真实 HTML/CSS/组件和媒体文件，再在本地项目工作区中预览、迭代和交付。
2. **核心抽象是四个可组合平面：Skills、Design Templates、Design Systems、Plugins。** `SKILL.md` 描述 Agent 行为，模板描述可渲染的起始产物，`DESIGN.md`/tokens 描述品牌约束，插件用 `open-design.json` 把流程、输入、管线和能力声明打包成可安装单元。
3. **最有价值的场景是 Agent 驱动的原型、Dashboard、网页 Deck、设计系统约束、媒体生成以及现有代码库的品牌刷新。** 产物可以继续交给 Codex、Cursor 或 Claude Code，也可以导出 HTML、PDF、PPTX、ZIP、Markdown 和 MP4。
4. **本地优先不等于完全离线或无遥测。** 项目文件和 BYOK 密钥默认留在本机，但产品分析/质量追踪默认可选开启；配置了遥测目标的构建仍会发送经过清洗的安全与可靠性诊断。模型、Agent CLI、连接器和 OpenDesign Cloud 也可能需要网络。
5. **目前适合试点，不宜把主分支直接当作稳定平台。** 本次核验时最新稳定 Release 是 `0.20.2`，仓库 package 版本已经是 `0.20.3`；`main` 相对 `0.20.2` 有 13 个 ahead、9 个 behind 的分叉提交，Issue 和 PR 仍以很高频率变化。

## 项目定位与状态

| 项目 | 本次核验结果 |
| --- | --- |
| 仓库 | [nexu-io/open-design](https://github.com/nexu-io/open-design) |
| 官方定位 | 开源的 Claude Design alternative、Agent-native Figma alternative |
| 代码许可 | 根仓库 Apache-2.0 |
| 技术主语言 | TypeScript；monorepo，使用 pnpm workspace |
| 支持形态 | macOS/Windows 原生桌面端、Web、Docker、可从源码运行；Linux AppImage 为可选发布通道 |
| 依赖基线 | Node `~24`、pnpm `10.33.x` |
| 稳定版本 | `open-design-v0.20.2`，2026-08-21 发布 |
| 主分支版本 | 根 `package.json` 为 `0.20.3`，早于稳定 Release 的版本提升提交已进入 `main` |
| GitHub 快照 | 约 90,297 Star、10,414 Fork、825 个 open issues；2026-08-22 仍有新提交 |

Star/Fork 数量只能说明项目传播度，不能替代兼容性、稳定性和安全审查。尤其是这个仓库从 2026-04-28 创建以来发展很快，当前代码、Release 安装包和文档可能不在同一版本线上。

## 能力总览

### 1. 多种设计与媒体产物

README 和模板目录覆盖的主要产物包括：

- Web、桌面和移动端原型；
- Live Artifact、KPI Dashboard、研究/决策房间；
- 网页式 Pitch Deck、杂志 Deck、周报、课程和报告；
- 文档、图片、音频以及基于 HyperFrames 的 HTML/CSS/GSAP 视频；
- 从已有代码、截图、URL、Figma 或 Pencil 工作流继续生成和迁移。

默认交付物偏向**可运行的单页文件**，而不是设计文件格式。Agent 写出真实 HTML、CSS、字体引用和组件后，OpenDesign 在沙箱 iframe 中预览；这使结果容易交给工程团队继续开发，但不等同于拥有 Figma 那种完整的矢量图层、组件实例和多人画布模型。

导出能力包括 HTML（可内联）、PDF（浏览器打印）、PPTX（Agent/渲染流程驱动）、ZIP、Markdown 和 MP4（HyperFrames）。0.20.2 的发布说明特别修复了字体或图片资源迟迟不返回时导出长时间卡住的问题，但不同运行方式的导出能力仍需逐项验证。

### 2. Skills：让 Agent 复用工作方法

`skills/` 中每个能力包至少包含一个 `SKILL.md`，可以附带 `assets/` 和 `references/`。格式沿用 Agent Skills 的目录和 YAML front matter，OpenDesign 通过 `od:` 扩展声明：

- `mode`：`prototype`、`deck`、`template`、`design-system`、`image`、`video` 或 `audio`；
- `surface`、`category`、`scenario` 和本地化名称/描述；
- 预览类型、示例提示词和是否需要完整 Design System 上下文；
- `craft` 引用，例如 typography、color、anti-ai-slop；
- Critique Theater 的 `required`、`opt-in` 或 `opt-out` 策略。

仓库宣称附带 100+ functional skills。兼容性设计允许没有 `od:` 的旧技能继续工作：缺省模式、预览类型和设计系统要求由注册器推断。用户技能目录优先于内置目录，同 ID 可以 shadow 内置条目，但显式 Workspace 的可见性还会经过绑定检查。

### 3. Design Templates：可渲染的起始结构

`design-templates/` 与功能型 Skills 分开。它面向创建流程提供网页原型、Deck、Dashboard、文档、图片、视频和音频的渲染蓝图；选择模板并不只是给 Agent 一段建议，而是决定产物的文件形状、预览方式和交付约束。

这一区分很重要：功能 Skill 是 Agent 工作时调用的能力，Design Template 是用户创建某一类产物时选择的起点。两者可以一起参与提示词组合，但注册、展示和 API surface 分离。

### 4. Design Systems：以 `DESIGN.md` 作为品牌合同

`design-systems/` 当前包含 151 个包。新包的最小结构是：

```text
design-systems/<slug>/
├── manifest.json
├── DESIGN.md
└── tokens.css
```

- `manifest.json` 保存 ID、名称、分类、描述、来源和文件声明；
- `DESIGN.md` 是给 Agent 读取的设计规则正文；
- `tokens.css` 是编译后的语义 Token 样式表；
- 可选文件包括 `USAGE.md`、组件 fixture、Design Tokens JSON、Tailwind 映射、字体、资产、预览和上游证据。

选择一个系统后，daemon 将设计上下文拼进本次 Agent 的提示词。这个机制适合把团队品牌、颜色、字体、间距、组件和反模板化规则放进版本库，而不必每次在自然语言中重复描述。

### 5. Plugins：把流程变成可安装产品单元

官方插件位于 `plugins/_official/`，社区插件位于 `plugins/community/`，注册表位于 `plugins/registry/`。一个插件通常包含：

```text
my-plugin/
├── open-design.json
├── SKILL.md          # skill/scenario 类型需要
├── README.md
├── preview/
└── examples/
```

`open-design.json` 负责稳定 ID、版本、标题、输入、工作流类型、输出模式、管线 stages、预览和能力声明。当前 v1 能力包括 `prompt:inject`、`fs:read`、`fs:write`、`mcp`、`subprocess`、`bash`、`network` 和 `connector` 等。受限安装默认只授予 `prompt:inject`，插件应声明最小能力集合。

README 的仓库快照统计为 277 个官方插件和 183 个可 remix 的参考示例，包含场景、图片模板、视频模板、设计系统、UI atoms 和示例产物。这个数字会随主分支变化，而且“官方插件”不意味着每个附带资源都使用根仓库的 Apache-2.0 许可。

CLI 和 Web UI 共用插件 API，可执行搜索、查看、安装、应用、升级、卸载、脚手架和校验：

```bash
od plugin list --json
od plugin search "landing page"
od plugin info od-default
od plugin install od-figma-migration
od plugin apply od-default --input brief="a one-page pitch for our seed round"
od plugin validate ./my-plugin
```

### 6. Agent 运行时与 MCP

仓库把 Agent 接入分成两种方式：

- **外部 Agent 消费 OpenDesign：** 通过 Skills、CLI 和 MCP，让 Codex、Claude Code、Cursor 等读取项目文件、选择技能和插件，或调用 OpenDesign 的生成能力；
- **OpenDesign 直接启动 Agent：** daemon 根据运行时定义探测本机 CLI、拼接提示词、启动子进程并标准化输出。

当前注册表有 27 个 runtime definitions，对应 26 个不同的本地 CLI 可执行文件；`byok-opencode` 与 OpenCode 共用可执行文件，因此不算新增本地 executable。源码按传输协议支持 `claude-stream-json`、`json-event-stream`、Copilot/Qoder stream、ACP JSON-RPC、Pi RPC、DeepSeek Harness profile JSONL 和 plain text 等类型。用户还可以扩展本地 runtime profile。

MCP 是一个 stdio server，`od mcp install <agent>` 会按目标 Agent 的配置格式生成或合并配置；对无法确认配置格式的客户端，安装器只打印 snippet，不冒险写入未知文件。MCP 工具覆盖：

- 列出项目、Skills、插件和 Agent；
- 读取项目元数据、文件列表、单文件内容和依赖文件 bundle；
- 创建项目、创建/写入/删除文件、启动/查询/取消运行；
- 从活动项目获取预览和产物，并读取 `od://skills/...`、`od://design-systems/...` 资源。

MCP server 自身不持有项目状态，真正的文件和运行请求通过本机 daemon 完成。读工具带只读标记，写工具带写入标记；删除项目要求显式项目和 `confirm:true`，这降低了 Agent 误操作的概率，但不能替代对外部 Agent、插件和运行时权限的信任审查。

## 工作流与架构

典型数据流可以概括为：

```text
brief
  ↓
Plugin / Skill / Design Template + DESIGN.md
  ↓
Next.js Web 或 Electron → Express daemon
  ↓
Runtime Registry → 本地 CLI / ACP / BYOK API
  ↓
项目工作区中的 HTML/CSS/JS/媒体文件
  ↓
沙箱 iframe 预览 → lint / critique / export / handoff
```

### 运行时组成

| 层 | 当前实现 | 职责 |
| --- | --- | --- |
| Web | Next.js 16 App Router、React 18、TypeScript | Home、聊天、文件工作区、预览、设置、插件市场和导出入口 |
| Daemon | Node 24、Express 5、SSE、SQLite、`better-sqlite3` | 项目/会话持久化、文件 API、运行时探测、提示词组合、Agent 生命周期、Skills/插件/媒体服务 |
| Desktop | Electron shell + sidecar IPC | 启动和管理 Web/daemon，提供原生桌面入口和 STATUS/EVAL/SCREENSHOT 等 IPC 能力 |
| Preview | URL 或 `srcDoc` sandbox iframe | 渲染项目文件；需要 Host bridge 的检查、评论、调色和 tweaks 功能走 `srcDoc` 路径 |
| CLI | `od`，入口为 daemon CLI | 启动 daemon、安装 MCP、管理项目/插件/技能、驱动媒体/导出等无 UI 操作 |
| 生命周期 | `pnpm tools-dev` | 统一管理 start、stop、run、status、logs、inspect、check |

### 两种生成交付模式

1. **Filesystem execution profile：** daemon 以项目工作区为 `cwd` 启动具备文件/工具能力的本地 Agent；Agent 写出 canonical project files，文件事件通过 SSE 回传，预览读取这些真实文件。
2. **Text-artifact execution profile：** plain-stream 运行时或 BYOK/API 模式没有文件工具；Agent 返回一个完整的 `<artifact>` 源代码块，Host 解析并物化为相同的文件工作区和预览入口。

因此，“BYOK 支持”不一定等于“模型可以直接编辑本地项目”。是否能写真实文件，取决于所选运行时的 transport 和执行 profile。

## 安装与使用方式

### 桌面端

官方推荐直接下载 macOS 或 Windows 安装包；Windows 提供 x64 安装包，macOS 提供 Apple Silicon 和 Intel x64，Linux 为 AppImage 可选通道。安装后会扫描 PATH 中可用的 Coding Agent CLI，再加载 Skills、模板和 Design Systems。

### Agent/MCP 模式

不打开 GUI 也可以使用：

```bash
od mcp install codex
od project list --json
od files list <project-id> --json
od plugin list --json
od skills list --json
```

macOS 或 WSL2 需要注意系统自带的 `/usr/bin/od` 是 octal dump 工具，可能遮蔽 OpenDesign 的 `od`。官方建议桌面用户从 Settings → MCP server 复制绝对路径配置，WSL2 用户先按仓库的 WSL 指南安装。

### Docker / 自托管

仓库提供 Docker Compose，默认在 `127.0.0.1:7456` 提供服务；公共或共享部署需要设置 API token、允许来源和反向代理的 SSE 配置。Sealos 也有应用模板。Docker 适合不想安装 Node/pnpm 的用户，但不能据此推导所有桌面端能力、媒体能力和 PPTX 导出都能在容器中使用。

### 从源码运行

```bash
corepack enable
pnpm install
pnpm tools-dev run web
```

仓库明确要求 Node `~24` 和 pnpm `10.33.x`。Windows 的 `better-sqlite3` 可能需要本机编译工具；本地生命周期不应使用旧的 `pnpm dev`、`pnpm start` 等别名，应通过 `pnpm tools-dev` 管理 daemon、Web 和 desktop。

## 安全、隐私与许可

### 安全边界

- daemon 默认绑定 `127.0.0.1`；暴露到 LAN/公网需要显式 bind、origin 和认证配置；
- BYOK provider URL 默认拦截 RFC1918、link-local、CGNAT 和云元数据地址，内部模型网关需要显式 `OD_ALLOWED_INTERNAL_HOSTS`；
- 预览和插件页面使用 sandbox iframe，文件操作会限制在当前项目根目录；
- MCP 连接本地 daemon，连接器凭据和 daemon 数据由 daemon 管理，不应复制到项目文件；
- 运行时的子进程权限取决于每个 Agent adapter，不能假定所有 CLI 都使用相同的 sandbox 或 permission flags。

这些边界适合本机开发和受控自托管，但不能把“默认 loopback”当作生产公网安全方案。公网部署必须同时审查 token、CORS/origin、反向代理 SSE、日志脱敏、Agent 子进程和文件导入路径。

### 隐私与遥测

仓库隐私说明将数据分为两类：

- 可选的产品分析和质量追踪：默认开启，但首次运行提示用户选择，Settings → Privacy 可关闭或缩小范围；可能包含运行次数、Token 使用量、错误率、时长、页面事件和经过遮罩的 session replay；若同时开启内容控制，质量追踪还可能发送截断/清洗后的 prompt、响应、工具输入输出和 artifact manifest 元数据；
- 安全与可靠性遥测：在构建配置了遥测目标时始终开启，发送经过限制和清洗的崩溃、启动、SSE、资源和长任务诊断。

文档声明不会主动收集生成文件全文、BYOK key、认证 token 或可读的遮罩后 prompt/artifact 文本。与此同时，遥测接收方仍能看到 HTTPS 连接的源 IP，因此“匿名”不应理解为网络层完全不可识别。要求严格离线或禁止任何诊断上报的团队需要使用无遥测目标的开发/派生构建，并自行复核代码和发布配置。

### 许可与第三方内容

根仓库代码声明 Apache-2.0，但插件 manifest、模板、设计系统和示例资产可以保留各自的许可证和来源。例如仓库中的官方 `od-default` 插件 manifest 声明为 MIT，README 也特别说明某些引入的 Deck skill 保留原始许可证。部署前应分别检查：

1. 根代码和打包产物的 Apache-2.0 条款；
2. 每个插件的 `open-design.json`、README、LICENSE 和 provenance；
3. 图片、字体、视频、模板和上游设计系统的再分发限制；
4. 使用 OpenDesign Cloud、模型 API 或第三方 Agent CLI 时各自的服务条款和数据政策。

## 与相邻工具的区别

| 维度 | OpenDesign | 传统 Figma/画布工具 | 只安装一个 Agent Skill |
| --- | --- | --- | --- |
| 核心对象 | Agent 可读写的项目文件和可组合工作流 | 图层、矢量节点、组件和画布 | 一段工作方法/提示词 |
| Agent 接入 | 运行时 registry、MCP、CLI、Skills、插件 | 通常依赖插件、API 或外部桥接 | 取决于宿主 Agent |
| 品牌约束 | `DESIGN.md` + tokens + fixtures，可进 Git | 设计库、组件和变量 | 通常由 skill 自行约定 |
| 交付 | HTML/CSS/组件、媒体、PDF/PPTX/MP4 | 设计文件、原型和导出资源 | 由 Agent 输出决定 |
| 本地控制 | 桌面端、daemon、Docker、自托管、BYOK | 取决于工具 | 通常较高，但能力较窄 |
| 主要短板 | 依赖 Agent/CLI，范围大，运行时适配复杂 | 代码交付和 Agent 自动化可能较弱 | 缺少项目、预览、持久化和统一运行时 |

OpenDesign 自称 Claude Design 和 Figma 的开放替代方案，但更准确的理解是：它开放了“brief → Agent → 文件 → 预览/导出”的闭环，而不是复刻一个成熟的像素/矢量画布。需要保留 Figma 图层模型、精确矢量编辑、组件实例和成熟多人协作的团队，仍应把 OpenDesign 作为代码生成和 Agent 自动化层来评估。

## 风险与待核查项

### 1. 版本线分叉，必须固定版本

最新 Release `0.20.2` 的稳定安装包与 `main` 的 package `0.20.3` 不同。本次 compare 显示 `main` 相对 `0.20.2` 有 13 个 ahead、9 个 behind，说明不能只看 README 或主分支代码后就假定下载包有同样能力。生产试点应固定 Release、commit、Docker image tag 或内部构建号。

### 2. 适配器数量带来兼容性矩阵

27 个运行时定义覆盖不同 stream、RPC 和 plain 输出协议，意味着认证、参数、权限、模型发现、取消、恢复和错误分类都可能是 Agent-specific。当前 Issues 中能看到 Windows PowerShell stdout、DeepSeek Harness 版本策略、MCP stdio 进程静默退出、Linux AppImage 冷启动竞争等问题，不能仅用“支持 26 个 CLI”判断每个适配器都达到相同成熟度。

### 3. 自托管能力不是桌面端能力的简单平移

当前 Issues 包含 Docker/self-hosted 部署中 PPTX 入口可能只能返回 501 的报告，以及 self-hosted Linux daemon CPU-bound 的报告。要自托管，必须单独测试媒体、PPTX、字体、浏览器渲染、反向代理 SSE、长任务和资源持久化；不能只验证首页能打开。

### 4. 插件和 Agent 都是可执行信任边界

插件能力声明提供了权限提示，但真正执行仍依赖 Agent CLI、子进程、MCP、网络和连接器。安装第三方插件前应阅读 `SKILL.md`、manifest、脚本、外部 URL、默认 capability 和数据流；对于团队注册表，应把校验、版本锁定、来源记录和升级审批作为发布流程的一部分。

### 5. 产物质量仍取决于上下文和审查

设计系统可以稳定颜色、字体、组件和布局约束，但不能自动保证品牌策略、无障碍、响应式行为、浏览器兼容性、PPTX 可编辑性或商标合规。应把截图/浏览器检查、artifact lint、Critique Theater、人工设计审查和前端测试放在生成之后。

## 适用场景建议

| 场景 | 建议 |
| --- | --- |
| Agent 生成 Landing Page、Dashboard、移动端原型 | 推荐试用；这是仓库最匹配的主场景 |
| 希望团队把品牌规范放进 Git，并让 Agent 自动遵循 | 推荐；采用 `DESIGN.md` + `tokens.css` + 组件 fixture |
| 在 Codex/Claude Code/Cursor 中读取和修改 OpenDesign 项目 | 推荐；优先测试 `od mcp install codex` 和 MCP 文件闭环 |
| 从已有代码库进行品牌刷新或 Figma/Pencil 到代码迁移 | 值得试点；先审查对应官方插件的输入和输出契约 |
| 需要快速做网页 Deck、PDF 或视频草稿 | 推荐，但按媒体和字体依赖做导出回归 |
| 需要 Figma 级矢量绘图、完整图层/组件模型和成熟多人协作 | 不应作为唯一工具 |
| 严格离线、禁止诊断上报、对密钥和内容有极高隔离要求 | 需要源码审计、无遥测构建和网络隔离后再采用 |
| 生产自托管且必须提供可编辑 PPTX、全媒体和多平台运行时 | 先做专项 POC，不要直接按 README 能力清单上线 |

## 建议的试点验证清单

1. 固定 `open-design-v0.20.2` 或某个 commit，分别记录桌面端、Docker image 和源码构建的版本。
2. 用 Codex 完成“创建项目 → 选择 Skill/Design System → 写入 HTML/CSS → MCP 读取 → 预览 → 修改 → 导出”的闭环，并保留文件 diff。
3. 分别测试 filesystem runtime 和 BYOK/plain-artifact runtime，确认两者的文件写入、错误提示、恢复和会话续接行为。
4. 用真实品牌包验证 `DESIGN.md`、`tokens.css`、字体、图片、组件 fixture 和响应式断点是否都进入生成结果。
5. 对 HTML、PDF、PPTX、MP4 和 ZIP 做相同输入的导出矩阵；重点测缺失字体、远程图片、长 SSE、浏览器关闭、daemon 重启和容器重启。
6. 在自托管环境配置 token、allowed origins、反代 SSE 超时和日志脱敏；若使用内网模型网关，显式配置并审查 `OD_ALLOWED_INTERNAL_HOSTS`。
7. 逐个审查第三方插件许可证和 capability，建立 registry allowlist、版本锁定、升级前校验和回滚方案。

## 资料来源

- [GitHub 仓库](https://github.com/nexu-io/open-design)
- [README：定位、产物、Agent、插件、架构和快速开始](https://github.com/nexu-io/open-design/blob/main/README.md)
- [QUICKSTART.md：Docker、源码运行、两种执行模式和故障排查](https://github.com/nexu-io/open-design/blob/main/QUICKSTART.md)
- [架构文档](https://github.com/nexu-io/open-design/blob/main/docs/architecture.md)
- [Agent Adapters](https://github.com/nexu-io/open-design/blob/main/docs/agent-adapters.md)
- [Skills Protocol](https://github.com/nexu-io/open-design/blob/main/docs/skills-protocol.md)
- [Design Systems README](https://github.com/nexu-io/open-design/blob/main/design-systems/README.md)
- [Plugin Spec](https://github.com/nexu-io/open-design/blob/main/plugins/spec/SPEC.md)
- [隐私说明](https://github.com/nexu-io/open-design/blob/main/PRIVACY.md)
- [最新稳定 Release 0.20.2](https://github.com/nexu-io/open-design/releases/tag/open-design-v0.20.2)
- [Issues：当前实现问题与需求](https://github.com/nexu-io/open-design/issues)
- [Issue #7273：MCP stdio 进程静默退出](https://github.com/nexu-io/open-design/issues/7273)
- [Issue #7249：Windows PowerShell 媒体 CLI stdout](https://github.com/nexu-io/open-design/issues/7249)
- [Issue #7213：Docker/self-hosted PPTX 导出入口](https://github.com/nexu-io/open-design/issues/7213)
- [Issue #7195：self-hosted Linux daemon CPU-bound](https://github.com/nexu-io/open-design/issues/7195)
