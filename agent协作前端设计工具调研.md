# 适合 Agent 协作前端设计的工具调研

调研日期：2026-08-18  
目标：筛选能够制作原型图、线框图、UI 图，并且适合与 Codex 或其他编码 Agent 协作的工具。

## 结论先行

### 最推荐的三种组合

1. **团队协作与正式交付：Figma + Figma MCP + Codex**
   - Figma 仍然是最稳妥的设计资产中心：支持多人协作、组件、变量、Auto Layout、原型和交付检查。
   - 官方 Figma MCP 已明确支持 Codex，既能把 Figma 设计上下文带入代码，也能让 Agent 创建或修改 Figma 原生内容。
   - 适合有设计师、需要评审、需要设计系统和长期维护的产品团队。

2. **Agent-first、设计与代码双向迭代：Paper + Codex**
   - Paper 的 MCP 支持读取和写入设计文件，官方文档提供 Codex 插件和本地 Streamable HTTP 接入方式。
   - 设计基于 HTML/CSS，Agent 可以读取节点、截图、计算样式、生成 JSX，也可以修改文本、节点和画板。
   - 适合希望 Codex 直接参与 UI 设计、快速反复调整，并把设计和真实代码放在同一工作流中的团队。

3. **代码库内的设计即代码：Pencil（pen.dev）+ Codex CLI**
   - Pencil 运行在 IDE 中，使用可提交到 Git 的 `.pen` 文件；官方文档明确列出 Codex CLI 支持。
   - 支持设计到代码、代码到设计，以及 Figma 文件导入；设计文件可以和前端代码放在同一仓库。
   - 适合工程师主导、重视 Git 分支和代码审查、不希望设计文件脱离代码仓库的团队。

### 我的推荐顺序

如果只能选一个：**先选 Figma**。它的团队协作和设计资产能力最完整，而且官方 Codex MCP 支持已经比较明确。

如果主要由工程师和 Agent 共同做设计：**优先试 Paper 或 Pencil**。Paper 更偏共享画布和可视化协作，Pencil 更偏本地文件、Git 和设计即代码。

如果还没有明确产品结构：**Relume 或 Stitch 做前置发散，再进入 Figma/Paper/Pencil**。不要把它们直接当作长期设计系统的唯一来源。

## 评分说明

评分为本次调研对“Agent 协作前端设计”场景的主观评估，满分 5 分：

- **Agent 接入**：是否有官方 MCP、Codex 支持或稳定的 API/SDK。
- **设计产出**：能否做线框、UI、组件、原型和多屏流程。
- **代码衔接**：从设计到代码、从代码回设计的能力。
- **协作与资产**：多人协作、版本管理、设计系统和可编辑性。
- **适合场景**：是否适合作为实际项目的长期工具，而不只是一次性生成。

| 工具 | Agent 接入 | 设计产出 | 代码衔接 | 协作与资产 | 综合判断 |
| --- | ---: | ---: | ---: | ---: | --- |
| Figma + MCP | 5 | 5 | 5 | 5 | 综合最稳，适合作为设计源 |
| Paper + MCP | 5 | 4 | 5 | 4 | Agent-first，适合设计代码闭环 |
| Pencil / pen.dev | 5 | 4 | 5 | 3 | 代码库内设计，适合工程团队 |
| Google Stitch | 3 | 5 | 4 | 2 | 发散和高保真初稿很强 |
| Relume | 3 | 4 | 4 | 4 | 网站信息架构和线框很强 |
| v0 | 3 | 3 | 5 | 3 | 直接生成可运行前端 |
| Lovable | 5 | 3 | 5 | 3 | Agent 驱动的全栈原型和部署 |
| Uizard | 2 | 5 | 2 | 4 | 非设计师快速做多屏原型 |
| Visily | 2 | 4 | 2 | 4 | 截图/草图/文字转可编辑 UI |
| Penpot | 2 | 4 | 3 | 5 | 开源、自托管和团队协作优先 |

## 工具详解

### 1. Figma：最适合作为团队设计源

**能力**

- 设计、组件、变量、Auto Layout、原型、FigJam 流程图和 Figma Make 均在同一生态中。
- Figma MCP 可以向 Agent 提供设计文件上下文和代码资源。
- 官方文档说明 Agent 可以创建和修改 Figma Design、FigJam、Figma Slides 中的原生内容，包括 Frame、Component、Variable 和 Auto Layout。
- 支持把本地或线上 Web UI 捕获回 Figma，形成可编辑设计层，用于对齐和继续迭代。
- Code Connect 可以把 Figma 组件和代码库中的真实组件关联起来，减少 Agent 重新生成相似组件。

**与 Codex 的适配**

- Figma 官方帮助文档明确把 Codex 列为支持 MCP 的客户端，并提供 Codex 安装入口。
- 推荐使用官方 Remote MCP，地址为 `https://mcp.figma.com/mcp`；桌面 MCP 主要适用于特定组织和企业场景。
- 常见工作方式是：在 Figma 复制选中图层链接，在 Codex 中要求读取该节点并实现；完成后再让 Codex 启动本地服务，把 UI 捕获回 Figma 做视觉核对。

**限制**

- MCP 的写画布能力处于快速迭代阶段，官方说明未来可能按用量收费，目前 Beta 阶段免费。
- 设计文件越大、层级越深，Agent 读取和生成越容易变慢或失真；应按页面、模块、组件拆分任务。
- 需要认真维护组件命名、变量和 Code Connect，否则 Figma 只是“截图来源”，不能充分发挥设计系统价值。

**适合**

正式产品、多人设计评审、需要设计系统、需要交付给开发和长期维护的项目。

来源：

- [Figma MCP Developer Docs](https://developers.figma.com/docs/figma-mcp-server/)
- [Figma MCP 官方帮助文档](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server)
- [Figma Dev Mode](https://www.figma.com/dev-mode/)

### 2. Paper：最贴近 Agent 原生设计闭环

**能力**

- Paper 是基于 Web 标准的设计画布，官方定位是把团队、Agent、代码和数据放在同一设计空间。
- Paper MCP 支持读取和写入设计文件，可以读取节点树、截图、JSX、计算样式和字体信息，也可以创建画板、写入 HTML、修改文本、复制节点和导出资源。
- 官方文档提供从设计生成 React/Tailwind 网站、基于多个画板生成响应式断点、从 Figma 同步设计 token、从 Notion 等来源同步真实内容的流程。
- 可以从 Figma、HTML 等来源粘贴设计，适合把已有资产逐步迁移到 Agent 工作流。

**与 Codex 的适配**

- 官方 Paper MCP 文档提供 Codex 插件方式：添加 `paper-design/agent-plugins` marketplace，然后安装 `paper-desktop` 插件。
- 也支持手动方式：在 Codex 的 MCP Servers 中添加 Streamable HTTP，地址为 `http://127.0.0.1:29979/mcp`。
- 需要安装 Paper Desktop；打开文件后会在后台启动本地 MCP 服务。

**限制**

- 本地 MCP 依赖 Paper Desktop，团队成员需要统一桌面环境和版本。
- 设计到代码的质量高度依赖画布结构。官方建议使用 Flex 布局和容器，并从较小的 Frame 开始。
- Paper 的团队协作、生态成熟度和设计系统深度仍不应默认等同于 Figma；更适合作为 Agent-first 工具或工程团队的补充。

**适合**

Agent 直接参与设计、代码与设计需要频繁往返、希望使用真实内容和真实数据做 UI 的前端团队。

来源：

- [Paper MCP Server 官方文档](https://paper.design/docs/mcp)
- [Paper 官方主页](https://paper.design/)

### 3. Pencil / pen.dev：最适合 Git 化的设计即代码

> 产品和文档目前使用 `pen.dev` 名称；部分 Codex MCP 列表和历史文档仍可能显示 `pencil`，这是同一工具链的命名差异。

**能力**

- Pencil 直接集成在开发环境中，不要求设计和代码分离在两个应用里。
- 设计文件是 `.pen` 格式，可以和代码一起放进项目仓库，并通过 Git 查看差异、提交、分支和合并。
- 支持变量、组件、Slots、设计库和 Code on Canvas。
- 支持设计到代码、代码到设计的双向工作流；官方示例覆盖 React、Next.js、Tailwind、Shadcn UI、React Hook Form 和 Lucide。
- 支持完整 Figma 文件导入、单独图层复制粘贴以及 PNG/JPEG/WEBP/PDF 导出。

**与 Codex 的适配**

- 官方 AI Integration 文档明确列出 Codex CLI，并说明 MCP Server 在本地运行。
- Agent 可以读取和修改 `.pen` 文件；设计文件与代码处于同一 workspace，便于 Codex 同时理解两者。
- 适合把设计变更和代码变更放在一个 Git 提交或同一个功能分支里。

**限制**

- 更适合工程团队，不一定适合有复杂设计运营、多人设计评审和大量外部协作者的组织。
- 文档说明代码仓库尚未公开，生态和第三方插件成熟度需要实际试用确认。
- 与 Figma 的转换不是完全无损，复杂组件和高级设计语义仍需要人工检查。

**适合**

代码库驱动的产品、独立开发者、小型前端团队、需要让设计文件参与 Git 工作流的项目。

**当前需要特别注意的问题**

- **不是完全离线工具。** 官方定价页显示 pen.dev 目前免费，但需要邮箱激活；AI 集成文档和安装文档还要求安装并登录 Claude Code。使用 Codex 作为 MCP 客户端时，建议把 pen.dev 激活和 `claude` 登录都完成，再验证 Codex 的 MCP 连接。Codex 自身的模型/账户费用仍按 Codex 方案计算，不能把“pen.dev 免费”理解为所有 AI 调用免费。
- **Windows 文档有状态不一致。** 下载页列出了 Windows x64 桌面包，但故障排除页又写着 Windows 桌面应用“当前不可用”。在 Windows 上优先使用 VS Code 或 Cursor 扩展；不要把桌面包作为唯一安装路径。
- **可能改写或重复 Codex 配置。** 官方故障排除页明确提示 pen.dev 可能修改或重复 Codex 的 `config.toml`，首次使用前应备份配置，并检查 MCP 条目是否重复。
- **没有自动保存和实时多人协作。** 官方文档说明当前没有 Auto-save，也没有实时多人编辑；协作主要依靠 Git 分支、提交、推送和 PR。设计文件丢失风险比 Figma 更高，必须形成频繁保存和提交习惯。
- **撤销/重做能力有限。** 官方建议在重大修改前保存，使用 Git 历史作为主要回退机制；不要把 pen.dev 当作已经成熟的 Figma 替代品。
- **设计导出可能和画布不一致。** 官方故障排除页承认导出结果和画布存在视觉不一致的已知问题；需要用截图作为参考，并在浏览器中做最终视觉验证。
- **Windows 用户没有完整桌面体验。** 即使扩展可以工作，部分桌面应用级功能、权限提示和文件访问行为仍可能受 IDE 或 Windows 权限影响。

来源：

- [pen.dev 官方主页](https://pen.dev/)
- [pen.dev 定价](https://pen.dev/pricing)
- [pen.dev 下载页](https://pen.dev/download)
- [pen.dev AI Integration](https://docs.pen.dev/getting-started/ai-integration)
- [pen.dev Installation](https://docs.pen.dev/getting-started/installation)
- [pen.dev Authentication](https://docs.pen.dev/getting-started/authentication)
- [pen.dev Design ↔ Code](https://docs.pen.dev/design-and-code/design-to-code)
- [pen.dev Import & Export](https://docs.pen.dev/core-concepts/import-and-export)
- [pen.dev Design as Code](https://docs.pen.dev/core-concepts/design-as-code)
- [pen.dev Troubleshooting](https://docs.pen.dev/troubleshooting)

### 4. Google Stitch：高保真 UI 发散和初稿生成器

**能力**

- 通过文字和图片输入生成 UI 设计和前端代码。
- 可以从白板草图、截图或粗略线框生成数字化 UI。
- 支持交互式对话、主题选择、生成多种视觉方向，并可粘贴到 Figma。
- Google 官方将其定位为 Google Labs 的实验项目，因此应把它看作快速探索工具，而不是默认的长期设计源。

**与 Codex 的适配**

- 官方公开资料显示 Stitch SDK 可以程序化生成 UI screen、读取 HTML 和截图，并提供设计系统 API；这为 Agent 集成提供了基础。
- 但本次核验没有发现 Google 官方明确提供 Codex 专属连接器，当前更稳妥的方式是 Stitch 生成初稿后粘贴到 Figma，再由 Figma MCP + Codex 落地。

**限制**

- 设计协作、版本管理、复杂设计系统和长期交付能力不如 Figma。
- 生成结果需要人工检查组件结构、响应式行为、可访问性和真实内容适配。

**适合**

需求早期快速探索风格、一次生成多个页面方向、从截图/草图快速得到高保真参考图。

来源：

- [Google Developers：Introducing Stitch](https://developers.googleblog.com/stitch-a-new-way-to-design-uis/)
- [Stitch](https://stitch.withgoogle.com/)
- [Google Stitch SDK](https://github.com/google-labs-code/stitch-sdk)

### 5. Relume：网站信息架构和线框图效率最高

**能力**

- 根据几句话生成网站 Sitemap。
- 可以将 Sitemap 一键转换为未定制样式的 Wireframe，并使用真实组件和文案。
- 提供 1000+ Web/Figma 组件，支持移动端变体。
- 可以导出或导入到 Figma、Webflow 和 React 工作流。

**与 Codex 的适配**

- Relume 更像前置产品/网页结构工具，而不是 Codex 的直接 MCP 工具。
- 推荐流程是 Relume 生成 Sitemap 和 Wireframe，导入 Figma 或导出 React 后，再由 Codex 负责项目实现和细节调整。
- 对营销站、官网、文档站和内容型网站特别合适；对复杂后台、移动端 App 和高度定制的产品 UI，优势会下降。

**限制**

- 组件库有明显的 Webflow/营销网站倾向，容易生成较模板化的页面。
- 设计系统和交互逻辑仍需进入 Figma、Paper 或代码库继续整理。

来源：

- [Relume 官方主页](https://www.relume.ai/)
- [Relume 资源文档](https://www.relume.ai/resources)
- [Relume Figma 导出说明](https://www.relume.io/resources/docs/figma-plugin)

### 6. v0：从提示词直接生成可运行前端

**能力**

- 官方文档将 v0 定位为生成真实代码、全栈应用和 Agent 的 AI 工具。
- 支持提示词生成、代码编辑、外部 API、GitHub、Figma、MCP 集成和实时原型。
- 适合把 UI 想法快速变成可运行的 React/Next.js 代码，再在本地继续开发。

**与 Codex 的适配**

- v0 自身已有 Agent 和代码工作区，与 Codex 的直接协作重点在 GitHub、代码导出和项目仓库，而不是把 v0 当作设计画布。
- 更适合“先让 v0 给出可运行初稿，再交给 Codex 接管代码库”的流程。

**限制**

- 它的核心产物是代码和实时原型，不是成熟的可编辑设计文件。
- 视觉稿与代码之间的设计 token、组件语义和长期版本管理需要团队自己补齐。

来源：

- [v0 官方文档](https://v0.dev/docs)
- [v0 官方网站](https://v0.dev/)

### 7. Lovable：Agent 驱动的全栈可运行原型

**能力**

- Lovable 官方 MCP 能让兼容 MCP 的客户端创建、编辑、部署和管理 Lovable 项目。
- 提供项目/工作区、Agent 对话、代码检查、知识库、数据库、连接器和部署等能力。
- 官方 Figma FAQ 说明 Lovable 主要导出到代码，而不是导出为 Figma；可以粘贴 Figma 链接或上传截图作为输入。

**与 Codex 的适配**

- 官方 README 明确提供 Codex CLI 接入：`codex mcp add lovable --url https://mcp.lovable.dev`，然后执行登录。
- 适合由 Codex 管理远端 Lovable 项目、让 Lovable Agent 生成一轮功能，再由 Codex 查看 diff、继续开发或接管代码。

**限制**

- `create_project` 和 `send_message` 会消耗 Lovable build credits，部署会发布公开 URL，使用前要明确权限和成本。
- 更偏“生成应用”和“快速验证业务流程”，不是专业 UI 设计系统或原型评审工具。
- 代码、数据库和部署状态受平台约束，正式项目需要评估迁移成本。

来源：

- [Lovable MCP 官方仓库](https://github.com/lovablelabs/mcp)
- [Lovable MCP 文档](https://docs.lovable.dev/integrations/lovable-mcp-server)
- [Lovable Figma FAQ](https://lovable.dev/faq/design/figma)

### 8. Uizard / Visily：原型和 UI 参考图工具

**Uizard**

- 官方定位是面向 App、网站和桌面软件的 AI UI 设计工具。
- 支持文本生成多屏可编辑原型、截图转可编辑 Mockup、手绘线框转数字设计、组件级 AI 修改和主题生成。
- 适合产品经理、非设计师快速表达想法；本次未发现官方 Codex MCP 接入，建议通过图片、链接或导出资产交给 Codex，而不是作为 Codex 的设计源。

来源：[Uizard 官方主页](https://uizard.io/)

**Visily**

- 官方定位是面向非设计师的 AI UI 设计软件。
- 支持从文字、截图、图表、模板或组合输入开始，编辑高保真线框和原型，并提供实时评论、共享库和协作模式。
- 适合快速做需求沟通图、竞品复刻参考和多屏线框；本次未发现官方 Codex MCP 接入。

来源：[Visily 官方主页](https://visily.ai/)

### 9. Penpot：开源和自托管优先

- Penpot 官方定位是面向团队的开源设计平台，覆盖 UI 设计、设计系统、原型、协作、代码和 AI 工作流。
- 如果团队对数据控制、自托管、开源许可或供应商锁定敏感，值得纳入候选。
- 本次核验没有发现 Penpot 官方的 Codex MCP 连接器；可以通过 API、导出文件或社区 MCP 间接接入，但不应把社区方案当作官方能力。

来源：

- [Penpot 官方主页](https://penpot.app/)
- [Penpot GitHub](https://github.com/penpot/penpot)

## 推荐工作流

### 工作流 A：正式产品团队

```text
需求/流程梳理
  -> Figma / FigJam
  -> Figma 组件、变量、Code Connect
  -> Figma MCP + Codex 实现
  -> 本地浏览器检查响应式和交互
  -> Codex 将本地 UI 捕获回 Figma
  -> 设计评审与下一轮迭代
```

### 工作流 B：工程师和 Agent 主导

```text
需求
  -> Paper 或 Pencil 生成小范围 Frame
  -> Codex 读取节点、截图和样式
  -> 生成 React/Tailwind 组件
  -> 浏览器验证
  -> Codex 根据视觉差异修改设计或代码
  -> Git 提交设计文件与代码
```

### 工作流 C：快速探索后正式落地

```text
Stitch / Relume / Uizard / Visily
  -> 选定方向
  -> 导入或粘贴到 Figma
  -> 整理组件、变量、断点和真实内容
  -> Figma MCP + Codex 实现
```

### 工作流 D：快速验证业务闭环

```text
v0 / Lovable
  -> 生成可运行原型
  -> GitHub 或项目仓库
  -> Codex 接管代码、补齐测试和工程结构
  -> 再把稳定的 UI 组件整理回 Figma/Paper/Pencil
```

## 落地建议

1. **不要让截图成为唯一设计交付物。** 至少保留组件、变量、断点、交互状态和真实文案；否则 Codex 只能做视觉猜测。
2. **先从单个页面或单个组件验证。** 让 Agent 读取一个小 Frame，生成代码并回看截图，确认工具链稳定后再扩大到完整流程。
3. **统一设计文件命名和组件语义。** 例如明确 `Button/Primary/Default`、`Input/Error`、`spacing/md`、`color/surface` 等，能明显提升 MCP 输出的一致性。
4. **让 Codex 负责视觉回归。** 每次设计改动都应启动本地服务，检查桌面、平板和移动端截图，而不是只看生成代码是否能编译。
5. **保留代码库的真实组件作为最终约束。** Figma Code Connect、Paper 的 JSX/CSS 上下文或 Pencil 的同仓库 `.pen` 文件，都应指向实际代码组件，而不是另建一套“看起来相同”的组件。
6. **先做小型试点。** 建议用一个登录页、一个后台列表页和一个移动端详情页，分别测试：设计读取、组件复用、响应式、图片/字体、交互状态和视觉回写。

## 最终选型

对于大多数希望使用 Codex 协作前端设计的团队，我建议：

- **主工具：Figma + Figma MCP**，作为正式设计资产和团队协作中心。
- **Agent-first 补充：Paper**，用于让 Codex 直接生成、读取和修改设计画布。
- **工程化补充：Pencil**，用于把设计文件放进代码仓库，并参与 Git 工作流。
- **发散工具：Stitch 或 Relume**，分别用于视觉方向探索或网站结构线框。
- **可运行原型：v0 或 Lovable**，用于验证业务流程，不作为唯一设计系统。

本次联网调研使用了 `agent-reach` 的 Exa 搜索和 GitHub/网页阅读路径；Exa 免费通道在首轮多查询后触发限流，后续改用已确认的官方页面直接核验。`agent-reach check-update` 当前版本为 `v1.5.0`，但 GitHub API 触发速率限制，未能完成更新检查。
