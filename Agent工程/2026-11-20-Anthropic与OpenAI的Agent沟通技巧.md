---
title: "Anthropic 与 OpenAI 的 Agent 沟通技巧调研"
date: 2026-11-20
category: "Agent工程"
tags:
  - AI编程
  - Claude Code
  - Codex
  - 提示词
  - 开发工作流
  - 任务验收
---

# Anthropic 与 OpenAI 的 Agent 沟通技巧调研

> 调研日期为 2026-11-20。收录 2026-09-20 至 2026-11-20 的公开材料，聚焦 Claude Code、Codex 和对应模型指南中的任务分派、推进与验收。没有独立署日的模型指南用同期发布公告及报道确认时间。X 的浏览量和点赞数来自 11 月 20 日的第三方公开镜像快照。X 站内搜索未返回可用结果，因此热度数据只描述有据可查的单帖。

## 最值得照做的三件事

1. **交代目标、投入和验收。** Claude Code 创建者 Boris Cherny 把和 Claude 沟通比作给同事分派任务。告诉它要做什么、值得花多少精力、如何确认做对。示例里他给了播客、可交互网站和视觉参考，把水彩风格留作可选项。这条 X 帖约 **153 万次浏览、1.39 万点赞**。 [Boris 原帖](https://x.com/bcherny/status/2107565388250874193) · [交付案例](https://x.com/bcherny/status/2107516876876362200) · [计数快照来源](https://api.fxtwitter.com/bcherny/status/2107565388250874193)
2. **让 Agent 知道可以自行决定什么。** OpenAI 的 10 月指南要求写清交付物、受众、上下文、限制和完成条件。常规取舍可以自行决定，范围变化或用户必须拍板的事项才暂停。GPT‑6.2 指南还建议明确授权 Agent 对可逆细节做合理假设，并持续完成运行、检查和修复。 [OpenAI GPT‑6 工作指南](https://openai.com/index/practical-guide-building-gpt-6/) · [GPT‑6.2 提示指南](https://developers.openai.com/api/docs/guides/latest-model/gpt-6.2-astra#task-completion)
3. **给它可以亲自跑的验收方法。** 一条能运行的测试命令、录制问题的视频、预期与实际的对照，都能让 Agent 在回报前自行修正。Claude Code 团队让 Agent 实际启动应用，从用户入口走完关键流程，并把重复使用的启动与检查步骤放进 `verify` Skill。 [Addy Osmani，Claude Code 团队如何验证](https://claude.dev/blog/how-the-claude-code-team-thinks-about-verification/)

## 按任务选沟通方式

### 需求还不清楚时，先让 Agent 找出要拍板的事

Thariq Shihipar 在 Claude Code 团队时公开分享的做法是给 Claude 一份初稿，让它访谈自己，把规格缺口问出来。Addy Osmani 也建议把访谈整理成规格，再开新会话按规格执行。问题集中在会改变结果的选择，例如交付的是原型还是生产功能、迁移允许多长的兼容期。Thariq 在访谈 15:37 和 21:52 附近还解释了短提示为何能奏效，以及补齐首轮上下文怎样减少返工。 [Thariq 的 effort 实验与工作流](https://claude.dev/blog/spending-your-effort/) · [Latent Space 访谈与逐字稿](https://latent.space/p/thariq/) · [Addy 的 X 文章](https://x.com/addyosmani/status/2115688494865465370)

```text
先读需求草稿，只问会改变交付物或实现范围的问题。问完后写一份可执行规格，包含目标、关键约束和验收示例。对能够从仓库确认的细节直接核实。
```

### 实现和验证可以投入不同的精力

Thariq 的个人流程是先补规格，以 `low` 实现和迭代，自己确认方向后再用 `high` 做验证和测试。他还建议让 Agent 留简短实现笔记，记录重要假设、关键取舍和需要人复核的决定。Anthropic 的 Opus 5.5 API 指南从 `medium` 起测，建议在自己的任务上比较质量、延迟和成本。两套做法分别适用于有人工看方向的分阶段操作和 API 起始配置。 [Thariq 原文](https://claude.dev/blog/spending-your-effort/) · [访谈约 25:56](https://latent.space/p/thariq/) · [Opus 5.5 提示指南](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#calibrate-effort)

```text
先实现规格里的主流程。完成后从真实入口跑一次，再检查空输入和失败重试。给我运行命令、实际结果，以及最多三项会影响后续维护的实现决定。
```

Claude Code 团队按故障类型选检查。用户流程用端到端测试。编码、归一化等操作可用性质测试检查往返一致性和幂等性。迁移可给新旧实现喂同一批样本，比对结果和已约定的差异。Addy 报告，团队在结构化文件生成的内部评估里，用独立评分器按验收规则检查，任务成功率最高提高 10 个百分点。这个幅度来自该项评估。 [团队验证文章](https://claude.dev/blog/how-the-claude-code-team-thinks-about-verification/)

### 长任务写成可检查的完成条件

Addy 的 10 月 29 日 X 文章建议把长任务的 `/goal` 写成执行结果能证明的条件，例如一组测试通过。该帖约 **37 万次浏览、2900 点赞**。Anthropic 的 Opus 5.5 指南还建议用清单追踪剩余项，进度消息之后按剩余工作继续，有后台命令就等到结果。自建 Agent 可以让独立模型在每轮后按完成条件检查对话，同时对反复卡在同一处的自动续跑设置上限。 [Addy 原帖](https://x.com/addyosmani/status/2115688494865465370) · [正文镜像](https://threadnavigator.com/thread/2115688494865465370/) · [计数快照来源](https://api.fxtwitter.com/addyosmani/status/2115688494865465370) · [Opus 5.5 长任务指南](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#unattended-agentic-runs)

```text
把 CSV 导出的三个入口迁到同一个实现。完成条件是三个入口的现有样本均能导出，中文列名和列顺序与基线一致，相关测试通过。只要还有可推进的步骤就继续，遇到样本基线缺失或必须更改导出格式时再问我。
```

### 中途纠偏时，说明新要求影响哪一部分

OpenAI 的指南建议在运行中补充要求，并明确哪些独立工作可以继续。GitHub 上 9 月 30 日的 Codex 改动把用户明确做出的目标修改写入模型历史，测试覆盖线程恢复和上下文压缩。11 月 15 日的提交则让终端里较短的粘贴目标直接显示在编辑器里。这些提交提供了可核实的产品实现依据。 [OpenAI 长任务指南](https://openai.com/index/practical-guide-building-gpt-6/) · [GitHub，保存用户目标修改](https://github.com/openai/codex/commit/de020167989fde4937204a064ea2fedd54f3c931) · [GitHub，终端目标编辑器](https://github.com/openai/codex/commit/9017fd33f3b9b2485fef9d061ed3802aa47bf480)

```text
更新导出要求：本次交付只覆盖 CSV。先完成 CSV 的实跑和测试。把已开始的 XLSX 工作记录到交接里，等我决定是否作为下一项任务。
```

### 重复纠正的问题才值得写进常驻指令

OpenAI 建议在 `AGENTS.md` 里写明什么时候读哪些文档、什么时候运行哪些测试、哪些常规动作已经授权。Skill 描述写清触发条件，细节按需加载。Addy 建议升级模型时在代表性任务上删减为老版本写的指令，再根据实际遗漏加回。Claude Code 团队会从多次人工评审或回滚里提取可识别的漏项，再决定是否固化为规则。 [OpenAI 工作指南](https://openai.com/index/practical-guide-building-gpt-6/) · [Addy 的 X 文章](https://x.com/addyosmani/status/2115688494865465370) · [团队验证文章](https://claude.dev/blog/how-the-claude-code-team-thinks-about-verification/)

## 可直接发给 Agent 的模板

以下是根据上述材料整理的中文模板。简单任务写前三项就够，长任务再补充自主范围和交付证据。

```text
目标：[做成什么，给谁用，交付成什么形式]。
投入：[快速做一个可试用版本 / 充分验证再交付]。参考：[关键文件、链接或样例]。
完成条件：[实际运行的命令、用户操作、期望结果或可核对的来源]。
范围：[这次涉及的功能和约束]。可逆的细节自行决定，涉及[具体决策]时问我。
持续做到交付物满足完成条件。最后给我结果、验证证据和仍需我决定的事项。
```

## 原文阅读顺序

| 日期 | 来源 | 值得读的部分 |
| --- | --- | --- |
| 09-25 | [Thariq Shihipar，Using Claude Code: Spending your effort](https://claude.dev/blog/spending-your-effort/) | 访谈补规格、低 effort 实现、高 effort 验证的个人流程 |
| 09-28 | [Latent Space 对话 Thariq](https://latent.space/p/thariq/) | 15:37 如何建立对 Agent 能力的判断，21:52 初始上下文，25:56 实现笔记 |
| 09-28 | [Anthropic，Prompting Claude Opus 5.5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) | `Calibrate effort` 和 `Unattended agentic runs`。指南没有独立署日，[9 月 28 日报道](https://www.searchenginejournal.com/anthropic-claude-opus-5-5-prompting-guidance/591278/)确认当时已公开 |
| 10-02 | [OpenAI，A model guide for the GPT‑6 family](https://openai.com/index/practical-guide-building-gpt-6/) | 第 2、3 节，含 OpenAI 开发者体验团队成员 Eric Provencher 对指令粒度的说明 |
| 10-06 | [Boris Cherny，X 原帖](https://x.com/bcherny/status/2107565388250874193) | 目标、精力、验证三项，以及实际交付的播客互动网页 |
| 10-29 | [Addy Osmani，You're probably overprompting Claude](https://x.com/addyosmani/status/2115688494865465370) | 用可检查的目标控制长任务，通过截图和命令输出验收 |
| 11-16 | [Claude Code 团队如何验证](https://claude.dev/blog/how-the-claude-code-team-thinks-about-verification/) | 验证 Skill、端到端检查、性质测试、差分测试 |
| 11-17 至 11-18 | [OpenAI，Using GPT‑6.2](https://developers.openai.com/api/docs/guides/latest-model/gpt-6.2-astra) | `Task completion` 与 `Frontend design` 两节。指南没有独立署日，由 [11 月 17 日发布公告](https://openai.com/index/introducing-gpt-6-2-astra-sol/) 和 [11 月 18 日报道](https://www.searchenginejournal.com/openai-gpt-6-2-astra-prompting-guidance/597877/) 确认同期已公开 |
