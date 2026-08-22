---
title: "OpenDesign、OpenPencil 与 pen.dev 对比报告"
date: 2026-08-22
category: "Agent工程"
tags:
  - OpenDesign
  - OpenPencil
  - pen.dev
  - 设计即代码
  - MCP
  - CLI
  - Figma
  - .pen
---

# OpenDesign、OpenPencil 与 pen.dev 对比报告

> 对比日期：2026-08-22。本文仅基于仓库中三份已有调研报告整理，版本、功能数量和官方页面状态均应视为对应调研日期的快照。

## 结论先行

这三个工具不是同一层级的直接替代品：

- **OpenDesign** 的核心对象是 Agent 可读写的项目工作区，重点是从 brief 生成 HTML/CSS/组件、媒体和可交付文件，再通过预览、审查和导出完成交付。它更接近“Agent 设计生产运行时”。
- **OpenPencil** 的核心对象是 SceneGraph 和设计文件，重点是编辑、解析和自动化操作 Figma `.fig`，同时提供 CLI、MCP、Figma API 风格脚本和 Vue SDK。它更接近“开源设计编辑器内核 + Agent 工具包”。
- **pen.dev** 的核心对象是 `.pen` 设计文件，VS Code 插件、桌面端、MCP 和 CLI 是不同宿主或接入层。它更接近“设计即代码文件格式的编辑器与 Agent 接口”。

按主要需求选择：

| 主要需求 | 首选 | 理由 |
|---|---|---|
| Agent 直接产出网页、Dashboard、Deck、PDF、PPTX、视频或媒体文件 | OpenDesign | 工作流围绕项目文件、预览、插件、设计系统和导出展开 |
| 打开、修改、检查和转换 Figma `.fig`，或构建定制设计编辑器 | OpenPencil | 有 Kiwi codec、SceneGraph、CLI/MCP、DOM/CSS 导入和 Vue SDK |
| 让设计文件与代码同仓库，在 IDE 中协作或使用 `.pen` 作为原生工作文件 | pen.dev | VS Code 插件和 `.pen` 文件工作流最直接 |
| 让多个 AI 客户端读取和修改设计 | 视文件格式选择 MCP | OpenDesign MCP 面向项目文件，OpenPencil MCP 面向 SceneGraph，pen.dev MCP 面向 `.pen` |
| 在 CI 中做设计分析、lint、批处理和导出 | OpenPencil 或 pen.dev CLI | OpenPencil 的文件检查/分析更明确；pen.dev CLI 更适合 `.pen` 任务批处理 |
| 需要开源、可嵌入、可自行扩展的设计编辑器内核 | OpenPencil | 暴露 Core、SceneGraph、Vue SDK 和文件格式包 |

因此，最稳妥的判断不是“哪个工具最好”，而是先确定团队的**主交付物**：如果交付物是代码和媒体，优先看 OpenDesign；如果交付物是可编辑的设计文件，优先看 OpenPencil 或 pen.dev；如果已有资产主要是 Figma `.fig`，优先验证 OpenPencil；如果已有资产主要是 `.pen`，优先验证 pen.dev。

## 一、定位与产品边界

| 维度 | OpenDesign | OpenPencil | pen.dev |
|---|---|---|---|
| 核心定位 | Agent 运行时、设计工作流和文件预览/导出平台 | AI 原生设计编辑器、可编程设计引擎和 Agent 工具包 | `.pen` 编辑器及其 VS Code、桌面端、MCP、CLI 接入方式 |
| 主要工作对象 | HTML/CSS/JS、组件、图片、音频、视频等项目文件 | SceneGraph、Figma `.fig`、可导入的 HTML/CSS/Tailwind | 基于 JSON、适合与代码同仓库管理的 `.pen` 文件 |
| 画布角色 | 沙箱 iframe 中的项目预览，不是完整图层画布 | 完整画布、图层、属性、组件、变量和布局编辑器 | VS Code 内画布或独立桌面画布 |
| Agent 角色 | 负责生成和修改项目产物，OpenDesign 负责运行、预览和交付 | 通过 MCP、CLI、内置 AI 和脚本直接读写设计节点 | 通过 MCP 或 CLI 读写 `.pen`，客户端负责提供编辑宿主 |
| 主要交付 | 可运行网页、代码、媒体、PDF/PPTX/ZIP/MP4 | `.fig`、图片、SVG、JSX、HTML、PPTX 等 | `.pen` 设计文件，以及 PNG、JPEG、WEBP、PDF、HTML 等导出 |
| 开源与部署信息 | 根仓库 Apache-2.0；支持桌面端、Web、Docker 和自托管 | MIT；支持 Web/PWA、Tauri 桌面端和本地文件模式 | 本组调研未覆盖许可证和自托管能力；重点描述本地客户端和 CLI |

### 不是同一种“设计工具”

OpenDesign 的最小闭环是“brief → Agent → 项目文件 → 预览/审查/导出”；OpenPencil 的最小闭环是“设计文件 → SceneGraph → 画布或脚本修改 → 保存/导出”；pen.dev 的最小闭环是“`.pen` 文件 → 客户端画布或 MCP/CLI → 保存/代码协作”。

如果把三者都按“能不能像 Figma 一样拖拽图层”评价，会低估 OpenDesign 的代码交付能力，也会忽略 OpenPencil 的文件解析和 SDK 能力；如果只按“能不能让 Agent 生成页面”评价，又会忽略三者对设计文件持久化和编辑模型的差异。

## 二、文件格式与设计到代码

| 能力 | OpenDesign | OpenPencil | pen.dev |
|---|---|---|---|
| 原生主格式 | 没有以 `.fig`/`.pen` 为中心的原生画布格式，主要保存项目文件 | `.fig` 是主要兼容目标；`.pen` 当前以读取和转换为主 | `.pen` 是主要设计文件格式，可与源代码一起进行 Git 管理 |
| Figma 兼容 | 可通过插件或迁移工作流接入 Figma/Pencil，但不是 Figma 图层模型的原生实现 | 直接解析、编辑和写回 `.fig`，支持 Figma clipboard 和部分 Plugin API 兼容 | 现有调研未把 Figma 兼容列为核心能力 |
| `.pen` 能力 | 可作为迁移输入或工作流来源，但不是核心存储模型 | 可解析并映射到 SceneGraph；当前未完成 `.pen` 写入和导出 | 以 `.pen` 作为核心编辑和协作文件 |
| 设计到代码 | Agent 直接生成真实 HTML/CSS/组件和媒体文件 | 导出 JSX、Tailwind JSX、HTML、SVG；也可导出 `.fig` 页面或节点 | 通过 AI 和 MCP 支持从设计到 React、Next.js、Vue、Svelte、HTML/CSS 等代码 |
| 代码到设计 | 可以从已有代码库、截图、URL、Figma/Pencil 工作流继续生成或刷新 | 可将 HTML、CSS、Tailwind 导入为可编辑设计节点 | 重点是设计与代码同处 IDE 工作区；原调研未提供与 OpenPencil 同等的 DOM/CSS 导入矩阵 |
| 往返编辑重点 | 代码文件和预览是否保持可运行、可交付 | `.fig` 读写和 SceneGraph 与 Figma 的结构/渲染差异 | `.pen` 文件与代码仓库、客户端和 Agent 操作的一致性 |

### 关键判断

1. **要保留 Figma 资产的编辑结构，OpenPencil 最有针对性。** 但其兼容性不是完整 Figma 替代：已有报告记录 158 项功能中 76 项完整支持、18 项部分支持、64 项尚未实现，且保留字段不等于字段已经可以渲染或编辑。
2. **要把网页代码直接变成可编辑设计，OpenPencil 的方向最清晰。** `@open-pencil/dom-css` 和 CLI 支持 HTML/CSS/Tailwind 导入；这与 OpenDesign 直接把代码作为最终产物的路径不同。
3. **要围绕 `.pen` 建立设计与代码协作，pen.dev 的路径最短。** 但不要把 OpenPencil 当前的 `.pen` 解析能力误认为完整的 `.pen` 原生写回能力。

## 三、Agent、MCP 与 CLI 对比

| 维度 | OpenDesign | OpenPencil | pen.dev |
|---|---|---|---|
| MCP 控制对象 | 项目、文件、Skills、插件、运行任务、预览和产物 | 文档、页面、节点、组件、变量、导出和正在运行的编辑器 | `.pen` 文档状态、节点操作、截图、导出和设计指南 |
| MCP 形态 | stdio；通过本机 daemon 执行文件和运行请求 | stdio 和 Streamable HTTP；支持文件模式和运行中应用模式 | pen.dev 启动后本地自动运行，供 Claude、Cursor、Codex 等客户端连接 |
| AI 内置能力 | 依赖外部 Agent CLI、BYOK 或运行时适配器 | 内置 AI Chat，可接多家 Provider 和兼容端点 | 桌面端内置 AI Chat；VS Code 主要使用 IDE/Claude Code 的 AI 面板 |
| CLI 重点 | 管理项目、插件、技能、MCP、媒体和导出，并启动 Agent 运行 | `tree`、`find`、`query`、`lint`、`analyze`、`import`、`export`、`eval` | Agent、interactive、batch/CI，支持 `--workspace`、`--prompt` 和 `PEN_CLI_KEY` |
| 结构化自动化 | 文件 API、插件管线、运行生命周期和预览产物 | JSON 输出、设计分析、Figma API 风格 `eval` 和节点级操作 | 交互式 MCP 工具调用、批量任务、截图和导出 |
| 典型 Agent 闭环 | 选择 Skill/Design System → 生成文件 → 预览 → critique/lint → 导出 | 读取页面树 → 修改节点 → 渲染/分析 → 保存或导出 | 读取 `.pen` → 执行设计操作 → 截图验证 → 保存/导出 |

### Agent 接入的本质差异

- OpenDesign 把 Agent 当作**产物生产者**，重点解决不同 Coding Agent CLI 的启动、协议适配、文件写入、会话和预览问题。它有 27 个 runtime definitions，对应多种 stream、RPC 和 plain text 协议，因此适配面广，但兼容性矩阵也更复杂。
- OpenPencil 把 Agent 当作**设计文档操作者**，工具粒度深入到节点、布局、变量、组件和导出；CLI 的 `eval` 还提供比单次 MCP 调用更低层的批量脚本能力。
- pen.dev 把 MCP 作为**设计文件的统一控制接口**，客户端、CLI 与外部 AI 共享 `.pen` 工作流。它最适合将设计作为代码仓库中的一个可被 Agent 操作的文件，而不是把 Agent 运行时本身做成平台。

## 四、扩展、协作与部署

| 维度 | OpenDesign | OpenPencil | pen.dev |
|---|---|---|---|
| 扩展方式 | Skills、Design Templates、Design Systems、Plugins、runtime profiles | Core、SceneGraph、文件格式包、MCP、CLI、Vue SDK、Agent Skill | VS Code 插件、桌面端、MCP、CLI；以官方客户端能力为主 |
| 品牌规范 | `DESIGN.md`、`tokens.css`、组件 fixture 和设计系统进入项目/版本库 | 变量、样式、组件和设计分析；完整变量/样式系统仍有缺口 | 通过 `.pen` 文件、设计指南、组件和 AI 工作流协作 |
| 嵌入定制 | 可通过插件和项目工作流扩展，但产品范围较大 | Core + Vue SDK 适合定制编辑器、内部工具和嵌入画布 | 原调研重点不在嵌入式 SDK，不能按 OpenPencil 的 SDK 深度评价 |
| 实时协作 | 本地优先；可通过桌面端、daemon、Docker、自托管部署 | WebRTC/Trystero/Yjs P2P 协作，支持光标、在线状态和跟随视口 | 原调研确认的是本地文件、客户端和 MCP 协作，未形成同等的协作服务比较 |
| 企业治理 | 公网部署、SSO、插件权限、SSE、遥测和自托管能力需要专项审查 | Cloud、组织权限、OIDC/SSO、版本历史、审计和持久中继仍在路线图 | Windows 版本和官方文档存在状态不一致，团队部署需要实机验证 |

### 开源与本地控制不能简单等同

OpenDesign 和 OpenPencil 都提供源码和本地运行路径，但两者的“本地”含义不同：OpenDesign 的本地文件和 BYOK 不代表完全离线，配置了遥测目标的构建仍可能发送诊断；OpenPencil 的 MCP 有本地 root、认证、CORS 和 `eval` 边界，但其企业云治理仍不成熟。pen.dev 的调研结论是 MCP 本身在本地操作设计文件，使用 AI 功能时提示词仍会发送到相应 AI 服务；这不能推导为所有客户端或模型调用都完全离线。

## 五、场景化选型

| 场景 | 推荐方案 | 选择理由与边界 |
|---|---|---|
| Agent 快速生成 Landing Page、Dashboard、移动端原型 | OpenDesign | 对“生成真实网页文件并预览交付”的流程支持最完整；仍需浏览器截图、响应式和前端测试 |
| 生成网页式 Deck、报告、PDF/PPTX 或视频草稿 | OpenDesign | 模板、媒体、插件和导出路径更贴近多媒体交付；Docker/自托管的 PPTX、字体和媒体能力要单独验证 |
| 现有 Figma `.fig` 的批量检查、颜色/字体/间距分析 | OpenPencil | 文件模式 CLI、JSON、lint 和 analyze 适合接入 CI；仍要做真实文件 round-trip 验证 |
| 需要一个可嵌入业务系统的设计画布 | OpenPencil | 有 Core、SceneGraph 和 Vue SDK；公共 API 稳定性和升级成本由采用方承担 |
| 设计师和前端在 VS Code 中共同维护设计与代码 | pen.dev VS Code 插件 + MCP | 设计文件和代码处在同一工作区，AI 可同时获得代码与设计上下文 |
| 设计师以独立画布为主，并需要内置 AI Chat | pen.dev 桌面端 | 画布工作流独立于 IDE；Windows x64 安装状态需要按官方下载页实测，ARM64 尚未提供 |
| 用 `.pen` 做无界面生成、批处理或 CI/CD | pen.dev CLI + `PEN_CLI_KEY` | 支持 headless、interactive 和 batch；认证、模型、导出和错误恢复需纳入流水线验证 |
| 将 `.pen` 文件交给 OpenPencil 做完整原生编辑和保存 | 不建议直接假设可行 | 当前 OpenPencil 的 `.pen` 主要是读取和 SceneGraph 转换，写回和导出仍在路线图 |
| 需要完整 Figma 原型、评论、版本历史和企业协作治理 | 三者都不应直接作为唯一方案 | OpenPencil 的相关功能仍有缺口，OpenDesign 偏代码产物，pen.dev 报告未覆盖完整企业治理 |

## 六、组合使用建议

三个工具可以形成互补链路，但不应假定它们之间已经存在稳定的直接互操作协议。

### 方案 A：Figma 资产到前端代码

```text
Figma .fig
   ↓ OpenPencil：读取、检查、转换、结构化修改
OpenPencil SceneGraph / JSX / HTML / Tailwind
   ↓ OpenDesign：接入 Design System、Agent 工作流和项目预览
可运行网页、Deck、PDF/PPTX 或媒体产物
```

适合需要保留 Figma 资产结构，同时希望最终交付真实代码或多媒体文件的团队。关键验证点是 `.fig` round-trip、字体/图片资源、导出差异，以及 OpenDesign 对输入产物的实际接管方式。

### 方案 B：`.pen` 设计与代码协作

```text
VS Code / pen.dev 桌面端
   ↕
.pen 文件 + Git
   ↕ MCP / CLI
Codex / Claude / Cursor / CI
```

适合把设计文件当作代码仓库的一部分维护。pen.dev 负责 `.pen` 编辑宿主和工具接口；若要进一步交给 OpenDesign 生成网页或媒体，应先验证对应迁移插件、导出形式和样式语义是否稳定。

### 方案 C：OpenPencil 作为设计自动化引擎

OpenPencil 更适合承担设计文件检查、节点级批量修改、HTML/CSS 导入、设计分析和截图验证；OpenDesign 更适合承担项目级 Agent 运行、品牌规范注入、网页预览和媒体导出。两者组合时应把文件边界、版本号、输入输出格式和失败回滚写入试点脚本，避免只依赖自然语言描述。

## 七、共同风险与落地判断

1. **版本和文档可能不同步。** OpenDesign 的稳定 Release 与 `main` 已存在版本分叉；OpenPencil 的 `master` 也比 `v0.14.0` Release 更新；pen.dev 的 Windows 下载页和旧 Troubleshooting 页面存在可用性冲突。评估时必须锁定 Release、commit、安装包或 CLI 版本。
2. **设计导入不等于设计可编辑。** OpenPencil 能保留部分未完全建模的 Figma 字段，但不代表都能渲染和编辑；`.pen` 在 OpenPencil 中当前更是明确的读取路径。跨工具迁移必须检查结构、视觉和再次保存结果。
3. **Agent 生成不等于交付质量。** 三者都需要截图/渲染验证、字体和图片检查、响应式检查、无障碍检查、代码测试或设计审查。OpenDesign 的 Design System、OpenPencil 的 analyze、pen.dev 的截图工具都不能替代最终验收。
4. **本地运行不等于低风险。** OpenDesign 涉及 Agent 子进程、插件能力、网络和遥测；OpenPencil 的 MCP `eval`、HTTP、CORS 和文件根目录需要配置；pen.dev 使用 AI 时提示词仍会进入相应 AI 服务。团队应分别审查密钥、文件范围、网络出口、日志和第三方插件。
5. **协作能力成熟度不同。** OpenPencil 的 P2P 协作适合轻量本地场景，但企业身份、版本、审计和数据治理仍需补齐；OpenDesign 的自托管能力要按媒体、PPTX、SSE 和 daemon 重启做专项 POC；pen.dev 的本组材料未覆盖完整企业协作后端。

## 八、建议的统一 POC

采用任何一个工具前，建议用同一组真实材料做对照：一个包含组件、变量、Auto Layout、图片和字体的 `.fig`，一个实际 `.pen` 文件，以及一个已有前端页面。

1. **格式验证：** 分别打开、修改、保存、再次打开并检查结构和视觉结果；明确哪些字段丢失、只读或无法导出。
2. **Agent 验证：** 让 Agent 完成“读取 → 修改 → 截图 → 导出”的闭环，记录工具调用、审批、错误信息和恢复方式。
3. **代码验证：** 比较 OpenDesign 生成的项目文件、OpenPencil 的 HTML/JSX/Tailwind 导出和 pen.dev 的设计到代码结果，检查可运行性、响应式和组件复用。
4. **流水线验证：** 测试 CLI 的 JSON 输出、lint、analyze、batch、非交互认证、资源缺失和进程重启。
5. **安全验证：** 检查 MCP 监听地址、认证 token、文件根目录、`eval`、插件 capability、遥测开关、模型 API 出口和日志脱敏。
6. **版本验证：** 固定每个工具的 Release/commit/安装包，保留输入文件、命令、截图、导出物和差异报告，避免用动态主分支结论直接做生产决策。

## 最终建议

如果只能先试一个工具，应按交付目标选择：**网页和媒体交付先试 OpenDesign，Figma `.fig` 自动化和可嵌入设计引擎先试 OpenPencil，`.pen` 文件与代码协作先试 pen.dev。**

对于同时存在 Figma、`.pen` 和前端代码的团队，建议先用 OpenPencil 或 pen.dev 负责设计文件层，再用 OpenDesign 负责 Agent 驱动的网页/媒体交付；这是一种需要通过 POC 验证的组合架构，不应把三者当成已经完全兼容的统一平台。

## 资料依据

- [OpenDesign 项目调研](./2026-08-22-OpenDesign项目.md)
- [OpenPencil 开源设计编辑器与 Agent 工具链调研](./2026-08-22-OpenPencil.md)
- [pen.dev VS Code 插件、Windows 桌面端、MCP 与 CLI 区别调研](./2026-08-20-pen.dev客户端与集成方式对比.md)

