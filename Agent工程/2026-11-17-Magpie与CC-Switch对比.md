---
title: "Magpie 与 CC Switch 对比调研"
date: 2026-11-17
category: "Agent工程"
tags:
  - AI编程工具
  - 模型管理
  - 本地网关
  - API路由
  - Agent配置
  - 隐私脱敏
---

# Magpie 与 CC Switch 对比调研

调研日期：2026-11-17。版本快照取自当日 16:56 UTC 前的核查结果（America/Halifax 当日 12:56）。

核查范围：重新检查两个项目的官方 README、参考文档、发布记录和影响选型的源码；CC Switch 的关键能力另外对照 v4.0.34 稳定标签。本文是文档与源码调研，未安装两款程序、未登录订阅账号，也未实测延迟、内存、工具调用成功率或额度回退。

## 结论

**Magpie 更适合把多个 Agent、模型供应商和订阅账号组织成一个持续运行的模型网关；CC Switch 更适合在桌面上管理各个编程工具的配置、账号、MCP、Skills、提示词和会话。** 两者现在都有请求代理、协议转换和故障转移，CC Switch 也已加入隐私过滤，不能再用“网关对静态开关”概括差异。

- **主要使用 Claude Code、Codex，想切供应商、维护配置和查看会话：优先 CC Switch。** 这两个客户端本身已有路由与模型聚合支持，不必仅为基础代理和回退迁移到 Magpie。
- **希望不同 Agent 共用 Provider/模型目录，多账号按额度调度，跨客户端使用受支持订阅，或把网关放到服务器/NAS：优先 Magpie。** 它提供面向模型、账号与 Key 的统一路由组，以及 CLI、TUI、Web UI 和 Docker 入口。
- **主要使用 Claude 官方订阅，需求是手动换自己的账号或每个终端固定一个账号：CC Switch 的专门账号工作流更贴合。** Magpie 的 Claude 订阅功能则通过本地真实 `claude` 进程加 MCP 桥接，把能力提供给其他 Agent；两者的请求路径不同。
- **只有一个 Agent，官方订阅已经够用：没有充分理由仅为“支持更多模型”增加一层网关。** 是否换工具取决于具体缺口，不能据功能清单推断 Magpie 更快、更稳定或更省额度。

## 本次更新

本篇更新并替代 2026-10-27 旧稿，保留其配置管理、网关、协议转换、订阅、插件、用量、隐私及组合使用的研究范围，主要修正与补充如下：

1. 将旧快照 Magpie v0.1.1706、CC Switch v4.0.21 更新为本次版本快照；补充 Magpie 的 Every version、Daily、Stable 三个频道，区分 GitHub 最新发布与稳定频道。
2. 补充 CC Switch v4.0.30 的隐私过滤、v4.0.31 的出口过滤覆盖及 WorkBuddy、v4.0.34 的多平台 Skill 仓库与兼容性修复。
3. 修正旧稿“CC Switch 聚合仅支持 Claude Code/Codex”的范围：Claude Desktop 在 10 月 16 日的 v4.0.13 已加入聚合，旧稿漏记；v4.0.34 源码再次确认支持，但 Desktop 没有完整路由故障转移功能。
4. 细化 Claude 官方订阅管理与跨客户端桥接的区别；补充 CC Switch 的 Claude 账号功能不支持 WSL，以及 Magpie 的 Antigravity 接入变更。
5. 补充 Magpie 的意图路由、更多路由策略、可还原脱敏和更新验证机制。这里只把发布记录明确的内容称为近期新增，其余是本次补充核查，不据旧稿未提及就断言当时不存在。

## 版本、平台与项目状态

| 项目 | Magpie | CC Switch |
| --- | --- | --- |
| 官方仓库 | [yetone/magpie](https://github.com/yetone/magpie) | [farion1231/cc-switch](https://github.com/farion1231/cc-switch) |
| 核心实现 | Go；当前依赖含 Wails v3；另有终端与无桌面运行方式 | Rust + Tauri 2，React/TypeScript 桌面界面 |
| 主要平台 | macOS、Windows、Linux；CLI/TUI/Web UI、Docker、Termux | macOS、Windows、Linux 桌面程序；Windows 可管理 WSL 内部分工具配置 |
| 许可证 | MIT | MIT |
| GitHub 最新发布 | **v0.1.4394**，2026-11-17 16:50:05 UTC | **v4.0.34**，2026-11-17 07:06:04 UTC，标记 Latest |
| 其他发布口径 | Stable **v0.1.3960**，该版本发布于 11-12 23:26:50 UTC；Daily **v0.1.4338**，该版本发布于 11-16 19:21:54 UTC | CHANGELOG 将 4.0.34 标为 **2026-11-16**，这是日志日期，与 GitHub 实际发布时间不同 |
| 本次源码快照 | `81c666e1c23ec2ca550bd404a0d7891a862569c6` | 稳定标签 v4.0.34，提交 `91245fae8f162a1a3aa5cfe1f6450c36065cb0f5` |

Magpie 的发布资产在 `yetone/magpie-releases`，不应只看源码仓库的 Releases。其默认 Every version 跟随每次发布；Daily 通常每天 20:00 UTC 选择测试通过且资产完整的一个版本；Stable 通常每周一选择已发布至少三天的版本，紧急版本可提前进入频道。**GitHub Latest 不等于 Magpie Stable。** 本文 Magpie 功能以所列最新源码和官方文档为准，不保证每项都已进入上述 Stable 版本。[M1][M2][M3][M4][M5]

两者软件许可均为 MIT；模型 API、订阅或第三方服务仍按相应供应商计费。频繁发布说明维护活跃，不能替代实际兼容性验证。Magpie 当前 README 还明确表示不接收外部 PR，问题与建议主要在 Discord 反馈。[M1][C1][C2]

## Magpie 实际做什么

### 统一接入：把 Agent 指向本机模型网关

Magpie 在 `127.0.0.1:3425` 提供网关。Agent 仍负责读写文件、执行工具和管理任务；Magpie 负责把模型请求发给选中的上游。它适配 Agent 的原生配置，也允许其他接受 Base URL 的客户端手动接入。[M1][M2]

原生适配覆盖 Claude Code、Claude Desktop、Codex、Gemini CLI、OpenCode、Pi、Cursor CLI、Zed、VS Code Chat、Copilot CLI、Hermes、WorkBuddy、OpenClaw 等许多工具。这里的“支持”通常表示配置接入、模型映射等具体能力，不表示每个工具都具备相同的会话、MCP 或 Skills 管理功能。官方参考表逐项列出了文件位置、可修改字段和重启要求，选型时应查目标客户端那一行。[M2]

网关接收 OpenAI Chat Completions、OpenAI Responses、Anthropic Messages 和 Gemini 协议，并处理流式输出、工具调用、推理字段的转换。Provider 可以是官方 API、自定义兼容地址、Ollama/LM Studio 等本地服务，或受支持的订阅/插件。模型以 `provider/model` 统一呈现；Profiles 可以保存并切换多个 Agent 的模型配置。官方说明配置采用原子改写，保留注释、顺序与格式，并记录被替换值以便断开后恢复。[M1][M2]

**这是一层有状态的模型接入服务。** 被接管的模型请求依赖网关运行；它不会让不支持的模型凭空获得原生能力。跨协议转换后的工具、推理、多模态和特定客户端功能，仍受客户端、转换实现与上游三方约束。

### 运行时路由：模型、账号和 Key 一起调度

路由组可把不同 Provider 的模型组合为一个 `group/<id>`，组也可以包含组。最新参考文档列出的策略包括：[M2]

| 策略 | 选择逻辑 |
| --- | --- |
| `smart` | 优先使用仍有额度、且额度较快重置的订阅 |
| `order` | 按成员顺序尝试，失败后转向后续成员 |
| `rotate` | 在不同轮次轮换成员 |
| `usage` | 优先较少使用的成员 |
| `pace` | 按剩余周额度与距重置时间的关系安排使用 |
| `weight` | 按配置权重分配请求 |
| `fast` | 根据近期首 Token 时间和生成速度优先选择成员 |

会话亲和性可让同一对话继续使用已应答账号，以利用上游缓存；也有固定整段会话、逐轮或严格不换账号等设置。**选路策略、会话亲和性和故障转移是不同的开关与规则**：例如严格固定账号时，其错误会返回客户端，不能笼统理解成“有额度就永不报错”。只有一个账号、没有其他可用成员时也无法回退。[M1][M2]

意图路由允许用另一个模型判断当前用户消息属于哪类任务，再决定本轮优先使用哪个模型。例如测试任务优先强模型，简单问答优先便宜模型；规则还可结合长度、图像、推理和 Agent 类型。通常每个新用户轮次决定一次，同轮工具调用延续该决定。分类调用本身有延迟、费用和数据流向：消息会发给所选分类模型；超时或判断失败时按普通规则继续，而非保证选中某一强模型。官方当前默认分类超时为 15 秒。[M6]

### 订阅桥接：不能把所有账号视作一种接法

| 订阅/账号 | 当前官方资料描述的工作方式与边界 |
| --- | --- |
| Claude | 调用本地真实 `claude` 二进制，通过 MCP 桥接调用方工具；必须已安装并登录 Claude Code。它是进程桥接，不是直接把 Claude 订阅令牌当通用 API Key |
| ChatGPT/Codex | 读取受支持的 Codex 登录，接入 ChatGPT 后端；为其他 Agent 适配流式和参数限制，支持多账号 |
| GitHub Copilot、Grok 等 | 使用各自支持的登录/API 或插件提供模型；具体模型、额度与工具能力由上游和插件决定 |
| Gemini、Antigravity | 应按具体账号和当前接入方式判断；不能把所有 Google 个人订阅统称为可通用桥接的 Provider |

一个明确的资料冲突是 **Antigravity**：参考文档仍保留直接调用 Code Assist 的旧描述，但最新 README、项目开发说明与迁移源码表明，11 月 15 日起旧 Antigravity 插件停止向新用户提供安装，当前指向 **Antigravity（local agy CLI）** 插件，通过用户本机 `agy` 的 ACP 接口工作。旧用户仍有迁移路径。因此本篇不沿用旧稿“Antigravity 订阅直接接入”这一宽泛结论。[M1][M2][M9]

Magpie 默认还会发现部分本地已登录订阅，可在 Privacy 中关闭 Subscription accounts，或使用 `magpie accounts discovery off`。从外部文件导入账号也不等于无影响地复制：官方文档明确写明，ChatGPT 导入会刷新登录，原持有该登录的工具可能需要重新登录；Claude 的旧副本也会在后续刷新后失效。已有账号管理工作流迁移时要把这个行为计入方案。[M1][M2]

### 模型网关之外的配套能力

- **Library**：统一维护 instructions、MCP servers、Skills、subagents 等，再按 Agent 格式写入；并非 CC Switch 才具备这类管理能力。
- **插件与中间件**：可接入受支持的 OpenCode auth plugin、pi package；中间件可改写请求、观察流式事件或处理响应。插件宿主使用 Bun，首次需要时下载。
- **工具补充**：通过 MCP 给 Agent 提供搜索、网页读取、图像等工具；搜索可接多家 API 或 Provider，不能据此保证任意模型的原生搜索语义完全一致。
- **Agent 维护**：查看已安装 CLI 的版本，按原安装方式升级，并为未安装工具提供命令。
- **用量与观测**：网关请求、路由尝试、Token、缓存、推理、费用估算、额度窗口和会话；可导出 OTLP。订阅的 API 等价估价不代表订阅账单实际又扣了相同金额。
- **部署与共享**：本机 GUI/TUI/CLI、Web UI、远程 Magpie、局域网 Gateway Key、Docker；支持文件备份、WebDAV、S3、加密 GitHub 仓库备份等。Docker 的模型端口和管理 UI 是不同入口，分别涉及 Gateway Key 和管理登录。[M1][M2]

## CC Switch 最新能力与限制

### 桌面配置管理仍是中心，但已有完整代理功能

v4.0.34 源码列出 16 种受管应用：Claude Code、Claude Desktop、Codex、Gemini CLI、Grok Build、OpenCode、OpenClaw、Hermes、Pi、MiniMax Code、Kimi Code、DeepSeek Harness、Antigravity CLI、Oh My Pi、Copilot CLI、WorkBuddy。[C3]

它管理 Provider 卡片、原生配置、账号、MCP、Skills、Prompts、Projects、Sessions 和用量，但具体能力逐应用不同。例如 Claude Desktop 的能力表没有独立 Skills/MCP/Prompts 管理，OpenClaw 也不是标准的统一 MCP/Skills 管理对象。某些应用是切换当前配置，另一些是把多个 Provider 累加写进同一配置，不能把“支持 16 种应用”理解成全部功能乘以 16。[C1][C3]

切换 Provider 时主要替换连接字段，保留其他设置，并在首次改写前自动备份；Projects 可保存和切换整套受支持配置，Apps 页面可检测、安装或更新 CLI。配置保留也有具体边界：例如 4.0.23 的 Kimi/MiniMax/Oh My Pi 原生格式编辑视图明确不保存手工输入的注释，不能承诺所有编辑路径都完整保留注释。[C1][C2]

### Direct、Routing、Aggregation 要分开看

下表以 **v4.0.34 源码能力表** 为准；README 中部分矩阵尚未同步。[C3]

| 客户端 | Direct/原生配置管理 | Routing 本地转发 | 完整路由及故障转移 | Aggregation 模型聚合 |
| --- | --- | --- | --- | --- |
| Claude Code | 有 | 有 | 有 | 有 |
| Codex | 有 | 有 | 有 | 有 |
| Gemini CLI | 有 | 有 | 有 | 无 |
| Grok Build | 有 | 有 | 有 | 无 |
| Claude Desktop | 有 | 有 | **无** | **有** |
| 其余 11 种受管应用 | 按各工具格式管理 | 能力表未提供上述 Routing 模式 | 无上述完整功能 | 无上述聚合模式 |

- **Direct**：客户端按写入的配置连接上游，对话请求不经过 CC Switch；需要其协议转换的卡片应使用 Routing。
- **Routing**：受支持客户端经本地代理转发，可做协议适配；完整路由的四种客户端还支持健康检查、熔断和 Provider 故障转移。
- **Aggregation**：把多个 Provider 的模型放到客户端可选模型列表里，按模型 ID 分发；未指定聚合模型时使用默认 Provider。**聚合没有路由模式的自动故障转移**，不能把“模型列表里有多个 Provider”理解为失败后自动换一家。
- Claude Desktop 共享转发服务，但没有完整故障转移队列与路由设置；Claude 官方订阅目前也不支持加入聚合。[C1][C3][C4]

因此，仅需要 Claude Code/Codex 的 API Provider 回退时，CC Switch 已能覆盖；Magpie 的主要增量是跨更多客户端的统一模型入口、跨账号/Key 的调度、路由组及服务化运行。

### 官方账号：Claude 的限制尤其重要

CC Switch 的 Claude 官方账号指南明确支持两种工作流：一是切换卡片来切换 Claude Code 登录；二是通过“打开终端”为不同账号使用独立 `CLAUDE_CONFIG_DIR`。账号由官方 `claude auth login` 流程登录，模型请求由本地 Claude Code 直接发送。[C5]

这项能力有明确边界：

- 只支持 **Claude Code CLI 与 IDE 插件**；不改变 Claude Desktop 的登录账号，**也不支持 WSL 下的这套 Claude 多账号工作流**。
- 仅限直连；不提供按额度自动轮换 Claude 订阅，也不把 Claude 订阅作为通用跨客户端模型服务。
- 普通“Windows 管理 WSL 工具配置”与“WSL 支持 Claude 官方账号切换”是两件事，不能互相推导。

另一方面，Accounts 支持将部分 **ChatGPT、GitHub Copilot、xAI/Grok** 账号作为 Claude Code、Claude Desktop、Codex 的上游，这类使用通常依赖 Routing；不要把对 Claude 订阅的限制扩展成“CC Switch 完全不能跨客户端使用任何订阅”。[C1][C5]

### 10 月 27 日之后值得关注的变化

| 版本 | CHANGELOG 日期 | 与本次选型有关的变化 |
| --- | --- | --- |
| 4.0.22 | 10-30 | 加入 DeepSeek Harness 原生 Provider、MCP/Skills/提示词、会话用量与账号适配，不含上述本地 Routing |
| 4.0.28 | 11-08 | 适配 Codex 0.167/桌面会话归属，支持暂停官方登录，将官方额度保护改为默认开启 |
| 4.0.29 | 11-10 | 加入 Copilot CLI 的原生 BYOK、官方启动卡及 MCP/Skills/会话用量适配 |
| 4.0.30 | 11-11 | 加入默认关闭的本地隐私过滤；修复 Codex 原生 Responses 与第三方上游的多项兼容问题 |
| 4.0.31 | 11-12 | 新增 WorkBuddy 配置、MCP/Skills/提示词、账号及会话用量；将过滤移到请求出口，覆盖故障重试及代理生成的部分请求体 |
| 4.0.32 | 11-14 | 可选后台下载更新，网络出口 IP 诊断；继续修复 Codex、Copilot 等兼容问题 |
| 4.0.33 | 11-15 | 修复已打开 Claude Code 会话切回官方后仍沿用旧 Provider；支持 Windows Antigravity 账号切换，Linux 该项仍未支持 |
| 4.0.34 | 11-16 | Skill 仓库扩展到公开 GitLab/Gitea/Forgejo（含 Codeberg）；新增 WorkBuddy 推理等级设置，修复 Copilot 图像声明、Gemini 模型列表及国内 S3 兼容问题 |

日期为 CHANGELOG 日期，不是逐条 GitHub 发布时刻。4.0.34 的 Copilot 图像能力修复还要求旧卡片重新获取模型列表并保存、重启 Codex；升级程序不一定自动修正所有已有配置。[C2]

## 核心能力对照

| 需求 | Magpie | CC Switch |
| --- | --- | --- |
| 切换 Provider/模型 | 统一 `provider/model`、跨 Agent Profiles，写入 Agent 接入配置 | 按应用维护 Provider 卡片、默认模型及原生配置 |
| 多协议代理 | 核心网关架构，接收 Chat/Responses/Messages/Gemini | 已有本地代理与协议适配，范围取决于客户端、Provider 与模式 |
| 回退和分配 | 路由组跨模型、Provider、账号、Key，多种策略及会话亲和性 | 四种客户端有完整路由回退；Desktop 仅转发；聚合不自动回退 |
| Claude 多账号 | 网关订阅桥接，可把真实 Claude Code 进程用于其他 Agent | 管理官方 Claude Code 登录、即时手动换号、账号隔离终端；不自动按额度轮换 |
| 其他订阅 | 通过内置支持或插件统一接入，适配范围广 | 已有部分跨客户端订阅 Provider；账号种类与工作流分别判断 |
| MCP/Skills/提示词 | Library 统一维护，按适配写入 | 专门面板与安装/更新/同步工作流；各客户端支持不一致 |
| 用量和会话 | 网关级路由与请求记录、额度、成本估算、会话和 OTLP | 本地会话日志 + 代理用量、费用/额度、会话查看；多项新客户端适配 |
| 隐私过滤 | 默认关闭；规则 + 可选本地模型；占位符可还原 | 默认关闭；正则/gitleaks；占位符不可逆，主要作用于代理请求文本 |
| 远程与无桌面部署 | 官方提供 Docker、Web UI、远程网关与客户端 Key/限额 | 主项目分发 GUI 桌面程序；README 推荐的无头 CLI 是独立社区项目，配置同步不等于远程模型网关 |
| Windows/WSL | Windows 程序可接入 WSL 内 Agent、统计其会话 | 支持 WSL 配置管理，但 Claude 官方多账号功能明确排除 WSL |

CC Switch 的 WSL 配置不是自动发现，需要把工具配置目录指向 `\\wsl.localhost\...`。其 Routing/Aggregation 写入 `127.0.0.1`；在 WSL2 默认 NAT 网络中，WSL 回环不能直接访问 Windows 本地代理，官方建议使用 mirrored networking。这会直接影响 Windows + WSL 场景能否使用所选模式。[C1]

CC Switch README 推荐的 [CC Switch CLI](https://github.com/SaladDay/cc-switch-cli) 独立维护和发布；它与桌面程序共用默认数据目录，支持的数据库版本有时落后。因此不把该社区项目的能力直接计入本次桌面主项目比较。[C1]

## 隐私、数据与日常使用差异

### 两者都有脱敏，但结果不同

**Magpie** 的隐私脱敏默认关闭，开启后可将规则命中的秘密与个人数据替换为稳定占位符，返回文本和工具调用参数中可还原。除规则外，可选约 1.6 GB、CPU 运行的本地模型识别部分姓名、地址、密码；可设置 Mask/Warn/Block 和例外规则。这更适合“上游不看到原值，但本地工具仍要使用原值”的场景，仍不能视为识别所有秘密的保证。[M1][M7]

**CC Switch** 在 4.0.30 加入过滤，4.0.31 补齐请求出口覆盖。默认关闭，使用正则、结构化校验、gitleaks 规则和熵等方式；占位符**不可逆**。它过滤用户输入、系统提示等请求文本，默认不改模型回复、thinking、工具参数和图片；工具结果另有默认关闭的子开关。打开工具结果过滤后，模型可能拿不到原文，导致 Edit/apply_patch 匹配失败或把占位符写入文件，这一点在源码中有明确说明。[C2][C6]

CC Switch 的过滤只对经过本地代理的请求生效。两者都应按实际需要选择过滤范围，不能把“存在脱敏功能”等同于已默认保护全部请求和内容。

### 本机运行与遥测不是同一件事

Magpie 的对话请求由本机网关发往所选上游；此外正式发布版默认每日发送随机安装 ID、版本、系统、受控的 Agent/Provider/Model 标识、部分插件和合作方展示/点击计数。官方称不发送提示词、回复、密钥、账号和用量。可在 Privacy 中关闭，或设 `DO_NOT_TRACK=1`/`MAGPIE_NO_STATS=1`。[M7]

这些开关不等于阻断程序所有外联：更新、模型目录、图标和合作方链接等仍有各自请求；启用意图分类、搜索、插件或 OTLP 时也会按所选服务产生额外数据流。旧稿关于 Magpie 遥测可关闭的结论仍成立，但不能简写成“关闭后完全无外联”。[M1][M7]

CC Switch 的 Direct 模式不经其代理；其 Claude 官方账号指南说明令牌由 Claude Code 保存在本机的账号配置目录/钥匙串，不进入 CC Switch 的数据库或云同步。这是 **Claude 官方账号工作流** 的承诺，不应扩展成所有第三方 Provider、所有账号类型和同步内容都不含凭据。[C5]

### 更新、迁移与两者共用

Magpie v0.1.3862 起加入更新清单签名，v0.1.3891 起还有构建证明验证。官方参考同时说明首次 `install.sh` 下载仍信任站点 HTTPS 与其给出的 SHA-256，不验证该签名；不要把后续更新验证扩大为所有安装路径都做同样校验。[M2]

Magpie 支持导入 CC Switch 的 Provider 配置，但导入是一次复制，不是两个配置库实时同步。两者都能改写 Agent 配置；组合使用时，应明确当前 Provider 配置由哪一个管理。一个可行分工是 Magpie 接管模型入口，CC Switch 管理所需的周边配置与会话；这也不保证双方所有写入操作互不影响。[M1][M2]

若当前 CC Switch 已满足需求，不建议为了功能数量直接全量迁移。先在一个 Agent 上检查：新会话和续聊是否正常、工具调用是否返回、所需图像/推理是否保留、额度耗尽后是否按预期回退。需要切换整个工作流时，再处理账号导入及配置接管。此处是后续使用建议，本次没有执行这些实测。

## 选型建议

| 你的主要目的 | 建议 |
| --- | --- |
| 给 Claude Code/Codex 切几个 API，中转站偶发失败时回退 | 先用 CC Switch 的 Routing；确认所需协议和卡片可进入回退队列 |
| 集中安装/更新 Skills、管理 MCP/提示词、查看本地会话 | 优先 CC Switch；逐项检查目标客户端功能表 |
| Claude 官方订阅手动多账号、工作/个人终端隔离 | 优先 CC Switch 的 Claude 官方账号工作流；WSL 不在该能力范围内 |
| 多个 Agent 共用订阅/API，跨账号和 Key 按额度分配 | 优先 Magpie；单独核实目标订阅当前是否允许接入、是否依赖本地 CLI/插件 |
| 想按任务内容选模型，或组合强模型/低价模型 | 优先研究 Magpie 的意图规则；分类开销与误判需要实测 |
| NAS/VPS 部署，一台网关供多台电脑使用 | 优先 Magpie 的 Docker/远程形态；客户端使用真实 Gateway Key，区分管理 UI 登录 |
| 想在上游隐藏原值，又让本地工具得到真实值 | Magpie 的可还原占位符更贴合；CC Switch 不可逆过滤不适合直接替代这一用途 |

**对普通桌面编程工作流，CC Switch 仍是合理的默认选择；出现明确的统一网关、多账号运行时调度或远程部署需求时，Magpie 的价值才更突出。** 本次没有足够实测证据给出“谁更快、谁更稳定、谁一定更省钱”的排名。

## 核查来源

以下来源均于 2026-11-17 阅读；源码优先固定到本次提交或稳定标签。实时频道链接后续会变化，本文表格保留本次读取值。

- [M1]：Magpie 官方 README 固定快照；定位、接入、订阅、Library、插件、部署、隐私和反馈渠道。
- [M2]：Magpie 官方 reference 固定快照；路由策略、账号导入、配置、Docker、费用、更新频道与签名。
- [M3]：Magpie v0.1.4394 发布页；最新发布版本、时间及 advisor 用量修复。
- [M4]：Magpie Stable 官方更新 feed；本次返回 0.1.3960 / `2026-11-12T23:26:50Z`。
- [M5]：Magpie Daily 官方更新 feed；本次返回 0.1.4338 / `2026-11-16T19:21:54Z`。
- [M6]：Magpie 意图路由说明；分类规则、时机、默认超时、失败路径和额外成本。
- [M7]：Magpie 官方隐私页；遥测字段、关闭方式、文件、外联与归档说明。
- [M8]：Magpie `go.mod`；[MIT 许可固定快照](https://github.com/yetone/magpie/blob/81c666e1c23ec2ca550bd404a0d7891a862569c6/LICENSE)。
- [M9]：Magpie 项目开发说明中的 Antigravity 退出/迁移段落；[迁移实现](https://github.com/yetone/magpie/blob/81c666e1c23ec2ca550bd404a0d7891a862569c6/internal/provider/agy_move.go)。此处用于交叉核实已滞后的 reference 段落。
- [C1]：CC Switch v4.0.34 README；产品工作流、Routing/Aggregation、账号和使用方式。
- [C2]：CC Switch v4.0.34 CHANGELOG；重点核查 4.0.22—4.0.34，其中隐私过滤在 4.0.30 新增。
- [C3]：CC Switch v4.0.34 前端应用能力表；[后端应用能力表](https://github.com/farion1231/cc-switch/blob/v4.0.34/src-tauri/src/app_config.rs)，用于交叉核实 Desktop 聚合及故障转移范围。
- [C4]：CC Switch v4.0.34 发布页；GitHub 发布时间、Latest 状态与发布说明。
- [C5]：CC Switch v4.0.34 Claude 官方多账号指南；直连、隔离终端、Desktop/WSL 限制、令牌和手动切换。
- [C6]：CC Switch v4.0.34 隐私过滤源码；[默认配置](https://github.com/farion1231/cc-switch/blob/v4.0.34/src-tauri/src/proxy/types.rs)，用于确认不可逆替换与默认关闭。
- [C7]：CC Switch v4.0.34 MIT 许可。

[M1]: https://github.com/yetone/magpie/blob/81c666e1c23ec2ca550bd404a0d7891a862569c6/README.md
[M2]: https://github.com/yetone/magpie/blob/81c666e1c23ec2ca550bd404a0d7891a862569c6/docs/reference.md
[M3]: https://github.com/yetone/magpie-releases/releases/tag/v0.1.4394
[M4]: https://usemagpie.ai/api/latest?channel=stable
[M5]: https://usemagpie.ai/api/latest?channel=daily
[M6]: https://usemagpie.ai/docs/intent
[M7]: https://usemagpie.ai/docs/privacy
[M8]: https://github.com/yetone/magpie/blob/81c666e1c23ec2ca550bd404a0d7891a862569c6/go.mod
[M9]: https://github.com/yetone/magpie/blob/81c666e1c23ec2ca550bd404a0d7891a862569c6/AGENTS.md
[C1]: https://github.com/farion1231/cc-switch/blob/v4.0.34/README.md
[C2]: https://github.com/farion1231/cc-switch/blob/v4.0.34/CHANGELOG.md
[C3]: https://github.com/farion1231/cc-switch/blob/v4.0.34/src/config/appSpecs.ts
[C4]: https://github.com/farion1231/cc-switch/releases/tag/v4.0.34
[C5]: https://github.com/farion1231/cc-switch/blob/v4.0.34/docs/guides/claude-official-accounts-guide-zh.md
[C6]: https://github.com/farion1231/cc-switch/blob/v4.0.34/src-tauri/src/proxy/privacy_filter/mod.rs
[C7]: https://github.com/farion1231/cc-switch/blob/v4.0.34/LICENSE
