---
title: "mattpocock/skills 调研"
date: 2026-08-01
category: "Agent工程"
tags:
  - Agent技能
  - AI编程
  - 工程工作流
  - Codex
  - Claude Code
---

# mattpocock/skills 调研

> 调研日期：2026-08-01。本文只引用仓库作者维护的 GitHub 仓库、源码与 GitHub Releases 等一手资料。

## 定位

[`mattpocock/skills`](https://github.com/mattpocock/skills) 是 Matt Pocock 日常使用的 AI 编程代理技能集合，仓库描述为 “Skills for Real Engineers. Straight from my .agents directory.”。其目标不是接管整个开发流程，而是提供**小、可修改、可组合**的工程实践；README 明确称其适用于任意模型，并面向 Claude Code、Codex 等代理的常见失败模式（需求未对齐、反馈不足、代码库劣化）。[来源：仓库简介](https://github.com/mattpocock/skills)；[README：定位与理念](https://github.com/mattpocock/skills/blob/main/README.md#skills-for-real-engineers)

从源码结构看，技能以 `SKILL.md` 为单元，主要分入 `engineering` 与 `productivity`；原生 Claude 插件仅显式发布这两个目录中的 22 项技能，不包含 `deprecated`、`in-progress`、`misc`、`personal` 等目录。[来源：技能目录](https://github.com/mattpocock/skills/tree/main/skills)；[插件清单](https://github.com/mattpocock/skills/blob/main/.claude-plugin/plugin.json)；[分发决策 ADR](https://github.com/mattpocock/skills/blob/main/.agents/adr/0002-ship-as-a-claude-code-plugin.md)

## 技能类型与清单

该仓库按“谁能调用”划分：**用户调用型**只在用户显式输入技能名时运行，主要承担流程编排；**模型调用型**既可由用户调用，也可在任务匹配时由代理自动选用，承载可复用的工程纪律。用户调用型不能调用另一项用户调用型技能，但可以调用模型调用型技能。[来源：README 的 Invocation 说明](https://github.com/mattpocock/skills/blob/main/README.md#reference)

### 用户调用型（12 项）

| 分组 | 技能 | 用途摘要 |
|---|---|---|
| Engineering | `ask-matt` | 为当前场景路由到合适技能/流程。 |
| Engineering | `grill-with-docs` | 深度访谈以澄清方案，同时维护术语、`CONTEXT.md` 与 ADR。 |
| Engineering | `triage` | 按状态机分流 Issue 和外部 PR，形成可供代理执行的简报。 |
| Engineering | `improve-codebase-architecture` | 扫描可加深（deepening）的架构机会，生成可视化报告并访谈选择。 |
| Engineering | `setup-matt-pocock-skills` | 为仓库设置 issue tracker、分诊标签及领域文档布局。 |
| Engineering | `to-spec` | 将已有对话整理为规范并发布到已配置的问题跟踪器。 |
| Engineering | `to-tickets` | 将计划/规范拆成带阻塞关系的 tracer-bullet 工单。 |
| Engineering | `wayfinder` | 为跨多个代理会话的大型工作构建“决策地图”。 |
| Engineering | `implement` | 按规范或工单实施，并在指定接缝使用 TDD、收尾做代码审查。 |
| Productivity | `grill-me` | 对计划或设计进行深度追问。 |
| Productivity | `handoff` | 将当前对话压缩为供另一代理接手的文档。 |
| Productivity | `teach` | 在当前工作区内教授新概念或技能。 |

### 模型调用型（10 项）

| 分组 | 技能 | 用途摘要 |
|---|---|---|
| Engineering | `prototype` | 用一次性原型回答状态/逻辑或 UI 设计问题。 |
| Engineering | `diagnosing-bugs` | 对疑难 Bug 或性能回归执行“复现→最小化→假设→埋点→修复→回归测试”循环。 |
| Engineering | `research` | 后台代理基于官方文档、源码、规范与第一方 API 调研，并产出带引用的 Markdown。 |
| Engineering | `tdd` | 以测试优先的 red-green 循环构建功能或修复问题。 |
| Engineering | `domain-modeling` | 建立和打磨领域模型、术语表与 ADR。 |
| Engineering | `codebase-design` | 提供深模块、接口、接缝等设计词汇和原则。 |
| Engineering | `code-review` | 从仓库规范和原始需求两条轴并行审查改动。 |
| Engineering | `resolving-merge-conflicts` | 处理进行中的 Git merge/rebase 冲突。 |
| Productivity | `grilling` | 可复用的深度访谈原语。 |
| Productivity | `writing-great-skills` | 编写、修改高质量技能的原则与术语参考。 |

以上清单以插件 manifest 的实际发布目录为准；每项技能的名称、描述和是否禁用模型自动调用均可在相应 [`SKILL.md`](https://github.com/mattpocock/skills/tree/main/skills) 中核验。[来源：README 技能参考](https://github.com/mattpocock/skills/blob/main/README.md#reference)；[插件 manifest](https://github.com/mattpocock/skills/blob/main/.claude-plugin/plugin.json)

## 安装与使用

### Claude Code：托管的原生插件

```bash
claude plugins install mattpocock-skills
```

也可在会话中输入：

```text
/plugin install mattpocock-skills
```

README 说明：该插件在 Claude Code 官方 marketplace 中；这一路径安装的是只读、受托管的整套技能，并会随作者发布自动更新。[来源：README 安装说明](https://github.com/mattpocock/skills/blob/main/README.md#installation-30-second-setup)

### Codex 与其他代理：skills.sh 安装器

```bash
npx skills@latest add mattpocock/skills
```

安装器会让用户选择要安装的技能及目标编码代理。README 特别要求勾选 `setup-matt-pocock-skills`；随后在**每个仓库**执行一次：

```text
/setup-matt-pocock-skills
```

它会交互式配置问题跟踪器（GitHub、Linear 或本地文件）、`triage` 使用的标签以及文档保存位置。以 `skills.sh` 方式安装时，技能文件复制到项目中，可自行编辑；更新使用：

```bash
npx skills update
```

不要同时安装 Claude 托管插件和 `skills.sh` 副本，否则同一技能会出现两份。[来源：README 安装与初始化](https://github.com/mattpocock/skills/blob/main/README.md#installation-30-second-setup)；[setup 技能源码](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/SKILL.md)

## 平台与运行时要求

- **Claude Code**：可使用其官方 marketplace 的原生插件；该插件显式列举 22 个已发布技能目录。[来源：插件 manifest](https://github.com/mattpocock/skills/blob/main/.claude-plugin/plugin.json)
- **Codex/其他代理**：仓库当前推荐 `npx skills@latest add` 这一跨代理安装方式。作者的 ADR 说明，原生 Codex 插件尚未提供：Codex manifest 当时只能写一个技能根目录，无法只选中两个已发布目录而排除草稿、废弃与个人技能；因此暂缓原生插件而保留 `skills.sh` 路径。[来源：README](https://github.com/mattpocock/skills/blob/main/README.md#installation-30-second-setup)；[ADR](https://github.com/mattpocock/skills/blob/main/.agents/adr/0002-ship-as-a-claude-code-plugin.md)
- **Node/npm**：上面的 `npx` 安装命令要求本机具备可运行 `npx` 的 Node/npm 环境。仓库 `package.json` 标注开发包管理器为 `npm@10.9.4`，但没有在 README 声明最低 Node 版本，因此不应据此推断最低版本要求。[来源：package.json](https://github.com/mattpocock/skills/blob/main/package.json)
- **问题跟踪器（可选且按配置）**：setup 技能原生支持 GitHub（`gh` CLI）、GitLab（`glab` CLI）或仓库内本地 Markdown；也可以记录 Linear/Jira 等自定义工作流。因此只有选择相应远程跟踪器时才需要对应 CLI/访问权限。[来源：setup 技能源码](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/SKILL.md)

## 许可证与发布状态

- 许可证为 **MIT**。[来源：LICENSE](https://github.com/mattpocock/skills/blob/main/LICENSE)
- GitHub 标记的最新正式 Release 是 **`v1.1.0`**，发布时间为 **2026-07-08 13:20:57 UTC**。该版本新增/推广了 `research`、`code-review`、`wayfinder` 等能力，并统一了 `to-spec`/`to-tickets` 规划流程。[来源：v1.1.0 Release](https://github.com/mattpocock/skills/releases/tag/v1.1.0)
- 截至调研时，默认分支源码中存在版本字段不完全一致的情况：`package.json` 为 `1.1.0`，`.claude-plugin/plugin.json` 为 `1.2.0`；这不是一个额外的 GitHub Release，实际“最近发布版本”应仍以 Releases 页面标记的 `v1.1.0` 为准。[来源：package.json](https://github.com/mattpocock/skills/blob/main/package.json)；[插件 manifest](https://github.com/mattpocock/skills/blob/main/.claude-plugin/plugin.json)；[Releases](https://github.com/mattpocock/skills/releases)

## 适用流程示例

一个典型的“大需求”链路是：先用 `/grill-with-docs` 校准需求和领域术语；用 `/to-spec` 固化规范；用 `/to-tickets` 建立带依赖的工单；再由 `/implement` 实施，期间按需让模型调用 `/tdd`、`/prototype`、`/research` 与 `/code-review`。这是一种推荐组合，不表示技能会无条件自动串行执行；README 强调它们应保持小而可组合。[来源：README 的理念及技能参考](https://github.com/mattpocock/skills/blob/main/README.md)
