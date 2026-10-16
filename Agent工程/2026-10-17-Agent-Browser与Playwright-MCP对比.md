---
title: "Agent Browser 与 Playwright MCP 对比调研"
date: 2026-10-17
category: "Agent工程"
tags:
  - Agent
  - 浏览器自动化
  - Playwright
  - MCP
  - CLI
  - Coding Agent
---

# Agent Browser 与 Playwright MCP 对比调研

> 调研日期：2026-10-17。本文比较 Vercel Labs 的 `agent-browser` 与 Microsoft 的 `playwright-mcp`，重点回答如何让 Agent 使用浏览器。调研时 npm latest 元数据分别为 `agent-browser` 0.39.1、`@playwright/mcp` 0.0.83；版本、工具数量和客户端集成会变化，正式部署前应锁定版本并复核官方文档。

## 结论

如果这里的 Agent 是 Claude Code、Codex、GitHub Copilot 这类能够运行命令的**编码型 Agent**，现在应在 **Agent Browser** 和 **Playwright CLI** 之间选择，而不是直接拿 Agent Browser 与 Playwright MCP 比较。两者都提供 CLI 和 Agent Skill：

- 需要通用网页操作、内容读取、会话管理和更强的 Agent 友好封装，优先 **Agent Browser**。
- 项目本来就使用 Playwright Test，且需要把浏览器操作直接沉淀为稳定测试、代码生成和失败修复流程，优先 **Playwright CLI + Playwright Test Agents**。

如果 Agent 运行在 Cursor、Claude Desktop 等 **MCP 原生客户端**中，或者需要把浏览器作为一个长期存在、可被多个 Agent 复用的 MCP 工具服务，选 **Playwright MCP** 更直接。它的官方文档、客户端安装路径和 Playwright 生态整合更成熟。

这不是浏览器引擎的二选一：两者都使用 Playwright 驱动浏览器，并主要通过可访问性/结构化快照定位元素，不依赖视觉模型。主要差异是 Agent 与工具之间的接口形态、上下文开销、工具封装粒度和默认工作流。

## Playwright 有没有 CLI 和 Skill

有，而且是 Playwright 官方为 Coding Agent 提供的独立入口。

### Playwright CLI

官方文档当前提供 `playwright-cli`，安装方式是：

```bash
npm install -g @playwright/cli@latest
playwright-cli open https://example.com
playwright-cli snapshot
playwright-cli click e15
```

Playwright 的新版本还支持从主包调用 `npx playwright cli`；项目应以目标版本的官方文档为准，固定版本后再接入 Agent。CLI 的输出是带元素引用的结构化快照，支持导航、点击、输入、截图、浏览器选择、会话查看和 WebMCP 工具发现。官方将它定位为适合 Coding Agent 的低上下文开销工作流。

### Playwright CLI Skills

Playwright CLI 自带面向 Agent 的 Skill 安装：

```bash
playwright-cli install --skills
playwright-cli install --skills -g
```

它向 Claude Code、GitHub Copilot 等 Coding Agent 提供命令说明和工作流上下文；`-g` 可以安装到用户级 Agent Skills 目录。也可以不安装 Skill，让 Agent 通过 `playwright-cli --help` 自行发现命令。

这与 Playwright MCP 不同：Skill 是给能执行 CLI 的 Agent 加上下文的本地说明，MCP 是给客户端注册工具的协议服务，两者可以并存。

### Playwright Test Agents

Playwright 还提供另一组与浏览器 CLI 不同的 Agent 定义：

```bash
npx playwright init-agents --loop=codex
npx playwright init-agents --loop=claude
```

内置角色包括 planner、generator 和 healer，分别负责探索并生成测试计划、把计划转换成 Playwright Test、执行失败用例并尝试修复。它们不是通用网页操作工具，而是围绕 Playwright 测试工程的 Agent 工作流；官方说明这些定义由指令和 MCP 工具组成，Playwright 更新后应重新生成。

## 核心差异

| 维度 | Agent Browser | Playwright MCP |
|---|---|---|
| 首要形态 | 面向 Agent 的 CLI；另有 `agent-browser mcp` | MCP Server，使用 `npx @playwright/mcp@latest` 接入 |
| 典型调用 | `agent-browser open`、`snapshot`、`click @e1` | MCP 的 `browser_navigate`、`browser_snapshot`、`browser_click` 等工具 |
| 页面表示 | 结构化快照，使用 `@ref`；支持交互元素过滤、紧凑快照和增量 diff | 结构化 accessibility snapshot，使用 `eN` refs；支持快照深度/目标限制 |
| Agent 侧成本 | CLI 不需在上下文加载完整工具 schema；官方强调 token-efficient | MCP 工具 schema 和快照直接进入客户端上下文；适合 MCP 工具调用循环 |
| 工作流封装 | 提供 `read`、内容提取、页面 diff、批量命令、会话、网络、cookies 等 Agent 友好命令 | 提供导航、表单、截图、网络、存储、标签页等 Playwright 能力，工具边界更贴近底层 API |
| 客户端覆盖 | 任何能执行本地命令的 Agent；MCP 客户端也可接入 | 任何 MCP 客户端，官方列出 VS Code、Cursor、Windsurf、Claude Desktop 等 |
| 持久状态 | CLI session/profile/state 等方式管理；可按任务隔离 | 支持持久上下文/连接配置，适合 MCP 服务长期运行和复用 |
| 生态与维护 | Vercel Labs，Apache-2.0，`agent-browser` npm 包 | Microsoft，Apache-2.0，Playwright 官方生态，`@playwright/mcp` npm 包 |
| 主要取舍 | Agent 友好能力更多，CLI/Skill 需要在项目中约定安装方式 | 接入标准化、文档和客户端支持强，但原始工具面和快照可能更占上下文 |

## Agent Browser 与 Playwright CLI/Skills 对比

| 维度 | Agent Browser | Playwright CLI + Skills |
|---|---|---|
| 产品定位 | 通用浏览器 Agent 工具链 | Playwright 官方 Coding Agent CLI |
| 安装 | `npm install -g agent-browser`，再 `agent-browser install` | `npm install -g @playwright/cli`，再 `playwright-cli install --skills` |
| Skill 机制 | `npx skills add vercel-labs/agent-browser`；也可用 `agent-browser skills get` 读取与 CLI 版本匹配的内置内容 | `playwright-cli install --skills`，可安装到项目或用户级目录 |
| 快照与引用 | `snapshot -i`、`@e1`、增量 diff、交互元素过滤 | `snapshot`、`eN`、深度/范围限制；输出更贴近 Playwright CLI |
| 通用网页工作流 | 强：`read`、内容提取、session、网络、cookies、批量命令等 | 强：浏览器交互、截图、WebMCP、Playwright 调试和测试衔接 |
| 测试代码闭环 | 可做浏览器回归和 QA，但测试工程能力需自行组织 | 强：与 Playwright Test、codegen、trace、Test Agents 直接衔接 |
| 生态一致性 | 独立工具链，使用 Playwright 后端但命令和 Skill 自成体系 | 与 Playwright Test/Library/MCP 同一官方生态 |
| 适合的 Agent | 通用浏览、资料读取、表单和多站点操作 Agent | 编码、测试生成、测试修复和 Playwright 项目维护 Agent |
| 主要代价 | 需要额外学习独立命令和 Skill 体系 | 通用内容提取和 Agent 专用封装少一些，测试场景优势更明显 |

表中“token 更省”是 Agent Browser 和 Playwright CLI 文档的产品定位描述，不能视为所有网站、模型和快照选项下的固定倍率。实际开销取决于页面大小、是否使用交互元素过滤、快照深度和模型上下文策略。

## Agent Browser 是什么

`agent-browser` 是 Vercel Labs 发布的浏览器自动化 CLI，目标是让 Agent 通过简短命令完成打开页面、快照、点击、填写、读取内容、截图、网络拦截和会话管理。常见流程是：

```bash
agent-browser open https://example.com
agent-browser snapshot -i
agent-browser click @e1
agent-browser fill @e2 "文本"
agent-browser read
```

它的快照为交互元素生成稳定的引用，命令输出可用 `--json`，并支持 `--session` 隔离浏览器状态。`agent-browser mcp` 可以把同一套能力以 stdio MCP 暴露给 MCP 客户端；默认工具配置可只启用核心工具，需要时再启用 network、storage 等工具组。

适合它的原因是 Agent 通常需要把浏览器命令和文件编辑、测试命令混在一个循环里。CLI 的参数是按动作设计的，Agent 可以按需查看帮助或 Skill，而不必让客户端预先加载一大组 MCP 工具定义。

## Playwright MCP 是什么

`playwright-mcp` 是 Microsoft 的 MCP Server，底层使用 Playwright，让 LLM 通过结构化 accessibility snapshot 观察页面，再用 refs 执行导航、点击、输入、选择、拖拽、截图、网络和存储操作。标准接入配置是：

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

它适合 MCP 原生客户端，因为安装后 Agent 通过统一的 `tools/list` 和 `tools/call` 发现、调用工具。官方文档明确把 MCP 定位为适合“持久状态、页面结构迭代和长期 Agent 工作流”的接口；同一文档也把 Playwright CLI 定位为更适合编码 Agent 的低上下文开销方案。

## 按任务选择

| 任务 | 推荐 | 原因 |
|---|---|---|
| 编码 Agent 做网页检索、填表、登录、回归测试 | Agent Browser | CLI 调用短，可与 Shell、代码和测试工具组合；快照可过滤和 diff |
| Cursor/Claude Desktop 中临时操作网页 | Playwright MCP | 配置简单，直接使用客户端的 MCP 工具面 |
| 多个 Agent 共享浏览器服务或长流程会话 | Playwright MCP | MCP 连接模型和持久上下文更自然；需自行管理隔离和生命周期 |
| 需要读取文章正文、提取链接/表格、减少页面噪声 | Agent Browser | 有 `read` 和内容提取等面向 Agent 的命令封装 |
| 需要直接使用 Playwright 生态能力或后续写测试代码 | Playwright MCP，或直接用 Playwright CLI/Library | 官方生态、文档和 API 一致性更强 |
| 不信任目标网站、需要最小权限 | 两者均可，显式配置隔离 | 使用独立 session/context、域名白名单、只读工具和最小权限；工具本身不是安全边界 |

## 给 Agent 的落地建议

1. **通用网页 Agent**：先装 Agent Browser，并在 Skill 中约定“先 `snapshot -i`，再用 ref 操作，页面变化后重新快照”。
2. **Playwright 测试项目**：装 Playwright CLI Skills；需要自动建测试时再运行 `npx playwright init-agents --loop=codex` 或目标 Agent 对应的 loop。
3. **MCP 原生客户端**：接 Playwright MCP；通过配置文件固定版本，不要长期使用未锁定的 `@latest`；大型页面使用 snapshot 的深度或目标限制。
4. 两者都不要把截图当作唯一定位依据。截图适合视觉核查，元素操作优先使用结构化快照和语义 ref。
5. 登录态、cookies、下载文件和网络拦截应按任务建立独立 profile；不要把个人浏览器的完整持久 profile 直接交给通用 Agent。
6. 对真实业务操作增加域名白名单、敏感动作审批、下载目录限制、超时和审计。MCP 的工具描述、`readOnly` 注解或 Agent 提示词都不能替代服务端权限控制。

## 最终建议

对“让一个通用编码 Agent 使用浏览器”这个问题，我建议在两种情况下分别选择：通用网页操作选 **Agent Browser**；已有 Playwright 测试工程、需要生成和修复测试选 **Playwright CLI + Skills**。两者都比直接把完整 MCP 工具面交给编码 Agent 更省上下文。

对“给现有 MCP 客户端增加一个浏览器工具服务器”这个问题，我建议选 **Playwright MCP**，因为连接方式标准、官方文档和客户端安装支持更完整。

如果项目后续需要同时做网页操作和 Playwright 测试开发，应把三种用途分开评估：Agent Browser 负责高频 Agent 操作，Playwright CLI/Skills 负责编码 Agent 的测试流程，Playwright Library/Test 负责可维护的确定性测试。不要把一次性 Agent 操作直接当成长期回归测试代码。

## 资料来源

- [Vercel Labs agent-browser 官方仓库](https://github.com/vercel-labs/agent-browser)
- [Agent Browser CLI 文档](https://vercel-labs-agent-browser.mintlify.app/)
- [Microsoft Playwright MCP 官方仓库](https://github.com/microsoft/playwright-mcp)
- [Playwright MCP 官方文档](https://playwright.dev/docs/getting-started-mcp)
- [Playwright Coding Agents / CLI 文档](https://playwright.dev/docs/getting-started-cli)
- [Playwright Test Agents 文档](https://playwright.dev/docs/test-agents)
- [agent-browser npm 包元数据](https://www.npmjs.com/package/agent-browser)
- [@playwright/mcp npm 包元数据](https://www.npmjs.com/package/@playwright/mcp)

