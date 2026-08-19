---
title: "Codex 在 VS Code 中的使用能力调研"
date: 2026-08-20
category: "Agent工程"
tags:
  - Codex
  - VS Code
  - IDE扩展
  - AI编程
  - 编辑器上下文
---

# Codex 在 VS Code 中的使用能力调研

> 调研日期：2026-08-20。本文以 OpenAI 官方 Codex IDE 扩展文档为主要依据，结论针对文档当前明确说明的能力。

## 结论

**可以。Codex 能够在 Visual Studio Code（VS Code）中使用。** OpenAI 官方提供 Codex IDE 扩展，文档将 Visual Studio Code 列为支持的编辑器，并提供 VS Code 扩展安装入口。安装并登录后，可以在 VS Code 侧边栏中打开 Codex，与当前项目和代码一起工作。

## 在 VS Code 中可以做什么

### 使用编辑器上下文

可以把当前打开的文件、代码选区和最近的 Codex 对话带入提示词。这样可以直接让 Codex 解释代码、定位问题或修改指定代码，减少重复粘贴上下文。

### 在编辑器内审查修改

Codex 会给出修改摘要并展示聚焦的差异，用户可以在 VS Code 中检查修改内容，只保留需要的改动，并继续追问后续修改。

### 委托较大的任务

短平快的迭代可以在本地编辑器中完成；当任务需要更多时间和空间时，可以从 IDE 连接 Codex web 继续处理，之后回到编辑器查看结果。

## 安装和打开方式

1. 在 VS Code 中安装官方扩展：[OpenAI Codex 扩展](https://marketplace.visualstudio.com/items?itemName=openai.chatgpt)。
2. 登录 Codex。
3. 打开项目，在 VS Code 中选择 Codex 图标。
4. 如果侧边栏没有显示图标，打开命令面板并执行 `Codex: Open Codex Sidebar`。
5. 在聊天框中提出代码解释、修改或调试请求。

官方文档还列出了 VS Code Insiders，以及使用相同扩展机制的 Cursor、Windsurf。Xcode 和 JetBrains IDE 则是各自的独立集成方式。

## 使用判断

| 问题 | 结论 |
|---|---|
| 能否在 VS Code 使用 Codex | 可以，官方提供 IDE 扩展 |
| 是否能读取当前编辑器上下文 | 可以，包括打开文件、代码选区和最近对话 |
| 是否能在编辑器内查看修改 | 可以，可查看摘要和差异后决定保留哪些改动 |
| 是否能处理较大任务 | 可以从 IDE 委托给 Codex web，再回到编辑器查看结果 |
| 是否需要安装扩展和登录 | 官方快速开始步骤要求安装或启用 Codex 并登录 |

## 边界与待核查项

- 本次官方 IDE 文档确认了 VS Code 的支持、安装入口和主要工作流，但没有在该页面给出最低 VS Code 版本、操作系统矩阵或具体账号套餐要求。
- 因此，实际部署前应以 VS Code 扩展市场的当前要求、组织管理员策略和账号可用性为准，不应仅凭本页推断所有账号都具备相同的 Codex 权限。

## 资料来源

- [OpenAI 官方文档：Codex IDE extension](https://developers.openai.com/codex/ide)
- [Visual Studio Marketplace：OpenAI Codex 扩展](https://marketplace.visualstudio.com/items?itemName=openai.chatgpt)
