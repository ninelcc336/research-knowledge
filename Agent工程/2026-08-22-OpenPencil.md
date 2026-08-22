---
title: "OpenPencil 开源设计编辑器与 Agent 工具链调研"
date: 2026-08-22
category: "Agent工程"
tags:
  - OpenPencil
  - Figma
  - .fig
  - .pen
  - MCP
  - CLI
  - Vue SDK
  - WebRTC
---

# OpenPencil 开源设计编辑器与 Agent 工具链调研

> 调研日期：2026-08-22。本文以 `open-pencil/open-pencil` GitHub 仓库、仓库文档、路线图和 `v0.14.0` Release 为依据。GitHub 仓库的 `master` 分支在 2026-08-22 仍有新提交，而 `v0.14.0` 于 2026-08-11 发布，因此本文会区分当前主分支和正式 Release。

## 结论

OpenPencil 是一个 MIT 许可的开源、AI 原生设计编辑器，同时也是可编程设计引擎和 Agent 工具包。它的核心价值不只是提供一个 Figma 风格画布，而是把 SceneGraph、文件格式解析、编辑器内核、CLI、MCP、DOM/CSS 导入和 Vue SDK 放在同一个公开代码库中。

关键结论如下：

- **主要兼容目标是 Figma `.fig`**：OpenPencil 使用 Kiwi 二进制 codec 读取、编辑和写回 `.fig`，同时支持 Figma clipboard 和部分 Figma Plugin API 兼容能力。
- **`.pen` 当前以读取为主**：`@open-pencil/pen` 可以解析 Pencil `.pen` 并转换到共享 SceneGraph，但格式参考表没有提供 `.pen` 写入或导出能力；路线图把这项工作列为后续目标。
- **Agent 自动化较完整**：提供内置 AI Chat、独立 MCP Server、无头 CLI、Figma API 风格 `eval`、JSON 输出、设计分析和 HTML/CSS/Tailwind 导入导出。
- **可扩展边界较深**：`@open-pencil/core`、`@open-pencil/scene-graph` 和 `@open-pencil/vue` 可用于构建定制编辑器、嵌入式画布、设计检查工具和业务专用工作台。
- **当前仍是快速演进中的项目**：Figma 原型、评论、版本历史、Code Connect、完整变量/样式系统和部分渲染兼容性仍有缺口；正式 Release 与主分支状态也可能不同。

## 项目概况

OpenPencil 的仓库定位是开源、AI 原生的 Figma 替代品，同时也是一个可编程设计工具包。GitHub API 在本次调研时显示仓库为公开、MIT 许可、非归档状态；仓库当时约有 7,926 个 Star 和 767 个 Fork。最新正式 Release 为 `v0.14.0`，提供 macOS、Windows x64/ARM64 和 Linux 构建；仓库 README 同时提供 Web/PWA 入口。

项目包含三种形态：

1. **设计编辑器**：浏览器/PWA 和 Tauri v2 桌面应用，提供画布、图层、属性、组件、变量、布局和导出能力。
2. **可编程工具包**：公开 SceneGraph、编辑器内核、文件格式包、DOM/CSS 转换包和 Vue SDK。
3. **Agent 自动化平台**：通过内置 AI、MCP、CLI、Figma Plugin API 兼容脚本和 Agent Skill 操作设计文件或运行中的编辑器。

## 技术架构

| 层 | 主要包或实现 | 作用 |
|---|---|---|
| 场景图 | `@open-pencil/scene-graph` | 节点、几何、命中测试、复制、吸附和撤销等共享设计模型 |
| Figma 文件 | `@open-pencil/fig`、`@open-pencil/kiwi` | 解析 Kiwi schema、读取/写入 `.fig`、处理压缩和资源 |
| Pencil 文件 | `@open-pencil/pen` | 解析 `.pen` 并转换为共享 SceneGraph |
| 编辑器内核 | `@open-pencil/core` | 编辑器状态、渲染、布局、工具、文档 I/O 和 RPC |
| 网页导入 | `@open-pencil/dom-css` | 将 HTML、CSS、Tailwind 转换为可编辑设计文档 |
| Vue 集成 | `@open-pencil/vue` | 提供 headless composables、结构化 primitives 和画布接入 |
| CLI | `@open-pencil/cli` | 文件检查、查询、导出、分析、转换和脚本执行 |
| MCP | `@open-pencil/mcp` | 为 AI 客户端提供 stdio/HTTP 工具服务 |
| UI | Vue 3、Reka UI、Tailwind CSS 4 | 编辑器界面、面板和控件 |
| 渲染 | CanvasKit WASM/Skia | 画布绘制、文本、渐变、图片和效果 |
| 布局 | Yoga WASM | Flex 和 Grid 布局 |
| 桌面 | Tauri v2 | macOS、Windows、Linux 原生应用 |
| 协作 | Trystero、WebRTC、Yjs CRDT | P2P 实时协作、光标、在线状态和跟随视口 |

官方 Vue SDK 文档明确把默认 OpenPencil 应用描述为 SDK 的一种组合，而不是唯一的使用方式。SDK 的目标是让开发者构建不同的编辑器壳层、内部工具和嵌入式设计工作流。

## 文件格式与渲染

### `.fig` 文件

OpenPencil 的 `.fig` 管道直接面向 Figma Kiwi 二进制格式：

```text
.fig 文件
  -> 解析 ZIP 容器和 fig-kiwi 头部
  -> 解压 Kiwi schema 与 message
  -> 解码 NodeChange[]
  -> 构建 SceneGraph
  -> 解析资源并渲染
```

仓库文件格式文档记录了以下实现细节：

- `.fig` 是包含 Kiwi 编码二进制消息的 ZIP 容器；
- 文件包含 schema、整个文档的 `NodeChange[]` 消息和图片/向量网络/字体等资源 blob；
- codec 覆盖 194 个 schema 定义和约 390 个 `NodeChange` 字段；
- 写回时重新编码 Kiwi、压缩数据并生成 Figma 需要的缩略图；
- 浏览器侧可以使用 deflate fallback，桌面端使用 Rust/Tauri 处理 Zstd 压缩；
- 未修改的 Figma 元数据可以尽量保留，已修改的对应字段会用新的 SceneGraph 值写回。

当前支持 `.fig` 的打开、保存、导出和 Figma clipboard 双向复制粘贴。它适合处理既有 Figma 资产，但不能把“字段被保留”理解为“字段已经可渲染或可编辑”。

### `.pen` 文件

`@open-pencil/pen` 是 Pencil `.pen` 文档解析器和 SceneGraph adapter。它会把 `.pen` 节点、文本、填充、描边、效果、Auto Layout、组件引用和变量映射到 OpenPencil 的共享设计模型。

但当前仓库格式参考表将 `.pen` 标为：

| 格式 | 打开/读取 | 保存/写入 | 导出 |
|---|---:|---:|---:|
| `.pen` | 支持 | 暂未支持 | 暂未支持 |

路线图明确把“让 `.pen` 成为跨 App、CLI、MCP 和 SDK 的一等可编辑保存目标”列为后续工作。因此，当前应把 `.pen` 能力理解为导入、解析和自动化使用，不应当当作完整的原生编辑和写回格式。

### 画布与布局

- CanvasKit/Skia 负责形状、文本、向量、渐变、图片填充、阴影、模糊和部分复杂效果的渲染；
- Yoga WASM 提供 Flex 和 Grid 布局，包括方向、间距、内边距、对齐、子项尺寸和网格轨道；
- SceneGraph 支持多页面、图层树、组件、实例、变量、Auto Layout、CSS Grid、向量网络和撤销/重做；
- 设计文档支持 PNG、JPG、WEBP、SVG、JSX、HTML 和 PPTX 等导出或转换路径，具体能力取决于文件模式、当前版本和目标对象。

## AI、MCP 与 Agent

### 内置 AI Chat

OpenPencil 的内置 AI Chat 可以通过用户自己的 API Key 连接 OpenRouter、Anthropic、OpenAI、Google、Z.ai、MiniMax 或兼容端点。AI 工具可以创建和修改节点、设置填充和描边、管理 Auto Layout、组件和变量，执行布尔操作、分析设计 token，并导出图片或代码。

README 将工具数量描述为 90+ 或 100+，具体数量会随主分支变化，因此不能把它视为固定 API 承诺。项目支持 BYOK，路线图也强调 AI 和媒体能力应保持由用户控制 Provider 和凭据。

### MCP Server

`@open-pencil/mcp` 提供两个命令：

```sh
npm install -g @open-pencil/mcp
openpencil-mcp       # stdio
openpencil-mcp-http  # Streamable HTTP
```

MCP 工具覆盖以下工作流：

- 文档：打开 `.fig`、创建空文档、列出文档和页面、保存文件；
- 读取：获取页面树、节点属性、边界、祖先、子节点、组件、变量、字体和当前选择；
- 创建：创建形状、向量、页面、组件、实例，或通过 JSX 一次生成节点树；
- 修改：调整填充、描边、效果、尺寸、布局、约束、文本、透明度、旋转、混合模式和可见性；
- 结构：复制、删除、重命名、分组、解组、重设父节点、布尔运算和排列；
- 导出：输出 PNG、JPG、WEBP、SVG，或获取当前视口和页面渲染；
- 分析：分析颜色、字体、间距、重复结构和节点差异；
- 脚本：通过 Figma Plugin API 兼容的 `eval` 执行更复杂的查询和批量修改。

MCP 支持文件模式和运行中应用模式。stdio 服务会发现运行中的 OpenPencil 应用；HTTP 服务适合浏览器扩展、脚本、CI 或其他 HTTP 客户端。

### MCP 安全边界

仓库文档给出的默认安全策略包括：

- Unix socket 和发现文件在 macOS/Linux 上使用仅所有者可读写的权限；
- TCP 默认绑定 `127.0.0.1`，默认端口为 7600；Windows 使用 TCP；
- 默认启用认证，并将生成的 token 存入私有发现文件；
- 文件操作可以通过 `OPENPENCIL_MCP_ROOT` 限定根目录，并拒绝符号链接逃逸；
- HTTP 模式默认禁用 `eval`；
- CORS 默认关闭，只有显式设置 `OPENPENCIL_MCP_CORS_ORIGIN` 才允许指定来源。

将 MCP 接入 Agent 时，应显式传递 `document_id` 和 `page_id`，避免多文档或多页面场景下误操作当前可见标签页。

### Agent Skill

项目提供可安装的 OpenPencil Agent Skill：

```sh
npx skills add open-pencil/skills@open-pencil
```

该 Skill 覆盖 CLI、MCP 工具、JSX 渲染、`eval` 和运行中应用的自动化桥接，可用于 Claude Code、Cursor、Windsurf、Codex 及其他支持 Skills 的 Agent。路线图下一步会继续完善“检查设计 → 执行修改 → 渲染/测量 → 比较 → 迭代”的完整闭环。

## CLI 与脚本化

`@open-pencil/cli` 的设计目标是让文件模式和运行中应用模式共用一套命令。指定文件时可以完全不打开编辑器：

```sh
npm install -g @open-pencil/cli
openpencil info design.fig
openpencil tree design.fig
openpencil find design.fig --type TEXT
openpencil query design.fig "//TEXT[contains(@name, 'Button')]"
openpencil lint design.fig --preset strict
openpencil analyze colors design.fig --json
openpencil export design.fig -f jsx --style tailwind
```

主要命令能力如下：

| 能力 | 说明 |
|---|---|
| `info`、`tree`、`node`、`pages` | 查看文档、页面、节点树和节点详情 |
| `find`、`query` | 按名称、类型或 XPath 查询节点 |
| `lint` | 检查命名、布局、结构和可访问性规则 |
| `analyze colors` | 统计颜色使用和调色板 |
| `analyze typography` | 统计字体、字号和字重组合 |
| `analyze spacing` | 检查间距和内边距的一致性 |
| `analyze clusters` | 发现可能应抽取为组件的重复结构 |
| `analyze overlaps` | 查找重叠或溢出的节点 |
| `export` | 导出位图、SVG、JSX、HTML 或 `.fig` 页面/节点 |
| `convert` | 在支持的文档格式之间转换 |
| `import` | 将 HTML、CSS、Tailwind 转成可编辑设计文档 |
| `eval` | 运行 Figma Plugin API 风格的 JavaScript |

所有命令支持 `--json` 或结构化输出，适合接入 CI、脚本和 Agent。`eval` 可以使用 `--write` 原地写入，或使用 `--output` 生成新文件；省略文件路径时，CLI 可以通过 RPC 控制正在运行的桌面应用。

## Vue SDK 与可嵌入能力

OpenPencil 的 SDK 分为三个层次：

1. `@open-pencil/core`：框架无关的编辑器引擎；
2. `@open-pencil/vue`：Vue 注入、响应式 composables、headless primitives 和画布接入；
3. 业务应用：自行负责样式、路由、文件流程和产品化交互。

SDK 已提供：

- 编辑器上下文和 `provideEditor`/`useEditor`；
- CanvasKit 画布接入；
- 选择、命令、菜单、页面、图层树和变量相关 composables；
- 属性面板、布局、外观、字体、填充、描边、效果和导出控件；
- `PageListRoot`、`PropertyListRoot`、`PropertySectionRoot`、`ToolbarRoot` 等 headless 结构组件；
- 国际化、颜色模型、渐变、约束和自定义文档工作区能力。

这使 OpenPencil 适合构建只读预览、定制属性面板、内部模板编辑器、Agent 控制的专用设计界面和嵌入式设计功能。需要注意，路线图仍将公共 API 稳定性、完整示例应用和嵌入式编辑器能力列为持续完善事项。

## 设计到代码与网页导入

OpenPencil 将设计和代码之间的转换拆成两个方向：

- **设计到代码**：将选区或页面导出为 JSX、Tailwind JSX、HTML、SVG 或其他目标格式；
- **代码到设计**：通过 `@open-pencil/dom-css` 和 CLI 导入 HTML、CSS、Tailwind，生成可编辑的设计节点，而不是只生成一张截图。

示例：

```sh
openpencil import card.html --css card.css -o card.fig
openpencil import card.html --tailwind "flex flex-col gap-3 w-80 p-6 rounded-xl bg-white" -o card.fig
openpencil export design.fig -f html --html standalone --css tailwind
```

此外，Figma Plugin API 风格的 `eval` 为批量读取、修改、生成和迁移提供了比单个 MCP 工具更低层的脚本接口。

## 协作与部署

OpenPencil 的实时协作使用 WebRTC P2P、Trystero 和 Yjs CRDT，当前特点是：

- 不要求中心服务器和账号；
- 通过分享链接进入协作房间；
- 支持光标、在线状态、选区和跟随视口；
- 使用本地持久化保存房间状态。

这适合本地优先和轻量协作试验，但不应等同于已经完整的团队云平台。路线图把以下能力列为后续方向：

- OpenPencil Cloud 和可维护的自托管部署；
- 组织、成员、角色、OIDC/SSO、服务账号和 API Token；
- 版本历史、命名检查点、恢复和审计；
- 持久协作中继、受限网络支持和同步健康状态；
- 存储、导出、删除、加密、保留和数据驻留治理。

因此，当前更准确的定位是“本地文件优先 + P2P 协作 + 可选远程存储”，而不是成熟的企业设计 SaaS 后端。

## Figma 兼容性与产品缺口

OpenPencil 的兼容性文档截至 2026-03-07 记录了 158 个 Figma 功能项中的 76 项完整支持、18 项部分支持、64 项尚未实现。这个统计说明项目已经覆盖大量基础编辑能力，但仍不能视为完整 Figma 替代品。

当前文档明确列出的缺口或部分支持包括：

- 原型连接、触发器、动作、动画、过渡、Smart Animate、Overlay 和完整预览；
- 评论、版本历史、Code Connect 和 Figma Dev Mode 体验；
- 完整的命名样式系统；
- 变量的完整编辑 UI 和变量驱动主题；
- 部分布尔运算、遮罩、模式填充、噪声/自定义填充和变量字体场景；
- 组件属性、插槽、共享库和发布更新流程；
- 非拉丁字体、阿拉伯语/波斯语 shaping、RTL 和更广泛的 CJK 场景。

项目会保留许多未完全建模的 Figma/Kiwi 字段以提高 round-trip fidelity，但这些字段可能只在导出时保留，未必能在画布中渲染或在 UI 中编辑。

## 适用场景

| 场景 | OpenPencil 的适配度 | 说明 |
|---|---|---|
| 直接打开、修改和批量检查 Figma `.fig` | 高 | 有原生 Kiwi codec、CLI、MCP 和 Figma API 兼容入口 |
| 在 CI 中做设计树检查、lint、颜色/字体/间距分析 | 高 | CLI 支持文件模式、JSON 输出和分析命令 |
| 让 Agent 执行“读取 → 修改 → 渲染 → 保存” | 高 | MCP、CLI 和内置工具覆盖完整操作链 |
| 构建嵌入式或业务定制编辑器 | 中高 | 有 Core、SceneGraph 和 Vue SDK，但需要自行承担集成与升级 |
| 将 HTML/CSS/Tailwind 变成设计文档 | 中高 | 有 DOM/CSS 包和 CLI 导入命令 |
| 使用 `.pen` 作为完整可写原生格式 | 当前较低 | 当前主要支持读取和转换，写回能力仍在路线图中 |
| 需要完整企业云治理和版本审计 | 当前较低 | Cloud、自托管、权限、版本历史和审计仍在路线图中 |

## 验证和落地建议

如果要在实际项目中采用 OpenPencil，建议先固定版本并进行以下验证：

1. 使用包含组件、实例、变量、Auto Layout、图片、渐变、文本和向量的 `.fig` 文件，测试打开、保存、再次打开和 Figma 对照渲染。
2. 分别测试浏览器、桌面端和 CLI 的字体加载、图片资源、压缩、导出和错误恢复。
3. 让 Agent 完成“读取页面树 → 修改组件 → 导出截图 → 保存文件”的闭环，记录工具审批、失败恢复、结构差异和视觉差异。
4. 在 CI 中验证 `--json`、`lint`、`analyze`、`eval`、HTML/CSS 导入和导出结果是否稳定。
5. 对 MCP 配置检查 `OPENPENCIL_MCP_ROOT`、认证 token、TCP 监听地址、CORS、`eval` 权限和多文档 `document_id/page_id` 选择。
6. 如果使用 Vue SDK，建立自己的 API 适配层和版本升级测试，不要把当前主分支导出接口视为长期稳定协议。

## 风险与状态说明

- 仓库 `master` 在本次调研时已经有比 `v0.14.0` Release 更新的提交。实际安装包与主分支文档、API 和工具数量可能不同，应固定版本后再评估。
- `.fig` 兼容性仍在持续提高，不是所有 Figma 字段都能被渲染或编辑；保留元数据也不等于功能可用。
- `.pen` 能力目前容易被误解。`@open-pencil/pen` 可以解析并适配 SceneGraph，但格式参考和路线图仍明确显示写入能力未完成。
- Vue SDK 已发布，但公共 API 稳定性、示例应用和嵌入式编辑器能力仍在完善；采用它需要关注版本迁移。
- P2P 协作适合本地优先场景，但企业级身份、版本、审计、备份和网络治理需要等待后续 Cloud/自托管能力，或自行补齐。
- README 中的工具数量会随主分支变化，应以固定版本实际暴露的 MCP schema 和 CLI 命令为准。

## 资料来源

- [GitHub 仓库](https://github.com/open-pencil/open-pencil)
- [README：能力、安装、CLI、AI、MCP、SDK 与技术栈](https://github.com/open-pencil/open-pencil/blob/master/README.md)
- [最新 Release v0.14.0](https://github.com/open-pencil/open-pencil/releases/tag/v0.14.0)
- [MCP Server 文档](https://openpencil.dev/programmable/mcp-server)
- [CLI：Inspecting Files](https://openpencil.dev/programmable/cli/inspecting)
- [CLI：Exporting](https://openpencil.dev/programmable/cli/exporting)
- [CLI：Scripting](https://openpencil.dev/programmable/cli/scripting)
- [CLI：Analyzing Designs](https://openpencil.dev/programmable/cli/analyzing)
- [文件格式参考](https://openpencil.dev/reference/file-format)
- [Figma 兼容性参考](https://openpencil.dev/reference/figma-compatibility)
- [Vue SDK](https://openpencil.dev/programmable/sdk/)
- [OpenPencil 路线图](https://openpencil.dev/development/roadmap)
