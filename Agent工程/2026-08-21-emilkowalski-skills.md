---
title: "emilkowalski/skills 调研"
date: 2026-08-21
category: "Agent工程"
tags:
  - Agent技能
  - AI编程
  - UI设计
  - 动画
  - Expo
  - Codex
---

# emilkowalski/skills 调研

> 调研日期：2026-08-21。本文以仓库 README、技能源码、GitHub API 元数据、提交记录和已合并 PR 为主要依据；GitHub 的 star、fork、更新时间等动态数据均为本次调研时的快照。

## 结论先行

[`emilkowalski/skills`](https://github.com/emilkowalski/skills) 是 Emil Kowalski 面向设计师和工程师维护的 Agent Skills 集合，重点解决 AI 在 UI 设计、动画取舍、组件选择和交互细节上的“能运行但没有品味”问题。它不是一个运行时库，也没有应用代码和构建系统；仓库主体是带 YAML frontmatter 的 `SKILL.md`，通过 [`skills.sh`](https://skills.sh/) 的安装器分发给编程代理。

它最有价值的部分是把经验固化成可执行约束：先判断是否应该动画，再确定目的、工具、属性、曲线、时长、可中断性和无障碍降级；对于审查和全库改进，又要求输出带证据的表格或可独立执行的计划。适合把它作为 UI/动效工作的第二层质量门，而不应当把每一条规则当成脱离产品语境的绝对规范。

## 仓库定位与状态

- **作者与定位**：仓库描述为 “Skills for Designers and Engineers.”，README 说明这些技能来自作者在 Vercel、Linear 等公司的经验，目标是帮助代理更快做出更好的界面。[来源：仓库简介](https://github.com/emilkowalski/skills)；[README：定位](https://github.com/emilkowalski/skills/blob/main/README.md)
- **数据快照**：截至 2026-08-21，GitHub API 返回约 31,235 stars、1,747 forks，主分支最近更新时间为 2026-08-21 08:53 UTC；仓库创建于 2026-03-16，非 fork，未归档。[来源：GitHub Repository API](https://api.github.com/repos/emilkowalski/skills)
- **当前提交**：主分支 HEAD 为 `e879241fab3cdb22e8d95587cdbf40b57a88d7da`，提交日期为 2026-08-18，主题是加入 Expo 技能。[来源：提交记录](https://github.com/emilkowalski/skills/commit/e879241fab3cdb22e8d95587cdbf40b57a88d7da)
- **许可证与技术形态**：MIT License；GitHub 识别的主要语言是 Markdown，仓库没有 `package.json`、应用源码或运行时依赖。[来源：仓库 API](https://api.github.com/repos/emilkowalski/skills)；[文件树](https://github.com/emilkowalski/skills/tree/main)
- **维护信号**：2026-07 至 2026-08 持续加入动画发现、原型、Sonner 和 Expo 内容；Expo PR 合并前处理了 Reanimated 4、`scheduleOnRN`、Expo 安装命令、React Compiler 兼容和原生导航能力等审阅意见。[来源：PR #25](https://github.com/emilkowalski/skills/pull/25)；[最新提交说明](https://github.com/emilkowalski/skills/commit/e879241fab3cdb22e8d95587cdbf40b57a88d7da)

## 技能组成

当前主分支 `skills/` 下有 11 个技能目录。README 将它们按“设计工程基础、动画构建与审查、设计原则、工具选择和具体产品指南”组织，而不是按技术栈拆成插件。

| 技能 | 角色 | 核心职责 |
|---|---|---|
| `emil-design-eng` | 基础原则 | UI 细节、动效决策、组件交互、性能和无障碍的总原则；还规定 UI 审查使用 `Before/After/Why` Markdown 表格。 |
| `animate` | Web 构建 | 从“是否该动”开始，依次确定目的、最低成本工具、动画属性、曲线/时长、打断和 reduced motion，然后直接编写实现。 |
| `animate-expo` | React Native/Expo 构建 | 将同一套判断迁移到移动端，强调 Reanimated UI runtime、Gesture Handler、Expo Router 原生能力、haptics 和真机验证。 |
| `review-animations` | 单次审查 | 审查现有动效或 diff；按十条不可妥协标准输出单一 findings 表和明确的 Block/Approve 结论。 |
| `improve-animations` | 全库审计与规划 | 只读扫描整个代码库，按八类问题排序，并生成包含路径、现状代码、目标值、边界和验证步骤的实施计划；不直接改源代码。 |
| `find-animation-opportunities` | 机会发现 | 查找本来应该有动效的交互缝隙，同时强制列出被拒绝的候选，避免把“更活泼”变成全站加动画。 |
| `animation-vocabulary` | 术语反查 | 将“从按钮长出来”“像 iOS 橡皮筋”等模糊描述映射到 Pop in、Rubber-banding、Stagger 等可用于继续提示代理的术语。 |
| `apple-design` | 交互原则 | 将 Apple 的流体界面原则翻译为 Web 实践，覆盖 1:1 手势、速度交接、动量投射、可中断弹簧、橡皮筋边界和 reduced motion。 |
| `pick-ui-library` | 依赖选择 | 从一份有明确偏好的清单中为 toast、popover、command menu、图表、虚拟列表、状态和样式等任务选一个库，并要求先检查项目已有依赖。 |
| `prototype` | 方案探索 | 在隔离路由或独立 HTML 中构建最多 3 个默认的真实变体，用 picker 即时切换；选定后才提升到生产代码并删除原型面。 |
| `ask-sonner` | Sonner 指南 | 覆盖 `Toaster` 挂载、`toast()` 调用、promise/loading、更新/关闭、多 Toaster、主题、样式和常见故障排查。 |

仓库还为部分技能提供配套文档：Web 与 Expo 动画有 `RECIPES.md`，全库审计有 `AUDIT.md` 和 `PLAN-TEMPLATE.md`，动画审查有 `STANDARDS.md`，原型有 `PICKER.md`，Sonner 有 `API.md`。[来源：仓库文件树](https://github.com/emilkowalski/skills/tree/main/skills)

## 关键方法论

### 1. 先过频率和目的门槛

动画不是默认增强项。技能将交互按出现频率分层：每天 100 次以上的快捷键、命令面板等不应动画；每天几十次的 hover、列表导航只能极轻微；偶发的 modal、drawer、toast 才使用标准动效；首次引导、成功和庆祝场景才有更大的 delight 预算。

继续实施前还必须给出明确目的：反馈、空间一致性、状态指示、避免突兀变化、解释或罕见场景的愉悦。如果只能说“看起来酷”，就应拒绝实现。这个门槛是仓库区别于普通动效素材库的核心。

### 2. 把“感觉”压缩成具体参数

Web 技能反复使用以下基线：进入/退出用 `ease-out`，屏幕内移动或形变用 `ease-in-out`，UI 尽量低于 300ms；按钮按下约 100-160ms，小 popover 约 125-200ms，下拉框约 150-250ms，modal/drawer 约 200-500ms。推荐的自定义曲线包括：

```css
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
```

实现层面的固定规则包括：避免 `scale(0)`，使用 `scale(0.95)` 加透明度；popover 从触发源而不是中心缩放；优先 `transform` 和 `opacity`；频繁可反转的交互使用 CSS transition，手势和带动量的交互使用 spring；hover 动效要放在 `@media (hover: hover) and (pointer: fine)` 内；`prefers-reduced-motion` 应降级为更温和的反馈而不是简单全部删除。

### 3. 用不同技能覆盖不同生命周期

仓库把“写、查、改进、探索”分得很清楚：

1. 新增 Web 动效使用 `animate`，Expo/React Native 使用 `animate-expo`。
2. 已有单个动效或一次 diff 使用 `review-animations`；它要求 findings 表，发现明显问题时可以直接 Block。
3. 想系统性改善整个代码库使用 `improve-animations`。它先做 recon，再按 purpose/frequency、easing/duration、physicality/origin、interruptibility、performance、accessibility、cohesion、missed opportunities 八类审计，最后写计划；源代码保持只读。
4. 想找“哪里值得加动效”使用 `find-animation-opportunities`，其输出必须包含 2-5 个被拒候选及拒绝原因。
5. 想比较多种 UI 方向使用 `prototype`，变体必须在布局、密度、个性、动效或交互模型上真正分叉，不能只是换色。

这种分工使审查结果可以被另一个代理执行，也使“发现机会”和“修复已有动效”不会混成一个无边界任务。

### 4. 移动端把线程和原生能力纳入设计

`animate-expo` 不是简单地把 Web CSS 规则翻译成 React Native。它明确要求：

- 手势连续值放在 Reanimated UI runtime，避免每帧 `setState`、`PanResponder` 或 JS 线程驱动动画。
- 触摸相关动作优先使用 shared value、Gesture 和 `useAnimatedStyle`；普通 press scale 使用更便宜的 Reanimated CSS transition。
- 屏幕转场、底部 sheet、原生 tabs、context menu、large title、键盘跟随和 pull-to-refresh 优先使用 Expo Router 或平台原生能力，不手写一个 JS 仿制品。
- 依赖安装使用 `npx expo install`，让 Expo 按 SDK 解析版本；技能还要求在 release build、目标支持的最慢设备上验证，而不是只在模拟器或开发模式旗舰设备上判断流畅度。

## 安装与使用

README 给出的跨代理安装入口是：

```bash
npx skills@latest add emilkowalski/skills
```

安装器的具体目标代理和选项由 `skills` CLI 决定；仓库本身没有 Claude/Codex 专属运行时代码。安装后可在支持 Skills 的编码代理中按技能名称或自然语言任务调用；本仓库 README 没有单独规定更新命令，更新方式应以当前 `skills` CLI 文档为准。

frontmatter 中明确标记 `disable-model-invocation: true` 的技能包括 `review-animations`、`pick-ui-library` 和 `prototype`，它们应当由用户显式调用；其余技能没有设置该标记，是否自动选用仍取决于宿主代理的 Skills 机制和上下文匹配。实际使用时建议显式指定边界，例如“只审查动画，不改代码”或“为这个 Expo 手势写实现并说明真机验证点”。[来源：各技能 `SKILL.md` frontmatter](https://github.com/emilkowalski/skills/tree/main/skills)

## 推荐工作流

| 场景 | 推荐组合 | 交付物 |
|---|---|---|
| 新建 Web 交互 | `animate`，必要时参考 `apple-design` | 直接实现的代码、频率/目的判断、曲线和验证点 |
| 新建 Expo 手势或 sheet | `animate-expo` + `apple-design` | UI runtime 上的实现、原生能力选择、真机 feel-check |
| 审查一个 PR 或组件 | `review-animations` | `Before/After/Why` 表格和 Block/Approve verdict |
| 全库动效治理 | `improve-animations` | 按优先级排序的 findings 和独立计划文件；随后交给执行代理 |
| 发现缺少的反馈或空间关系 | `find-animation-opportunities` | 少量高置信建议，以及明确拒绝的候选 |
| 组件或依赖选型 | `pick-ui-library` | 一个有理由的库选择；先尊重现有依赖 |
| Toast 接入或故障 | `ask-sonner` | Toaster 放置、调用方式、样式/主题和问题定位 |
| 设计方向不确定 | `prototype` | 隔离的多变体 picker，用户选择后再合并 |

## 优势与局限

### 优势

- **可执行性强**：不仅讲原则，还给出曲线、时长、属性、线程、输出表格和验证动作，适合直接作为代理约束。
- **克制而非堆动效**：频率门槛、目的门槛和 rejected candidates 能压制代理“哪里都加动画”的倾向。
- **覆盖设计到落地的闭环**：从术语、原则、构建、审查、全库规划到原型探索，职责边界清晰。
- **对移动端有真实工程意识**：线程模型、Expo SDK 依赖解析、原生导航/手势能力和最慢设备验证都被纳入。
- **细节知识密度高**：例如 popover 的 `transform-origin`、toast 的可中断性、拖拽速度交接、reduced motion 和 Sonner 的多 Toaster 问题，都是代理容易遗漏但用户能感知的细节。

### 局限与风险

- **规则高度观点化**：例如“快捷键不动画”“tabs 不滑动”“只动画 transform/opacity”“不要使用内置 easing”适合作为高质量默认值，但复杂产品、品牌动效、可视化和平台惯例可能需要例外；应以现有设计系统、可用性测试和产品频率覆盖这些硬规则。
- **主要是提示词/文档质量，不是自动化检查器**：仓库没有应用构建、运行时测试或静态规则执行器。`review-animations` 与 `improve-animations` 的结论仍需要代理逐行核实，并且“感觉是否正确”必须通过浏览器慢放、逐帧和真机体验确认。
- **选型清单带有作者偏好**：`pick-ui-library` 推荐 Base UI、Sonner、Motion、Virtuoso 等，优点是减少代理随意选库，代价是可能与团队已采用的竞品或长期技术路线冲突。技能本身要求先看 `package.json`，团队还应增加自己的依赖白名单和兼容性约束。
- **版本信息会随上游技术变化**：Expo、Reanimated、浏览器 CSS 能力和 Sonner API 都可能变化；虽然最新 Expo PR 已处理一批版本问题，但使用前仍应按项目 SDK、浏览器支持矩阵和当前官方文档复核。
- **全库审计可能产生较大认知成本**：`improve-animations` 会扫描很多交互并写计划，适合有明确动效治理目标的项目；小项目直接使用 `animate` 或 `review-animations` 更经济。

## 对当前知识库的判断

它与已有的 [`mattpocock/skills`](./2026-08-01-mattpocock-skills.md) 不是同类替代品：后者更偏需求澄清、规格、工单、TDD、研究和工程协作流程；`emilkowalski/skills` 更偏 UI 设计工程、动效质量和前端依赖选择。两者可以并存，前者负责“把事情定义清楚并交付”，后者负责“把界面做得有感知质量”。

如果在 Codex 或其他支持 `skills.sh` 的代理中使用，建议先安装后只启用与任务对应的技能；对生产代码先让 `review-animations` 或 `improve-animations` 观察，再让普通执行流程根据计划修改。不要把整套动效规则无条件写进项目 `AGENTS.md`，否则可能覆盖已有设计系统或让不该动的高频数据界面变得迟钝。

## 主要来源

- [仓库 README](https://github.com/emilkowalski/skills/blob/main/README.md)
- [仓库文件树](https://github.com/emilkowalski/skills/tree/main/skills)
- [仓库 GitHub API 元数据](https://api.github.com/repos/emilkowalski/skills)
- [主分支最新提交](https://github.com/emilkowalski/skills/commit/e879241fab3cdb22e8d95587cdbf40b57a88d7da)
- [Expo 技能合并 PR #25](https://github.com/emilkowalski/skills/pull/25)
- [emil-design-eng](https://github.com/emilkowalski/skills/blob/main/skills/emil-design-eng/SKILL.md)
- [animate](https://github.com/emilkowalski/skills/blob/main/skills/animate/SKILL.md)
- [animate-expo](https://github.com/emilkowalski/skills/blob/main/skills/animate-expo/SKILL.md)
- [review-animations](https://github.com/emilkowalski/skills/blob/main/skills/review-animations/SKILL.md)
- [improve-animations](https://github.com/emilkowalski/skills/blob/main/skills/improve-animations/SKILL.md)
- [find-animation-opportunities](https://github.com/emilkowalski/skills/blob/main/skills/find-animation-opportunities/SKILL.md)
- [apple-design](https://github.com/emilkowalski/skills/blob/main/skills/apple-design/SKILL.md)
- [pick-ui-library](https://github.com/emilkowalski/skills/blob/main/skills/pick-ui-library/SKILL.md)
- [prototype](https://github.com/emilkowalski/skills/blob/main/skills/prototype/SKILL.md)
- [ask-sonner](https://github.com/emilkowalski/skills/blob/main/skills/ask-sonner/SKILL.md)
