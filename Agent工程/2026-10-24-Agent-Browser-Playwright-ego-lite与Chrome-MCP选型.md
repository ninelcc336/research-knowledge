---
title: "Agent Browser、Playwright、ego lite 与 Chrome MCP：面向 Agent 的浏览器工具选型"
date: 2026-10-24
category: "Agent工程"
tags:
  - Agent
  - 浏览器自动化
  - Playwright
  - MCP
  - CLI
  - Coding Agent
  - Token成本
---

# Agent Browser、Playwright、ego lite 与 Chrome MCP：面向 Agent 的浏览器工具选型

> 调研日期：2026-10-24。目标是比较 Agent 的任务完成率、失败恢复、模型往返、Token 与实际结果成本。本文核查官方文档、公开运行时说明、两组厂商基准及部分问题报告；未在本机安装并对四套工具进行同条件端到端跑分。功能事实、厂商测试与选型判断分别说明。

## 1. 先给结论

**没有充分证据证明四者中存在适用于所有 Agent、所有网站的唯一冠军。** 但可以明确给出使用建议：

| 实际目标 | 优先选择 | 理由与条件 |
|---|---|---|
| 给能运行 Shell 的编码 Agent 配一套通用浏览器工具 | **Agent Browser** | 短命令、结构化快照与元素引用、范围过滤、增量输出和批处理，适合作为跨项目通用入口；这是工程默认建议，不是已证实的成功率第一 |
| 本地 Agent 经常操作已登录 SaaS、社交平台、后台，且人也在同时用浏览器 | **ego(lite)，优先试用验证** | 浏览器自带 Task Space、登录资料导入与 JS 工作流，减少连接、切页和人工接管的摩擦；厂商真实网站基准表现突出，但优势尚不能普遍化 |
| 已有 Playwright 项目，AI 要生成、修改、修复测试 | **Playwright CLI + Skills，最终沉淀为 Playwright Test** | Agent 交互入口较轻，定位、等待、断言、Trace、CI 与测试资产衔接更自然 |
| 固定流程需要大量重复执行 | **Playwright Library/Test 或确定性脚本** | 首轮由 AI 编写并验证，以后直接运行，通常不需要每一步都调用模型 |
| AI 要排查慢页面、失败请求、控制台错误、内存泄漏 | **Chrome DevTools MCP / CLI** | 浏览器诊断信息更直接，有机会减少找原因的返工；不应只按点按钮的 Token 比较 |
| 现有 Agent 只能接 MCP | **Playwright MCP**；按任务再选 DevTools 或社区 Chrome MCP | 客户端能力决定可用入口，CLI 的理论优势在不能执行 CLI 的环境里没有意义 |
| 必须保留原生 Chrome、已有扩展、标签页、书签和历史记录 | **社区 Chrome MCP，如 hangwin/mcp-chrome**，或其他工具的连接已有浏览器模式 | 主要优势是环境复用，没有可靠依据证明其通用任务成功率或 Token 成本领先 |

对本次问题的直接回答是：**通用编码 Agent 默认用 Agent Browser；以真实登录网站上的个人助理任务为主，ego(lite) 是值得优先对照实测的候选；测试工程用 Playwright，浏览器诊断用 Chrome DevTools。**

如果只问“哪种接口形态更适合会写代码的 Agent”，倾向于：**按需发现能力、能组合执行动作、只返回必要结果的 CLI/JS 接口。** 这个优势可以由多种产品实现，不能简单归结为某个品牌或“MCP 一定不适合 AI”。

## 2. 先对齐比较对象

### 2.1 四个名称不处于同一层

| 名称 | 实际是什么 | Agent 通常怎么使用 |
|---|---|---|
| Agent Browser | Vercel Labs 的 `agent-browser`，以 Agent CLI 为核心，也提供 MCP | Skill + Shell，读取带 `@eN` 的文本快照后调用命令 |
| Playwright | 浏览器自动化代码库与测试生态，另有官方 CLI、Skills 和 MCP | 编写 TS/Python 等代码；或使用 `playwright-cli`；或调用 `@playwright/mcp` |
| ego lite | Citro Labs 的 ego(lite)，一个为人与 Agent 共用设计的 Chromium 浏览器 | `ego-browser` Skill/运行时，在任务 Space 中执行 JS |
| Chrome MCP | 泛称，不是唯一产品 | 下述两类必须分开 |

“Chrome MCP”至少包括：

- **Google 官方 `ChromeDevTools/chrome-devtools-mcp`**：DevTools 诊断与自动化，底层使用 Puppeteer；当前同时提供 `chrome-devtools` CLI、Skills 和 MCP。
- **社区 `hangwin/mcp-chrome`**：Chrome 扩展 + 本地桥接，重点是复用日常 Chrome、登录态、标签页和浏览器数据。

这里比较的是 Agent 入口与任务环境。CDP 是浏览器控制协议，MCP 是 Agent 与工具之间的协议，Playwright 是自动化框架；它们能出现在同一套方案中。

### 2.2 版本与资料快照

2026-10-24 查询 npm `latest` 得到：`agent-browser` **0.39.2**、`@playwright/cli` **0.1.22**、`@playwright/mcp` **0.0.83**、`chrome-devtools-mcp` **1.12.0**、`mcp-chrome-bridge` **1.0.31**。这是包元数据，不表示本机安装或验证过这些版本。

本次读取的官方仓库主分支快照：

| 仓库 | Commit 前 12 位 |
|---|---|
| vercel-labs/agent-browser | `372ac5ce20fa` |
| microsoft/playwright-cli | `b1a5903c5b63` |
| microsoft/playwright-mcp | `0c4107acbecb` |
| citrolabs/ego-lite | `d392204d1ea1` |
| ChromeDevTools/chrome-devtools-mcp | `ada7963246c0` |
| hangwin/mcp-chrome | `f48e71751e00` |

主分支文档可能领先于发布包；具体命令应以锁定版本的帮助和 Skill 为准。ego(lite) 的公开仓库与浏览器二进制是不同交付物：仓库内容采用 MIT，不能据此把整个浏览器称为 MIT 开源。

## 3. 什么才是“AI 用着舒服”

应将主观描述转成能检查的指标：

| 用户标准 | 可测指标 | 工具应提供的帮助 |
|---|---|---|
| 用着舒服、容易完成 | 完成一个任务需要的模型轮数、人工接管次数 | 简洁观察、明确目标、批量动作、可恢复的会话 |
| 返工率低 | 首次完成率、无效动作数、重试次数、整任务重跑次数 | 等待正确状态、定位歧义报错、失败原因清楚、验证结果 |
| Token 少 | 非缓存输入、缓存输入、输出/推理、图像输入分别记录 | 小快照、定向读取、避免重复全页和无关工具定义 |
| 成本低 | 含失败和恢复的总费用 ÷ 验收成功数 | 减少无效尝试，而非仅缩短某一条工具响应 |
| 完成得轻松 | 端到端耗时 P50/P95、安装与登录准备时间 | 状态复用、少连接故障、方便观察与接管 |

**一次工具调用成功，不代表业务任务成功。** 点击返回成功之后，仍需确认页面出现目标状态、文件实际下载或记录确实保存。页面引用也不是跨导航永久有效的业务 ID。

## 4. 各方案对 Agent 的真实取舍

### 4.1 Agent Browser：通用 Agent 的工程默认选项

核心优点是 Agent 可直接读取 accessibility 快照，用 `@eN` 引用动作；支持交互元素过滤、压缩、深度与 CSS 范围限制、`snapshot --delta`、`diff snapshot`，以及 `batch --bail`。登录态可通过 profile、state、会话恢复或连接已有 Chrome 复用。[S1]

这意味着“Agent Browser 每次只能一个动作、不能复用登录、必须全页快照”的说法不成立。批处理还能减少进程启动和模型调用之间的开销。需要区分：**CLI 命令数、执行工具调用数、模型轮数不是同一个数**；Agent 可以在一次执行工具调用中顺序运行多条命令。

局限是独立命令与状态管理仍需要学习；批处理本身不像完整 JS 流程那样自然支持条件分支。过度使用 `snapshot -i` 会遗漏正文或非交互状态，之后反而需要补读。最新文档说明当前原生 Rust 守护进程无需 Playwright 或 Node.js，不能再照旧资料把它描述为“只是 Playwright 包装器”。

与返工直接相关的例子是公开 Issue #1922：报告者在 0.37.1/0.38.0 路径中发现，点击触发导航后旧引用仍可被接受并报告成功，实际未动作。本次查询时 Issue 仍开放，但本文未复现，不能据此断言 0.39.2 一定仍有该问题。它说明**导航后重新观察并验证业务状态**比相信 `Done` 更可靠。[S13]

### 4.2 ego(lite)：真实登录网站上的使用体验值得优先验证

它的差异不只是命令语法，而是把浏览器、任务 Space、登录资料和 Agent JS 运行时一起提供。人可以继续使用自己的标签页，Agent 在自己的 Space 工作；需要时可以查看和接管。JS 流程可以组合动作、等待、循环、筛选，并只把必要信息打印给模型。[S4][S5]

对于“找表格 → 筛选 → 填表 → 导出”这类任务，少处理登录、连接和焦点冲突，可能比单次快照减少几个 Token 更重要。官方当前提供 macOS 和 Windows，Linux 尚在路线图；它也不是 Playwright Test 这样的测试执行器。

需要区分已实现能力与宣传：

- “经验积累，越用越快”在 README 中仍标注 **coming soon**，不计入稳定现有优势。
- 当前公开运行时 Skill 要求按已知状态组合动作，遇到会改变下一步决策的中间状态时再观察。它不会让模型在不读取新信息时自动获得新判断能力。
- 公共文档的旧 API 示例和当前 `taskSpace()` / `page.snapshot()` 风格有变化，应使用安装版本的 `ego-browser skill`。
- Windows PowerShell 5.1 的引号与编码是官方 Skill 特别处理的问题，推荐 `.mjs` 文件方式，不能直接照抄 Bash heredoc。
- Task Space 主要是任务与标签页的工作区隔离；共享登录资料时，不应把它理解为多个账号之间的独立身份隔离。

它的“更少返工”也有现实前提：Chrome 资料能顺利导入。公开 Issue #546 报告 Windows 上 ego 0.6.0.22 导入 Chrome 156 资料因数据库兼容性失败，并有另一位用户确认。本次查询时仍开放；这是具体用户报告，不代表所有环境，但与你当前 Windows 使用场景直接相关。[S14]

此外，官网宣传能可靠处理深层 iframe，公开仓库的 native snapshot 需求文档却仍记录某些 iframe/OOPIF 可读但缺少动作 ref、frame 元数据或定位歧义的边界。当前 Skill 已提供子树与 frame 搜索工作流，因此也不能反过来断言完全不支持 iframe；准确结论是：**不要将宣传中的“都能读、都能点”视作已经对所有版本证实。**[S15]

### 4.3 Playwright：必须拆分代码、CLI 与 MCP

**Library/Test** 适合把可重复流程写成程序。定位器、动作前自动等待、可重试断言、Trace 和跨浏览器测试生态能减少长期维护中的不确定性。使用语义定位或 test id，不等于录下一串脆弱 CSS；页面改了也不必然每次都重写。[S11][S12]

**CLI + Skills** 则是官方专门给编码 Agent 的入口。微软在 CLI 和 MCP 两个 README 中都明确说明，能运行命令的编码 Agent 应考虑 CLI，以避免大型工具 schema 和冗长页面树占用上下文。CLI 支持引用、局部快照、深度限制、查找和会话持久化；它并非只适合人手写脚本。[S2][S3]

**MCP** 适合只能通过标准工具接口工作的客户端，以及持续观察页面的交互循环。默认快照策略可能让上下文变大，但可配置 `--snapshot-mode none`、按需快照；还支持多字段填表和代码执行工具，因此不能把“一动作必定一轮模型”当作协议限制。代码执行工具具体名称与可用性需匹配目标版本。

登录能力也不是空白：MCP 默认持久 profile，支持 storage state、CDP 和浏览器扩展连接已有 Chrome/Edge。ego、社区 MCP 或第三方文章中“Playwright 只能全新浏览器、无法复用登录态”的绝对描述与官方文档不符。[S3]

对于重复任务，成本应按全生命周期算：第一次用 AI 探索、编写和修复；之后脚本直接运行。即便首轮探索多花 Token，多次执行后的单位结果成本仍可能最低。

### 4.4 Chrome DevTools MCP / CLI：诊断任务可能最省返工

官方提供可访问性树和 uid，自动化有动作等待；并非只能截屏识图或手写 CSS。它擅长网络请求、控制台、性能 Trace、Lighthouse 和可选内存调试工具。[S7][S8]

当前还提供 CLI 与 Skills；MCP 的 `--slim` 只暴露导航、脚本执行和截图三类工具。普通动作的 `includeSnapshot` 当前默认 false，多字段 `fill_form` 可降低调用次数，快照和脚本结果可落文件；这些都意味着 Token 成本取决于工具配置与输出方式，而非“Chrome MCP”这个名称本身。[S8][S9]

若只是网页搜索和填表，全量诊断工具未必值得常驻。但若问题是“结账按钮无效是接口、脚本异常还是性能导致”，一次精准读取请求或控制台可能比反复重试页面动作更省总 Token。

### 4.5 社区 Chrome MCP：主要购买的是环境便利

`hangwin/mcp-chrome` 用扩展和本地桥接连接日常 Chrome，提供页面内容、交互元素、网络、控制台、标签页语义搜索、历史和书签等能力。对于必须利用既有扩展和真实标签页的任务很方便。[S10]

安装链涉及扩展、Native Messaging 桥接和 MCP 客户端，多一层并不自动意味着不稳定，但每层都可能产生连接排查工作。缺少与其他三者同条件且可审计的结果，本文不对其成功率、返工率和 Token 排名。

## 5. 实测证据究竟支持什么

### 5.1 Citro Labs 的 Real-World Bench：ego 明显领先，但有适用范围

ego 官方发布了同一组 **31 个真实网站任务**的结果：使用 pi Agent、`gpt-5.6-sol`、最高思考档及同一 rubric judge。README 基准图给出的数值如下：[S4][S6]

| 工具入口 | 全部要求通过率 | 平均模型轮数 | 平均耗时 | 每次任务尝试平均模型费 | 每个成功结果模型费，按前两列成本/成功率推算 |
|---|---:|---:|---:|---:|---:|
| ego-browser | 93.5% | 26.5 | 398 秒 | $1.64 | 约 $1.75 |
| playwright-cli | 71.0% | 44.0 | 648 秒 | $3.42 | 约 $4.82 |
| agent-browser | 62.9% | 45.6 | 605 秒 | $2.66 | 约 $4.23 |
| chrome-devtools-cli | 61.3% | 44.9 | 587 秒 | $4.95 | 约 $8.08 |

在这组测试里，ego 比 agent-browser 平均耗时少约 **34.2%**，比 playwright-cli 少约 **38.6%**；每个成功结果费用分别低约 **58.6%** 和 **63.7%**。官网用“52%/63% faster”表达速度优势，不能改写成“耗时减少 52%/63%”：两者分母不同。

该测试很贴近真实登录网站助理场景，值得认真参考，但必须带上限制：

1. **是 ego 自己发布的测试，不是中立机构复测。** 工具版本、Skill、登录准备和任务选择都会影响结果。
2. **它测试的是 CLI，不能直接当成 Playwright MCP 或 Chrome DevTools MCP 的实测。** 官网某些 MCP 对比页复用 CLI 结果，是推断而非 MCP 独立参赛结果。
3. 图下注明 ego-browser 与 chrome-devtools-cli 跑一次，其他三个工具跑两次，按各自全部轮次汇总；点估计可参考，方差不能视作等同。ego 结果标注为 2026-08-25 批次，无法代表 10 月最新版本。
4. 公开了测试框架、数据集与评分规则，但本次检查的框架主分支只看到单独的“16 模型比较”报告，没有定位到这张五工具图全部匹配的原始 session 与逐任务 judge 包。不能直接宣称本文已独立复算全部成绩。
5. 美元费用不是 Token 数。它受缓存输入、输出/推理长度和费率影响；公开汇总图不足以给四者作精确的总 Token 排行。
6. 完成率不是重试率或返工率，平均轮数也不能全部解释为无效返工。它们只是相关指标。

### 5.2 另一家厂商的结果：四者成功率很接近，耗时排序反转

HyperTeams 发布了使用 Claude Code、同一 `claude-sonnet-5` 的 **400 场景**测试，其中 **392 个功能任务**加 8 个提示注入场景。功能任务包含 REAL 121、WebGames 53、MiniWoB++ 128、GAIA web 90。其 392 项总体准确率为：[S16]

| 工具 | 总体准确率 | REAL 成功任务耗时中位数 | MiniWoB++ 成功任务耗时中位数 |
|---|---:|---:|---:|
| Playwright MCP | 80.9% | 92 秒 | 36 秒 |
| Agent Browser | 80.6% | 90 秒 | 93 秒 |
| ego lite | 80.4% | 190 秒 | 107 秒 |
| Chrome DevTools MCP | 80.1% | 85 秒 | 69 秒 |

这里准确率差不到 1 个百分点，ego 的耗时也没有领先。这不能与上一组数值直接合并：模型、任务、入口、统计口径和工具版本不同；耗时这里只统计成功任务的中位数，上一组是全部尝试的平均数。

HyperTeams 同样是竞品厂商，本文未复跑其成绩，也不接受其所有定性比较。例如页面把 Playwright/Agent Browser/DevTools 统称为无法观看或接管，与这些工具的有头、dashboard 等能力不符。其“真实站点”部分使用克隆服务，也不能等同于带真实登录与动态风控的公网网站。

**两组材料共同支持的结论：ego 在某类真实任务中可能有明显优势，但“所有 Agent 都成功率最高、最快、最省 Token”没有被证实。**

## 6. Token 消耗的关键，不是 CLI 与 MCP 的标签

概念上，一次任务的模型开销来自：

```text
工具说明/Skill
+ 各轮页面观察和错误输出
+ 被后续模型调用再次计入的上下文
+ 动作代码和推理输出
+ 截图等多模态输入
+ 失败、恢复及整任务重跑
```

实际账单再区分缓存输入、非缓存输入和输出，不能把它们用同一价格简单相加。

以下做法通常比品牌差异更直接：

- 读正文时用正文提取；点控件时用局部结构化快照；视觉问题再截图。
- 当前信息足够时，将填写、选择、点击、等待目标状态组合执行；中间结果会影响决定时才回到模型。
- 只输出需要的字段和最终状态；不要把 HTML、全页树、网络体和截图每轮全部返回。
- 页面导航或结构改变后刷新引用；等待 URL、元素或应用状态，避免固定 sleep 后盲点。
- 使用少量相关工具，按需发现能力。现代 MCP 客户端可以延迟加载工具定义，不能假设所有 MCP 都预先塞入完整 schema。
- 不要把“先只读 100 字、失败后反复补读”误认成节省；应优化每个验收成功结果的总消耗。

ego 的 JS 组合、Agent Browser 的 batch/delta、Playwright CLI 的局部快照、DevTools 的 slim/文件输出，都能实现其中一部分。**Code-first 是接口设计上的优势，不是 ego 独占的能力；MCP 的问题通常是工具粒度与默认输出，而不是 JSON-RPC 本身有固定的巨额 Token 成本。**

## 7. 给当前需求的落地建议

若你希望 Codex、Claude Code 等长期拥有一个通用工具，**先用 Agent Browser 作为默认入口**。它无需迁移日常浏览器，能服务普通页面、自动化任务和开发工作流；配套安装版本一致的 Skill，按需读取命令即可。

若主要任务是操作已经登录的真实网站，**用 ego(lite) 做下一轮优先试用**。一旦在你的 Windows 环境验证登录资料导入、Space 与目标站点可靠，它可能比通用 CLI 更省人工配合。当前资料不足以直接承诺换上就降低多少 Token 或返工。

若目标是测试项目，直接采用 **Playwright CLI + Test**，避免把已经确定的回归流程继续交给模型逐轮探索。Chrome DevTools 作为需要诊断时才启用的工具；社区 Chrome MCP 留给确实依赖原生 Chrome 环境的数据与扩展任务。

最终选择可用一个小规模验收来决定，无需先全面迁移：挑选你自己的 10–20 个代表任务，所有工具使用同一模型、思考档、登录起点和验收标准；各工具使用其推荐 Skill 与合理配置。分别记录首次成功率、无效动作、人工介入、模型轮数、各类 Token、含失败总费用及 P50/P95。登录准备成本与正式执行成本分开列出；每种任务重复 3–5 次、轮换顺序，并用站点状态或产物验收。选取**成功结果成本最低、人工介入最少、长尾耗时可接受**的方案。

## 8. 与仓库旧调研的关系

本仓库 2026-10-17 的《Agent Browser 与 Playwright MCP 对比调研》仍有场景拆分价值，但其中“Agent Browser 使用 Playwright 后端”的描述与本次当前官方文档不符，后续引用应以本篇及目标版本源码为准。旧篇关于“哪个更省上下文”的绝对表达也应结合 batch、按需工具发现、局部读取和完整任务成本重新判断。本文未修改旧调研正文。

## 9. 资料来源与证据性质

- **[S1] Agent Browser 官方仓库/当前能力**：[README 固定快照](https://github.com/vercel-labs/agent-browser/blob/372ac5ce20fab885dbf45ee0c9d510ec8c3b6fdb/README.md)。核查 Rust/CDP、batch、delta、profile、MCP 分组与版本匹配 Skill。
- **[S2] Playwright CLI 官方仓库**：[README 固定快照](https://github.com/microsoft/playwright-cli/blob/b1a5903c5b633b8c9f843ad7b629c2a78437040c/README.md)。核查 CLI/Skills 定位、Token 设计、快照、session、CDP 与持久状态。
- **[S3] Playwright MCP 官方仓库**：[README 固定快照](https://github.com/microsoft/playwright-mcp/blob/0c4107acbecb75472b23c05c0e42471a0011874f/README.md)。核查持久 profile、扩展、storage state、snapshot mode 与工具。
- **[S4] ego lite 官方仓库**：[README 固定快照](https://github.com/citrolabs/ego-lite/blob/d392204d1ea1a8027a591ab2b0129c2e11d85f26/README.md)。产品定位、平台、Space、基准条件、coming soon 和许可范围。
- **[S5] ego 当前公开运行时 Skill**：[正文固定快照](https://github.com/citrolabs/ego-lite/blob/d392204d1ea1a8027a591ab2b0129c2e11d85f26/package/ego-browser/skill-body/SKILL.md)。实际 API、Windows Shell 处理、观察/执行边界与 iframe 工作流。
- **[S6] ego 厂商基准**：[原始成绩图](https://github.com/citrolabs/ego-lite/blob/d392204d1ea1a8027a591ab2b0129c2e11d85f26/docs/assets/real-world-bench.jpg)、[Agent Browser 对比页](https://lite.ego.app/compare/ego-lite-vs-agent-browser)、[Playwright 对比页](https://lite.ego.app/compare/ego-lite-vs-playwright)、[测试框架](https://github.com/citrolabs/ego-browser-benchmark-framework)。图表是厂商结果，本文没有独立复测；五工具图不同于仓库已有的 16 模型报告。
- **[S7] Chrome DevTools 官方仓库**：[README 固定快照](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/ada7963246c0c91eae185125a57aecd13cbe0376/README.md)。诊断定位、Puppeteer、CLI 与 Skills。
- **[S8] Chrome DevTools 工具参考**：[固定快照](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/ada7963246c0c91eae185125a57aecd13cbe0376/docs/tool-reference.md)。a11y uid、fill_form、includeSnapshot、文件输出、性能与内存工具；部分工具需选项启用。
- **[S9] Chrome DevTools 配置与 CLI**：[配置](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/ada7963246c0c91eae185125a57aecd13cbe0376/docs/configuration.md)、[CLI](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/ada7963246c0c91eae185125a57aecd13cbe0376/docs/cli.md)。核查 slim 和连接已有 Chrome。
- **[S10] 社区 Chrome MCP**：[hangwin/mcp-chrome 固定快照](https://github.com/hangwin/mcp-chrome/blob/f48e71751e00bc09725c7e173423cff4f2ccd12a/README.md)。核查扩展/桥接安装、登录复用与工具范围；其自有 Playwright 比较表有过度概括，不作事实依据。
- **[S11] Playwright 动作等待与断言**：[Auto-waiting](https://playwright.dev/docs/actionability)。官方说明动作可执行性检查及重试断言。
- **[S12] Playwright 定位器**：[Locators](https://playwright.dev/docs/locators)。官方语义定位、test id 与动态重新解析机制。
- **[S13] Agent Browser 用户问题报告**：[#1922](https://github.com/vercel-labs/agent-browser/issues/1922)。特定旧版本报告；本文未复现，不推广为当前所有版本结论。
- **[S14] ego Windows 用户问题报告**：[#546](https://github.com/citrolabs/ego-lite/issues/546)。Chrome 156 profile 导入错误及独立用户确认；查询时开放，本文未本机复现。
- **[S15] ego native snapshot 需求文档**：[固定快照](https://github.com/citrolabs/ego-lite/blob/d392204d1ea1a8027a591ab2b0129c2e11d85f26/docs/native-snapshot-ref-requirement.md)。记录 iframe ref、frame 归属和 locator 边界；不直接等同于最新闭源浏览器二进制的缺陷清单。
- **[S16] HyperTeams 竞品厂商基准**：[Benchmark 页面](https://hyperteams.net/benchmarks)。392 功能任务准确率、成功耗时中位数、版本和统计方法。本文未独立复跑，定性功能对比需另行核查。

检索使用 agent-reach 的 Exa、GitHub CLI 和网页读取流程；受可用渠道影响，未将需要登录的社交平台评论计入结论。官方主分支、npm 与网页均为调研时快照。
