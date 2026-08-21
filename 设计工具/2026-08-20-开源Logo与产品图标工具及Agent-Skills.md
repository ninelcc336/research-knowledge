---
title: "开源 Logo、产品图标工具及 Agent Skills 调研"
date: 2026-08-20
category: "设计工具"
tags:
  - Logo设计
  - 产品图标
  - SVG
  - 开源设计工具
  - Agent Skills
  - 资产导出
  - 图标库
---

# 开源 Logo、产品图标工具及 Agent Skills 调研

调研日期：2026-08-20  
目标：筛选可以制作 Logo、产品图标和配套品牌资产的开源工具、资产流水线和 Agent Skills，并给出适合工程团队的组合方式。

## 结论先行

1. **需要制作最终 Logo 主稿：优先 Inkscape。** 它以 SVG 为原生格式，适合路径、布尔运算、文字沿路径、精确对齐和多格式导出；最终交付应保留可编辑 SVG，而不是只保存 PNG。
2. **需要团队协作或自托管：优先 Penpot。** 它支持 SVG、CSS、HTML、JSON 等开放格式，也支持组件、设计令牌、实时协作、自托管、开放 API 和 MCP；它更像产品设计基础设施，不是专门的品牌策略工具。
3. **需要让 Agent 参与 Logo 迭代：优先采用“可编辑 SVG + 尺寸检查 + 导出”的工作流。** `logo-designer-skill` 适合生成多方向 Logo 概念，`inkpilot` 适合在 VS Code 内通过 MCP 编辑 SVG，`open-assets` 适合把源文件批量渲染成各尺寸资产。
4. **需要批量生产 App Icon、favicon 和 wordmark：使用 `open-assets` 或 `brand`。** 它们不是专业矢量绘图软件，而是把已确认的 SVG/HTML 源文件稳定导出到多个尺寸和平台。
5. **产品界面图标不应全部交给图像模型生成。** 优先从 Lucide、Tabler、Phosphor 等统一风格的 SVG 图标库中选择或改造；Iconify 适合检索和接入多个图标集，但每个图标集的许可证要单独核对。
6. **不要把位图生成或 PNG 自动描摹直接当成 Logo 终稿。** 图像模型适合发散、材质和展示图，品牌主稿仍应经过人工几何整理、字体授权核查、商标相似性检查和小尺寸测试。

本次把“开源”理解为项目或对应 skill 文件明确提供 MIT、Apache、GPL、MPL、ISC、CC0 等公开许可证。只写“免费”但没有许可证的项目，不按已确认开源处理。GitHub 维护状态按 2026-08-20 的页面快照判断，星标数量不作为质量结论。

## 工具总览

| 工具 | 类型 | 许可证 | 最适合 | 关键输出或能力 | 主要注意事项 |
| --- | --- | --- | --- | --- | --- |
| [Inkscape](https://inkscape.org/) | 桌面矢量编辑器 | GPL（项目官网称 GPL） | 最终 Logo、图标主稿、复杂路径 | 原生 SVG、布尔运算、文字路径、PNG/PDF/EPS 等导出、命令行转换 | 专业绘制能力强，但团队协作和设计资产管理需要另外搭配工具 |
| [Penpot](https://penpot.dev/) | Web 设计与原型平台 | MPL-2.0 | 团队协作、自托管、产品设计系统 | SVG/CSS/HTML/JSON、组件、设计令牌、实时协作、API、MCP | 更偏产品设计与系统化协作，品牌细节仍需人工精修 |
| [SVG-Edit](https://github.com/SVG-Edit/svgedit) | 可嵌入的 Web SVG 编辑器 | MIT | 浏览器内编辑、二次开发、内部工具 | Web 编辑器和可复用的 `@svgedit/svgcanvas` | 适合定制和集成，复杂排版与专业品牌工作流不如 Inkscape |
| [Graphite](https://github.com/GraphiteEditor/Graphite) | 程序化矢量/栅格编辑器 | Apache-2.0 | 几何探索、程序化 Logo、非破坏编辑 | 节点图、非破坏工作流、矢量与栅格处理 | 当前仍是 alpha，不宜作为唯一的生产交付工具 |
| [Method Draw](https://github.com/methodofaction/Method-Draw) | 轻量 Web SVG 编辑器 | MIT | 快速画简单图形、提供在线编辑入口 | 简洁的 SVG 编辑体验、可本地部署 | 明确移除了图层等复杂能力，README 的版本说明停留在 2021 年，维护成熟度需自行评估 |
| [open-assets](https://github.com/Parra-Inc/open-assets) | 本地资产预览与导出工具 | MIT | Logo、App Icon、favicon、商店截图的批量导出 | HTML/CSS/SVG 模板、实时预览、增量渲染、多尺寸、Xcode asset catalog、复制 SVG 源文件 | 不是矢量绘图软件，需先准备源模板 |
| [brand](https://github.com/macalinao/brand) | Rust 品牌资产 CLI | Apache-2.0 | 已有图标后的品牌资产生成 | 从 SVG 图标和字体配置生成 favicon、App Icon、Logo、wordmark、manifest 和 HTML head 片段 | 不能替代 Logo 创意和几何设计；依赖字体与 SVG 源文件质量 |
| [icon-generator](https://github.com/schwa/icon-generator) | Swift App Icon CLI | MIT | Apple 平台 App Icon 快速打包 | PNG、SVG、`.appiconset`、Xcode Icon Composer `.icon`、SF Symbols、渐变和角标 | 偏 Apple/Xcode，不能当作跨平台品牌设计工具 |
| [inkpilot](https://github.com/jamesmichaelredmond/inkpilot) | VS Code 扩展与 MCP SVG 编辑器 | MIT | 让 Agent 在编辑器里创建和修改 Logo、图标、徽章 | 实时 SVG 预览、14 个 MCP 工具、5 个设计提示、`.inkp` 项目文件、SVG 导出 | 项目很新、社区信号较弱；官网说明支持 MCP 客户端，但没有把 Codex 作为专属客户端核验过 |

## 1. 矢量绘制与协作工具

### Inkscape：最稳妥的 Logo 主稿工具

Inkscape 官方 FAQ 明确把 Logo、图标、技术插图和网页图形列为 SVG 矢量图的典型用途；官方功能页列出原生 SVG 编辑、XML 编辑器、PNG/OpenDocument/DXF/PDF/AI/EPS/PostScript 等导出以及命令行转换能力。

它适合承担以下工作：

- 画几何标志、字母标、抽象符号和 wordmark；
- 做路径合并、节点清理、描边转路径、文字转路径和 viewBox 修整；
- 输出 Logo 的彩色、黑白、反白、图标、横版、竖版和单色变体；
- 在 CI 或资产脚本中调用命令行，把 SVG 转成 PNG/PDF 等交付格式。

**判断：** 如果只选一个真正负责“设计和修整 Logo 主稿”的开源工具，选 Inkscape。它的缺点是多人协作、设计令牌、审阅流程和 Agent 上下文管理不如 Penpot，需要通过 Git、文件命名和导出脚本补齐。

### Penpot：更适合产品团队的开源设计基础设施

Penpot 官方仓库和网站把它定位为面向产品团队的开源设计平台。它支持自托管、实时协作和 SVG/CSS/HTML/JSON 等开放标准；仓库还说明了设计令牌、组件、变量、插件、开放 API 以及 MCP 相关能力。

在 Logo/产品图标项目中，它的价值主要是：

- 把 Logo、产品界面、组件库和设计令牌放在同一个可协作空间；
- 让设计师、产品经理和工程师共同评审不同尺寸和明暗背景；
- 通过开放格式和检查面板把设计交给代码实现；
- 在需要内网部署、权限治理或数据自托管时保留控制权。

**限制：** Penpot 解决的是设计协作和系统化管理，不会自动替代品牌定位、字体设计或 Logo 几何精修。对于一个单独的 Logo，Inkscape 通常更直接；对于 Logo 和整个产品设计系统一起建设，Penpot 更有价值。

### SVG-Edit：适合嵌入产品的 SVG 编辑器

SVG-Edit 是基于浏览器的 JavaScript SVG 编辑器，项目拆成编辑器界面和底层 `@svgedit/svgcanvas`。README 提供了本地构建方式，也说明底层画布可以被其他 Web 应用和框架复用。

它适合做：

- 公司内部的 Logo/徽标定制器；
- 允许用户编辑 SVG 的在线产品功能；
- 需要把基础形状编辑能力嵌入现有 Web 应用的场景。

**限制：** 如果目标是高质量品牌主稿，它更像可集成的编辑器组件，不像 Inkscape 那样是完整的专业桌面设计工具。

### Graphite：适合程序化和非破坏式探索

Graphite 采用节点图驱动的非破坏编辑，官方仓库说明它从矢量编辑器发展为包含矢量、栅格、动画和程序化设计的图形工具，并明确标注当前为 alpha。

它适合用来探索：

- 可重复生成的几何纹理、点阵和路径变体；
- 需要保留节点关系、方便批量调整的 Logo 系列；
- 具有生成艺术、程序化图形或动态视觉特征的产品标志。

**判断：** 值得关注和试验，但应把它定位为探索工具。正式交付前仍应导出干净的 SVG，并在 Inkscape、浏览器或目标平台中做最终检查。

### Method Draw：简单、易嵌入，但功能边界明确

Method Draw 的目标是提供简单易用的 Web SVG 编辑体验，README 明确说它为了简洁而移除了图层等能力，并建议需要更完整功能时使用 SVG-Edit。它适合快速画简单形状或做一个轻量在线编辑入口，不适合作为复杂 Logo 项目的唯一工具。

## 2. 资产导出和工程化流水线

### open-assets：最贴近 Git 和 Agent 的资产流水线

`open-assets` 把营销资产当作代码仓库中的 HTML/CSS/SVG 模板。`assets.json` 用 collection、template、export size 和 output action 描述资产；同一个源模板可以渲染多个尺寸，并写入 `exports/`、Web 应用目录或 Xcode `.appiconset`。

对 Logo 和产品图标最有用的能力包括：

- 用多个 SVG 模板保存 Logo 的浅色、深色、图标和 wordmark 变体；
- 用一个 App Icon 模板输出 1024、512、180 等尺寸；
- 用 `copy-source` 保留 SVG 源文件，避免交付物只剩 PNG；
- 用 `assets.lock` 做增量渲染，适合提交到 Git 和放进 CI；
- 支持本地实时预览、导出和本地化，适合把图标、商店截图、OG 图片放在一套资产工程里；
- 仓库内置 `open-assets` Agent Skill，可以让 Agent 根据 `assets.json` 创建和维护模板。

**推荐定位：** `Inkscape/Penpot` 负责设计，`open-assets` 负责版本化预览和批量导出。不要期待它代替节点编辑、字体排版或品牌评审。

### brand：已有标志后的“品牌资产编译器”

`brand` 的输入是一个 SVG 图标、品牌名称和 TOML 配置，输出 favicon、App Icon、Logo、wordmark 的深色/浅色版本，同时生成 `site.webmanifest` 和 HTML head 片段。README 还说明 PNG 会用 `oxipng` 优化，SVG 文本会用 `usvg` 转成路径，以减少目标环境缺少字体造成的差异。

它适合在 Logo 已经定稿后统一生产 Web 和 App 所需文件。它不负责探索图形方向，也不应被误解为 Logo 生成器。

### icon-generator：Apple 平台的专用打包器

`icon-generator` 使用 Swift 构建，支持单个 PNG/SVG、iOS/macOS/watchOS `.appiconset` 和 Xcode Icon Composer `.icon`，还支持 SF Symbols、渐变、中心内容和角标。它适合已经有明确图形或文字标志，需要快速生成 Apple 资产包的场景。

它的范围明显偏 Apple 平台：不要用它代替跨平台 Logo 设计，也不要把一个带圆角的预览图直接当成所有平台的原始图稿。应先保存无平台装饰的源标志，再按目标平台生成预览和打包文件。

## 3. Agent Skills 与 AI 辅助工作流

### `logo-designer-skill`：Logo 的结构化迭代流程

[neonwatty/logo-designer-skill](https://github.com/neonwatty/logo-designer-skill) 是 MIT 许可证的 Claude Code 插件，README 将流程拆为四阶段：

1. 访谈品牌、受众和审美偏好；
2. 生成 3 至 5 个可比较的 SVG 方向；
3. 选择方向后迭代颜色、布局和细节；
4. 用 `resvg`、Inkscape 或 `librsvg` 等工具输出 16 至 2048 像素的 PNG。

它比“给我生成一个 Logo”的单轮提示更适合工程项目，因为它把多方向探索、小尺寸检查和导出写入同一个工作流。需要注意的是，仓库文档明确以 Claude Code 插件方式安装，没有声明 Codex 的专属安装路径；在 Codex 中可以借鉴其 `SKILL.md` 思路，但要根据当前 Agent 的权限和协作规则调整。

### `open-assets` skill：把设计稿变成可审查的资产工程

`open-assets` 除了 CLI，还提供名为 `open-assets` 的 Agent Skill。它要求 Agent 读取 `assets.json`，按照 collection 和模板创建 Logo、图标、商店截图并运行渲染命令。它最适合和已经存在的 SVG/HTML 设计稿结合，而不是让 Agent 直接凭空完成品牌策略。

可采用的安装方式以仓库当前 README 为准，例如：

```bash
npx skills add https://github.com/Parra-Inc/open-assets --skill open-assets
```

### `app-icon-skill`：验证优先的 App Icon 流程，但许可证待确认

[xzhih/app-icon-skill](https://github.com/xzhih/app-icon-skill) 的 README 明确将自己描述为 Codex skill collection，包含 `app-icon-gen` 和 `app-icon-template-creator`。它强调：

- 保留源证据文件和可编辑的 SVG/PNG 图层；
- 将圆角预览图和无圆角 App/源文件分开；
- 用规范、报告、side-by-side、overlay、diff 和几何检查验证结果；
- 只有明确请求 Apple 目标时才生成 Icon Composer `.icon`；
- 生成失败时回到 spec 修复并重新生成，而不是手工修改生成物。

这套约束很适合真正的 App Icon 交付。不过截至本次核验，仓库根目录只有 README、package.json、skills 等文件，没有发现明确的根目录许可证声明。因此它可以作为流程参考或待审查候选，但在确认作者授权前，不应在商业项目中直接按“开源 skill”安装和分发。

### `inkpilot`：在 VS Code 内通过 MCP 操作 SVG

`inkpilot` 是 MIT 许可证的 VS Code 扩展，官网和 README 描述了实时 SVG 预览、14 个 MCP 工具、5 个内置设计提示（Logo、Icon、Badge、Banner、Graphic）以及 `.inkp` 项目文件。它的优点是 Agent 的每次修改可以立即在 SVG 编辑面板中查看，导出结果也是同一份 SVG。

它适合尝试“对话式编辑 SVG”，但目前仓库规模和社区信号都很早期。使用前应验证：MCP 客户端连接、Windows/VS Code 版本、字体渲染、路径导出和项目文件的可恢复性。不要因为有“AI SVG 编辑器”标签就跳过人工的几何和商标审查。

### `design-system-assets`：以品牌系统约束产品图标

[justnardo/design-system-assets](https://github.com/justnardo/design-system-assets) 是 MIT 许可证的 Claude Code skill。它先读取 `DESIGN.md`、Tailwind 配置或 CSS 变量，再路由资产类型：图标走 Lucide SVG，照片等位图资产走图像模型，并提供品牌一致性审查和成本估算脚本。

它不是 Logo 生成器，但适合产品已经有设计系统后，批量生成和检查页面图标、OG 图片、插画和产品展示图。它要求自带 API key；如果只要 UI 图标，直接使用 Lucide/Tabler 通常更简单，也更容易离线和可复现。

### `ui-ux-pro-max`：产品界面和图标系统的辅助 skill

[nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) 是 MIT 许可证项目，README 当前列出 79 个可搜索 UI 风格、192 个产品类型/配色规则、74 组字体搭配、119 条 UX 指南、25 类图表和 22 个技术栈，并提供 Codex CLI 的安装方式。

它适合帮助 Agent 先确定产品类型、视觉风格、颜色、字体、交互和可访问性，再设计一套一致的产品图标使用规则。需要注意其 README 明确区分开源基础版与高级版：品牌识别、Logo 设计、企业视觉项目和定制图标属于高级版描述的扩展范围。因此，开源版应被视为产品 UI 设计系统 skill，而不是专门的 Logo skill。

### Anthropic 的 `canvas-design` 与 `frontend-design`

Anthropic 的公开 [skills 仓库](https://github.com/anthropics/skills)中有 `canvas-design` 和 `frontend-design`。对应 skill 文件带有 Apache-2.0 的 `LICENSE.txt`：

- `canvas-design` 先建立视觉哲学，再输出 PNG 或 PDF，适合做品牌情绪板、海报式探索和展示图；它的输出契约不是 SVG，因此不能直接替代 Logo 主稿流程。
- `frontend-design` 强调有个性的视觉方向、字体、色彩、布局和自我审查，适合产品官网、品牌页面和 UI 表面；它不是几何 Logo 绘制工具。

## 4. 开源图标素材库

| 素材库 | 许可证 | 用途 | 注意事项 |
| --- | --- | --- | --- |
| [Lucide](https://github.com/lucide-icons/lucide) | ISC；其中继承自 Feather 的部分图标保留 MIT 说明 | 轻量、统一线性 UI 图标，支持多个前端框架和静态 SVG | 项目明确不接受品牌 Logo；不要把它当作品牌标志库 |
| [Tabler Icons](https://github.com/tabler/tabler-icons) | MIT | 以 24×24 网格和 2px 描边为基础的通用 UI 图标，提供线框和填充版本 | 若用于产品图标体系，应统一 stroke、圆角和尺寸，不要混用不兼容的图标风格 |
| [Phosphor](https://github.com/phosphor-icons/core) | MIT | 提供多种粗细和风格的通用图标，适合建立层级化的 UI 图标系统 | 需要在项目中固定权重、网格和命名规则 |
| [Simple Icons](https://github.com/simple-icons/simple-icons) | CC0-1.0（仓库图标数据） | 查找 GitHub、平台、技术品牌等官方品牌图标 | 开放许可证不等于取消商标限制；用于第三方品牌展示前仍要核对品牌规范 |
| [Iconify](https://github.com/iconify/iconify) | 框架代码 MIT；图标集各自遵循其许可证 | 用统一 API、组件和搜索方式接入 200+ 图标集 | Iconify 的框架许可证不覆盖所有图标本身，必须逐个查看图标集许可证 |

## 5. 推荐组合

### 组合 A：独立开发者制作一个产品 Logo

```text
品牌 brief
  -> logo-designer-skill 或人工发散 3-5 个方向
  -> Inkscape 清理几何、文字和路径
  -> 输出 SVG 主稿、单色稿、反白稿、wordmark
  -> brand 或 open-assets 生成 favicon/App Icon/PNG
  -> 在 16/24/32/64/1024 px 和浅色/深色背景检查
```

这是成本和控制力最平衡的方案。AI 负责加快方向探索，Inkscape 负责可编辑主稿，导出工具负责重复性工作。

### 组合 B：产品团队维护 Logo 与 UI 设计系统

```text
Penpot 自托管
  -> 建立 Logo 变体、颜色、字体、图标使用规则
  -> 与产品界面组件和设计令牌关联
  -> 导出/检查 SVG
  -> open-assets 在 Git 中生成多尺寸资产
```

适合多人评审、内网部署、需要长期维护设计资产的团队。Penpot 是设计协作中心，Git 中的 SVG 和 `assets.json` 是工程交付源。

### 组合 C：Codex/Agent 参与 App Icon 设计

```text
产品 brief 与已有品牌标志
  -> app-icon-skill（先确认许可证）或人工编写 icon spec
  -> 保留 origin、图层和验证报告
  -> open-assets 生成通用 Web/商店尺寸
  -> icon-generator 生成 Apple 专属包（确有 Apple 需求时）
  -> 人工检查圆角预览、无圆角源图和平台渲染差异
```

如果不能接受未确认许可证的 skill，就使用 `logo-designer-skill`/`inkpilot` 生成和编辑 SVG，再以 `open-assets` 和 `icon-generator` 负责确定性的导出。

### 组合 D：产品 UI 图标体系

```text
Lucide / Tabler / Phosphor 选定一个主库
  -> 固定 16/20/24 网格、stroke、圆角和填充规则
  -> 用 Iconify 辅助检索，不直接混合不同风格
  -> 只有确有独特语义时才制作自定义 SVG
  -> 将图标作为组件和设计令牌纳入代码库
```

通用 UI 图标和品牌 Logo 是两种不同的设计任务：前者重视识别速度、状态覆盖和一致性，后者重视独特性、商标风险和跨尺寸记忆点。

## 6. 选择建议

| 需求 | 首选 | 辅助 | 不建议直接承担的工具 |
| --- | --- | --- | --- |
| 专业 Logo 主稿 | Inkscape | Penpot、`logo-designer-skill` | 位图生成器、只支持简单形状的在线编辑器 |
| 团队协作和自托管 | Penpot | Inkscape、Git | 只保存 PNG 的生成器 |
| 浏览器内嵌 SVG 编辑 | SVG-Edit | `inkpilot` 作为 Agent 入口 | 直接把桌面工具远程拼接到产品中 |
| App Icon 多尺寸导出 | open-assets | `brand`、`icon-generator` | 手工逐个缩放 PNG |
| UI 线性/填充图标 | Tabler、Lucide、Phosphor | Iconify | 用图像模型逐个生成 UI 图标 |
| AI 视觉发散 | `logo-designer-skill`、`canvas-design`、本地 `imagegen` | `inkpilot` | 把首轮 AI 输出直接发布为商标或 Logo |
| 品牌一致的页面资产 | `design-system-assets`、`ui-ux-pro-max` | `frontend-design` | 没有设计系统就批量生成大量资产 |

## 7. 交付前检查清单

- **源文件：** 保留自包含 SVG，修整 `viewBox`，删除无用隐藏对象和外部字体依赖；最终 wordmark 根据交付需要把文字转为路径，同时保留未转曲的可编辑源稿。
- **变体：** 至少准备彩色、单色、反白、图标、wordmark 和横向组合；明确哪些是正式标志，哪些只是展示图。
- **尺寸：** 在 16、24、32、48、64、180、512、1024 等实际尺寸查看，不只在大画布中检查；favicon 和 App Icon 要单独测试细节是否消失。
- **背景：** 检查浅色、深色、纯黑、纯白和透明背景；不要只用一个渐变背景判断对比度。
- **平台：** 区分无平台装饰的源标志、圆角预览图和平台打包文件；Apple 的圆角预览不能直接替代跨平台源图。
- **字体：** 核对字体的商业授权、嵌入限制和转曲后的许可义务；工具的开源许可证不自动覆盖字体和第三方素材。
- **图标库：** 固定图标库、网格、描边粗细、圆角、填充和命名规则；Iconify 中的图标必须查看原始图标集许可证。
- **AI 与商标：** 对生成结果做相似 Logo 检索和人工审查，不把模型没有版权/商标保证误解为可以直接注册或商业使用。
- **可复现性：** 将源 SVG、配置、导出命令和工具版本放入 Git；不要只保存一次性导出的 PNG。

## 资料来源

### 绘制工具与资产工具

- [Inkscape FAQ](https://inkscape.org/learn/faq/)
- [Inkscape Features](https://inkscape.org/about/features/)
- [Penpot GitHub 仓库](https://github.com/penpot/penpot)
- [Penpot 官方网站](https://penpot.dev/)
- [SVG-Edit GitHub 仓库](https://github.com/SVG-Edit/svgedit)
- [Method Draw GitHub 仓库](https://github.com/methodofaction/Method-Draw)
- [Graphite GitHub 仓库](https://github.com/GraphiteEditor/Graphite)
- [Graphite 官方网站](https://graphite.art/)
- [open-assets GitHub 仓库](https://github.com/Parra-Inc/open-assets)
- [open-assets Agent Skill](https://github.com/Parra-Inc/open-assets/blob/main/skills/open-assets/SKILL.md)
- [brand GitHub 仓库](https://github.com/macalinao/brand)
- [icon-generator GitHub 仓库](https://github.com/schwa/icon-generator)
- [inkpilot GitHub 仓库](https://github.com/jamesmichaelredmond/inkpilot)
- [inkpilot 官方网站](https://inkpilot.dev/)

### Agent Skills

- [logo-designer-skill](https://github.com/neonwatty/logo-designer-skill)
- [logo-designer-skill 使用说明](https://neonwatty.com/posts/logo-designer-skill-claude-code/)
- [app-icon-skill](https://github.com/xzhih/app-icon-skill)
- [design-system-assets](https://github.com/justnardo/design-system-assets)
- [ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)
- [Anthropic Agent Skills 仓库](https://github.com/anthropics/skills)
- [Anthropic canvas-design skill](https://github.com/anthropics/skills/blob/main/skills/canvas-design/SKILL.md)
- [Anthropic frontend-design skill](https://github.com/anthropics/skills/blob/main/skills/frontend-design/SKILL.md)

### 图标素材库

- [Lucide](https://github.com/lucide-icons/lucide)
- [Tabler Icons](https://github.com/tabler/tabler-icons)
- [Phosphor Icons Core](https://github.com/phosphor-icons/core)
- [Simple Icons](https://github.com/simple-icons/simple-icons)
- [Iconify](https://github.com/iconify/iconify)
