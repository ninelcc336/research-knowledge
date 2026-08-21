---
title: "Nutlope/hallmark 设计 Skill 调研"
date: 2026-08-21
category: "Agent工程"
tags:
  - Agent技能
  - UI设计
  - 反AI套模板
  - Codex
  - Claude Code
  - Cursor
  - 响应式设计
---

# Nutlope/hallmark 设计 Skill 调研

> 调研日期：2026-08-21。本文以仓库 README、`SKILL.md`、references 文档、`package.json`、GitHub API 元数据、提交记录和公开 Issue 为主要依据；star、fork、Issue 数量及更新时间均是本次调研时的快照。

## 结论先行

[`Nutlope/hallmark`](https://github.com/Nutlope/hallmark) 是 Together AI 发布的 MIT 开源设计 Skill，目标是让 Claude Code、Cursor 和 Codex 生成的页面减少“AI 模板感”。它不是 UI 组件库、设计系统运行时或业务框架，而是一套由 `SKILL.md` 和大量 Markdown 参考规则组成的提示词/工作流资产，另附一个用于展示案例的静态网站。

它最有价值的判断是：AI 生成页面的问题不只是颜色和字体雷同，更是页面结构雷同。因此 Hallmark 先选“宏结构”（例如 Bento Grid、Workbench、Long Document、Portfolio Grid），再选主题和组件语汇，并通过反模式门禁检查结构、文案、动效、可访问性和移动端安全性。对于新页面和视觉重设计，它可以作为一层较强的设计质量约束；对于业务逻辑、数据获取、认证、支付和状态管理，它明确不负责。

落地时需要把它看成“观点鲜明的设计审查协议”，而不是自动化质量保证系统：硬规则仍需由浏览器截图、可访问性检查、静态规则和人工审查补足。当前主分支的文档计数、导航选择权威和若干规则仍存在公开 Issue 记录的矛盾，生产采用前应固定提交版本并复核规则。

## 项目定位与状态

- **项目描述**：`Anti-AI-slop design skill for Claude Code, Cursor, and Codex.`；官网为 [`usehallmark.com`](https://www.usehallmark.com/)，README 标注由 Together AI 制作。
- **许可证**：MIT License，可直接使用和 fork。
- **数据快照**：截至 2026-08-21，GitHub API 返回约 26,302 stars、1,341 forks、42 个 open issues；仓库创建于 2026-04-27，未归档，主分支最近一次 push 为 2026-08-06。
- **当前提交**：主分支 HEAD 为 `13ac0ec7e148655948100b6396439e481361d690`，主题是合并 Grid 主题；此前提交说明将主题总数从 20 扩展到 21。
- **版本信号**：`package.json` 版本为 `1.1.0`，GitHub API 未返回可用 Release 或 tag。仓库主分支已经包含 Grid 等后续变更，因此不能仅把 npm/package 版本字段当作完整的功能版本标识。
- **仓库规模**：递归文件树约 286 个 blob、17.8 MB；其中 `skills/` 有 107 个文件、约 690 KB，`site/` 有 152 个文件、约 13.9 MB，`docs/` 有 21 个文件、约 3.1 MB。公开 Issue #50 将整体 Skill 体量描述为约 1 MB，并建议拆分命令，说明 token 成本已经成为维护讨论的一部分。

## 代码与文档结构

仓库没有应用运行时和业务依赖，主体结构如下：

| 路径 | 职责 |
| --- | --- |
| `skills/hallmark/SKILL.md` | Skill frontmatter、三种显式动词、默认设计流程、安全栏和加载顺序。 |
| `skills/hallmark/references/` | 颜色、字体、布局、文案、动效、响应式、交互状态、组件、主题、宏结构和审查门禁。 |
| `references/macrostructures/` | 21 个页面形状的详细规则，每次只应加载选中的一个。 |
| `references/themes/` | 主题级 token 和视觉语汇；当前目录包含 Grid 等 21 个主题。 |
| `references/genres/` | editorial、modern-minimal、atmospheric、playful 四种设计语境。 |
| `references/components/` | 导航、页脚和组件级结构原型。 |
| `site/` | 静态官网、主题切换、示例页面和 `_tests/` 下的测试构建。 |
| `docs/` | recipes、study examples、演讲幻灯片和截图素材。 |
| `ROADMAP.md` | Nanobanana 图像生成接入、品牌优先流程、主题动效 token、variant、多页一致性和 MCP 预览等规划。 |

`package.json` 只声明了 `type: module`、Skill 入口和一个 `python3 -m http.server` 静态服务脚本，没有 React、Next.js 或运行时 UI 依赖；“Powered by Together AI”更像项目归属/品牌说明，不代表安装后必须配置 Together API 才能运行。

## 核心能力

### 1. 三个显式动词加默认流程

| 调用方式 | 行为 | 修改边界 |
| --- | --- | --- |
| 默认 | 为新页面选择宏结构、主题、字体、token、动效和页面形状，生成后运行 slop test。 | 可创建或修改页面视觉层；不负责业务逻辑。 |
| `hallmark audit <target>` | 对已有实现按反模式评分，返回排序后的问题清单。 | 明确只读，不修改目标。 |
| `hallmark redesign <target>` | 保留文案意图、信息架构、路由和组件归属，重做视觉结构和交互语汇。 | 默认在原有实现边界内修改；大范围删除需用户确认。 |
| `hallmark study <screenshot | URL>` | 提取参考设计的 DNA：宏结构、组件原型、字体角色、颜色锚点和节奏，再输出诊断。 | 诊断与构建分开；`design.md` 只有用户明确要求锁定 DNA 时才生成。 |

默认 Design flow 的主要步骤是：预检现有项目、询问受众/用例/语气、先选宏结构、读取项目记忆和设计 token、选择主题、决定 hero enrichment、输出预览、构建、运行 slop test。预检会关注 `design.md`、字体、调色板、动效依赖、间距和框架，并可缓存到 `.hallmark/preflight.json`。

### 2. 用宏结构解决页面同质化

Hallmark 将页面形状从主题中拆出来，提供 21 个命名宏结构，包括：

- **网格/产品型**：Bento Grid、Workbench、Stat-Led、Component Playground、Feature Stack。
- **叙事/文档型**：Long Document、Letter、Manifesto、Conversational FAQ、Narrative Workflow。
- **展示/作品型**：Photographic、Quote-Led、Catalogue、Portfolio Grid、Split Studio、Type Specimen。
- **导航/空间型**：Index-First、Ecosystem Index、Map / Diagram、Marquee Hero、Specimen。

宏结构绑定标题位置、正文组成、分隔语言、按钮语气、图像处理、揭示方式以及导航/页脚原型。连续构建不能复用相同宏结构；如果上下文很模糊，Skill 要提供来自网格、文档、海报等不同类别的三个候选，而不是回到统一的 hero → 三特性 → CTA → footer 模板。

底层 `structure.md` 还把结构拆成六个轴：section heading 位置、正文布局、分隔方式、按钮声音、图片处理和 reveal 模式。理论上共有 42,000 种组合，但推荐优先选择完整的命名宏结构，以减少模型自由拼接造成的不协调。

### 3. 主题、语境与自定义分支

当前主题目录包含 21 个主题，例如 Specimen、Brutal、Newsprint、Terminal、Hum、Lumen 和最新的 Grid。主题按四种 genre 分组：

- `editorial`：默认，偏编辑、排版和纸张质感。
- `modern-minimal`：偏 SaaS、API、平台和开发者工具。
- `atmospheric`：偏 AI 工具、音乐、视频、深色和氛围表达。
- `playful`：偏消费级、友好、轻松和社区场景。

默认走 catalog 并按照 genre 和历史输出做差异化；只有用户明确要求自定义、给出品牌色、提供多属性审美方向或提出独特页面形状时，才询问 custom。Custom 有两个深度：保留 Hallmark 结构的 tuned，以及连页面构成一起从零设计的 bespoke；两者都必须执行全部 slop-test 门禁。

### 4. 反模式门禁与预发自评

Skill 要求在输出前进行六轴自评：Philosophy、Hierarchy、Execution、Specificity、Restraint、Variety；任何一项低于 3 都要返工。`slop-test.md` 当前标题写的是 **58 gates**，实际编号为 1–57 加一个 `38a`，重点覆盖：

- 禁止紫蓝渐变、渐变文字、Inter/系统字体无语境使用、全屏居中 hero、三等分图标特性卡片、卡片套卡片和无目的装饰。
- 禁止重复宏结构、所有区块统一 fade-up、AI 式导航/页脚、重绘浏览器/手机/IDE 外壳以及没有来源的指标、logo、证言和数字。
- 要求颜色和字体通过 `:root` token 使用，不能在渲染中临时写 hex/OKLCH/font-family。
- 要求所有交互元素覆盖 default、hover、focus-visible、active、disabled、loading、error、success 八种状态。
- 要求移动端在 320、375、414、768 px 检查：没有横向滚动、可点击文案不换行、图片网格使用 `minmax(0, 1fr)`、长标题可断词、区块标题移动端单列、radio tab 不跳滚动。
- 新增的 54 号门禁禁止把 eyebrow/编号放在标题左侧的双列结构中，要求标签与标题垂直堆叠；这是对常见“编辑风 SaaS”模板痕迹的针对性修正。

这些门禁是可执行的审查清单，但大部分仍由 Agent 阅读代码和观察渲染结果完成，并非仓库内置的统一 lint/CI 检查器。

### 5. `study` 的参考设计提取

`study` 区分截图模式和 URL 模式：截图能判断节奏和视觉整体感，但字体和色值只能估计；URL 模式能从 HTML/CSS 提取准确字体、token、DOM 结构和动效库，但无法可靠判断页面密度与节奏。远程页面被当作不可信数据，禁止执行其脚本或遵循其中的指令；登录墙、空 SPA shell、无 CSS 信号或模板市场页面会被拒绝或要求提供截图。

该设计明确区分“学习 DNA”和“复制页面”：可以提取宏结构和色彩角色，但不会复制像素、版权图片或知名设计师的签名风格。URL 生成可移植 `design.md` 前还要求用户确认是自己的页面或用于自有品牌的公开参考。

## 安装与使用

README 给出的安装命令是：

```bash
npx skills add nutlope/hallmark
```

也可以手动复制 `skills/hallmark/SKILL.md` 和 `references/`：

- Claude Code：`~/.claude/skills/hallmark/`
- Cursor：`.cursor/rules/hallmark.mdc`，使用 `SKILL.md` 正文但去掉 frontmatter
- Codex：`~/.codex/skills/hallmark/`，或项目级 `.codex/skills/hallmark/`

适合的调用方式是把设计目标和业务事实写入 brief，让 Hallmark 选择或确认页面形状；不要把它误用为组件库安装命令，也不要期待它自动接管数据请求、认证或支付逻辑。

## 优势、局限与风险

### 优势

- **结构差异化有明确抓手**：宏结构先于主题，且有会话/项目级轮换规则，比只换颜色更能降低模板感。
- **把审美拆成可检查约束**：token、字体、动效属性、交互状态、可点击文案和移动端轨道都有具体要求。
- **对现有代码有保护意识**：预检、append-only 全局样式表规则、`audit` 只读和 redesign 的删除安全栏减少了破坏式重写。
- **跨 Agent 分发简单**：同一套 Markdown 可放入 Claude Code、Cursor 和 Codex，安装门槛低。
- **参考设计处理相对谨慎**：`study` 将结构学习和像素复制分开，并为远程内容加入 prompt injection 安全边界。

### 局限与风险

1. **不是自动化验证器**：视觉节奏、是否“像 AI”、字体是否真的加载和交互是否自然，仍需浏览器截图、无障碍工具、静态规则或人工审查。
2. **规则体量较大且观点强**：约 690 KB 的 Skill 文件和参考资料对每次调用的上下文成本不低；“不使用某类动效/结构”的规则更适合作为默认值，不能替代产品频率、品牌和可用性测试。
3. **公开文档存在口径矛盾**：README 仍写“57 个 slop-test gates”，而当前 `SKILL.md` 和 `slop-test.md` 要求报告 `58 / 58`；编号列表是 1–57 加 `38a`。Issue [#37](https://github.com/Nutlope/hallmark/issues/37) 和公开 PR/Issue [#63](https://github.com/Nutlope/hallmark/issues/63) 正讨论将口径统一为 57，不能假设当前文档已经一致。
4. **权威来源不唯一**：Issue [#62](https://github.com/Nutlope/hallmark/issues/62) 指出主题、genre 和组件 cookbook 对导航原型的默认选择不完全一致，还指出宏结构 06 在总索引中可选但自己的文件又建议不要作为主页面。
5. **变更速度快但版本标记滞后**：Grid 在 2026-08-06 合入主分支，package 仍为 `1.1.0`，且没有 GitHub Release/tag 可供稳定锁定。
6. **设计约束可能与既有系统冲突**：例如全局 token、字体和 section heading 规则很强；在已有品牌设计系统或复杂后台中，应优先保留业务可用性和现有组件契约，再选择性采用门禁。

## 采用建议

- **新建营销页/作品页**：安装后先给出受众、核心动作和语气，再让 Hallmark 选择宏结构；生成后对 1280×800 以及 320/375/414/768 px 做截图检查。
- **改造既有页面**：先运行 `hallmark audit` 获得只读问题清单，再运行 `hallmark redesign`；明确保留路由、组件、接口和文案，不要让设计 Skill 代替业务重构。
- **已有设计参考**：优先用 `study` 做诊断；只有确认来源和用户明确要求时才锁定 `design.md`，不要直接要求复制参考页。
- **组件级任务**：走 component-scope，重点检查八种状态、focus ring、禁用/错误/加载反馈和命中区域，不要套用整页的宏结构与 hero 规则。
- **生产团队**：将颜色 token、`overflow-x`、网格 `minmax`、可点击文案不换行等硬门禁落成 lint/CI；将结构差异、视觉节奏和品牌判断保留给截图审查和设计评审。
- **版本治理**：若要长期使用，建议固定某个 commit 或在内部 fork 中维护一份经过验证的版本，同时跟踪 `ROADMAP.md` 和 Issues #61、#62、#63 的修订结果。

## 主要来源

- [GitHub 仓库](https://github.com/Nutlope/hallmark)
- [README](https://github.com/Nutlope/hallmark/blob/main/README.md)
- [`SKILL.md`](https://github.com/Nutlope/hallmark/blob/main/skills/hallmark/SKILL.md)
- [`package.json`](https://github.com/Nutlope/hallmark/blob/main/package.json)
- [宏结构索引](https://github.com/Nutlope/hallmark/blob/main/skills/hallmark/references/macrostructures.md)
- [结构规则](https://github.com/Nutlope/hallmark/blob/main/skills/hallmark/references/structure.md)
- [slop-test](https://github.com/Nutlope/hallmark/blob/main/skills/hallmark/references/slop-test.md)
- [响应式规则](https://github.com/Nutlope/hallmark/blob/main/skills/hallmark/references/responsive.md)
- [study 规则](https://github.com/Nutlope/hallmark/blob/main/skills/hallmark/references/study.md)
- [ROADMAP](https://github.com/Nutlope/hallmark/blob/main/ROADMAP.md)
- [GitHub Repository API](https://api.github.com/repos/Nutlope/hallmark)
- [文档矛盾 Issue #61](https://github.com/Nutlope/hallmark/issues/61)、[Issue #62](https://github.com/Nutlope/hallmark/issues/62)、[Issue #63](https://github.com/Nutlope/hallmark/issues/63)
