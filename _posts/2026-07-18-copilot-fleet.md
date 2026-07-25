---
layout:     post
title:      VS Code 多 Agent 名词地图：Local Agent、Subagent、/fleet 与 Agents window
subtitle:   VS Code 与 Copilot CLI 多 Agent 工作方式指南
date:       2026-07-18
author:     Bruce Wong
header-img: img/brett-jordan-M3cxjDNiLlQ-unsplash.jpg
catalog: true
tags:
    - AI Coding
    - Copilot
    - AI
---

VS Code 有 Agent Mode、Subagent 和 Agents window，Copilot CLI 又有 `/fleet`。这些名称看起来都在做“多 Agent”，但它们并不处于同一个层级。

更容易被忽略的是，Copilot CLI 是一种 Agent runtime，不只存在于独立终端中：VS Code 的 Chat view 和 Agents window 也可以启动、监控和管理 Copilot CLI 会话。因此，“VS Code 还是 CLI”本身就不是一个严格成立的二选一问题。

本文基于 2026 年 7 月的产品状态。具体支持范围、模型和并发限制请以官方文档为准。

---

## 一、先建立三个层级

这些概念之所以容易混淆，是因为我们常把三个维度混在一起：

| 维度 | 它回答的问题 | 例子 |
|------|--------------|------|
| **交互界面** | 我在哪里查看和控制 Agent？ | VS Code Chat view、Agents window、CLI 终端 |
| **Agent runtime** | Agent 在哪里、以什么方式运行？ | VS Code Local Agent、Copilot CLI、Copilot Cloud |
| **编排方式** | 一个任务如何交给多个 Agent？ | Subagent 委派、CLI `/fleet`、多个独立会话 |

这三个维度可以组合。例如，你可以：

- 在 VS Code Chat view 中使用 Local Agent，并让它调用多个 Subagent
- 在 VS Code Chat view 中启动后台 Copilot CLI 会话，并在该会话中使用 `/fleet`
- 在终端中直接运行 Copilot CLI，然后使用 `/fleet`
- 在 Agents window 中管理不同 workspace 的多个 Copilot CLI、Copilot Cloud 或 Claude agent 会话

所以，真正需要区分的不是“VS Code 还是 CLI”，而是：

1. 使用哪个界面观察和控制 Agent
2. 使用 Local Agent 还是 Copilot CLI 等 runtime
3. 使用按需 Subagent 委派、显式 Fleet 编排，还是多个独立顶层会话

这两种 runtime 还可以接力：你可以先在 VS Code 中与 Local Agent 讨论需求或完成计划，再把完整会话历史和上下文交给 Copilot CLI，让它在后台继续实施。

---

## 二、四个概念分别是什么？

### 2.1 VS Code Local Agent：在编辑器中交互式完成任务

在 VS Code Chat view 中选择 Agent 后，Local Agent 可以围绕一个目标反复执行：

```text
理解需求
  ↓
读取与搜索代码
  ↓
修改文件、运行命令
  ↓
检查结果并继续修正
  ↓
返回最终结果
```

这通常被称为 Agent Mode。它强调的是 **Agent 可以使用工具自主执行多步任务**，并不意味着每次都会启动多个 Agent。

Local Agent 适合需要实时反馈的工作，例如：

- 需求还不完全清楚，需要边讨论边实现
- 需要利用当前编辑器中的文件、选区、诊断和测试结果
- 希望随时查看改动并调整方向
- 需要使用 VS Code 扩展提供的工具或 MCP server

### 2.2 VS Code Subagent：主代理按需委派

当 Local Agent 认为某个子任务适合隔离处理时，可以调用内置的 `agent/runSubagent` 工具。这里的 `agent/runSubagent` 是工具标识，不是用户在 Chat 输入框中执行的命令；使用前需要确保该工具已启用。

```text
用户
  ↓
VS Code 主代理
  ├── Subagent A：分析安全风险 ──┐
  ├── Subagent B：分析性能问题 ──┼── 返回各自结果
  └── Subagent C：分析测试缺口 ──┘
  ↓
主代理综合结论
```

每个 Subagent 使用独立上下文，只接收完成子任务所需的信息。主会话上下文通常只接收 Subagent 返回的结果或摘要，避免把大量搜索过程和中间信息塞进主会话；用户仍可以展开工具调用，查看传给 Subagent 的 prompt、工具调用过程和返回结果。

Subagent 通常由主代理自主触发。用户可以在 prompt 中提示或请求主代理采用独立、并行的委派方式，但这不是一条强制调用语法；最终是否调用 Subagent、调用多少个，仍由主代理决定。

```markdown
请使用三个独立的 subagent 并行分析：
1. 身份验证相关的安全风险
2. 错误处理的一致性
3. 单元测试覆盖缺口

等待所有分析完成后，将结果合并成一份按优先级排序的报告。
```

### 2.3 Copilot CLI `/fleet`：显式启动批量并行编排

`/fleet` 是 Copilot CLI 中由用户显式输入的 slash command。它告诉主代理：“请分析这个任务是否适合拆分，并把适合的部分交给多个 Subagent 并行完成。”

```bash
/fleet Refactor each SDK package independently, run its tests, and summarize the changes
```

用户显式触发了 `/fleet`，但不代表用户必须亲自完成所有任务拆分，也不保证一定会创建多个 Subagent。主代理仍会分析 prompt 和任务依赖，判断哪些部分适合并行，并负责调度实际需要的 Subagent。

Fleet 由主代理协调子任务及其依赖；Copilot CLI 的 `/tasks` 用于查看当前会话中的后台任务，包括 Fleet 交给 Subagent 的任务。

```text
总体目标
  ↓
主代理分析任务及依赖
  ├── 重构 auth package  ── Subagent A
  ├── 重构 api package   ── Subagent B
  └── 重构 utils package ── Subagent C
  ↓
通过 /tasks 查看后台任务、进入详情或终止任务
  ↓
父会话汇总结果
```

### 2.4 Agents window：多会话控制台

Agents window 是 VS Code 提供的 agent-first UI。它与传统 Chat view 的侧重点不同：

- **Chat view** 是 code-first：围绕当前打开的 workspace 和代码工作
- **Agents window** 是 agent-first：跨 workspace 创建、查看和管理多个 Agent 会话

目前，Agents window 支持 Copilot CLI、Copilot Cloud 和 Claude agent 会话，并不泛指所有本地或第三方 Agent。它解决的是“如何同时观察多个顶层会话”，而不是定义一种新的 Subagent 编排算法。

这里还要区分两种并行：

- **多个独立会话并行**：你分别启动多个顶层任务，每个会话有自己的目标
- **一个会话内部的 Subagent 并行**：主代理为了同一个目标委派多个子任务

两者都叫“多 Agent”，但管理边界和结果汇总方式不同。

---

## 三、一张表总结它们的层级

| 名称 | 它本质上是什么 | 主要解决的问题 |
|------|----------------|----------------|
| **VS Code Local Agent** | 在编辑器内运行的交互式 Agent runtime | 贴着代码讨论、实现和迭代 |
| **VS Code Subagent** | Local Agent 通过内部工具进行的上下文隔离和任务委派 | 研究、分析、多视角评审 |
| **Copilot CLI `/fleet`** | Copilot CLI runtime 中基于多个 Subagent 的显式批量编排 | 计划清晰、边界独立的后台并行任务 |
| **VS Code Agents window** | 跨 workspace 和 Agent 类型的会话管理 UI | 同时管理多个顶层会话 |

因此，它们不是简单的竞争关系：Local Agent 负责“在编辑器中交互式干活”，Subagent 负责“让主代理按需委派”，`/fleet` 负责“显式启动批量编排”，Agents window 负责“统一查看和管理会话”。

真正重要的第一步不是判断哪个功能更强，而是先确认自己正在选择 UI、Agent runtime，还是 orchestration。

下一篇将比较 VS Code Subagent 与 Copilot CLI `/fleet` 的默认模型、Custom Agent、嵌套、并发和隔离方式，并给出实际决策树。

---

**You can outsource your thinking, but you cannot outsource your understanding.**

## 官方参考资料

1. **VS Code Agent 与界面**
   - [Build with agents in VS Code](https://code.visualstudio.com/docs/agents/overview)
   - [Local agents in Visual Studio Code](https://code.visualstudio.com/docs/agents/agent-types/local-agents)
   - [Copilot CLI sessions in Visual Studio Code](https://code.visualstudio.com/docs/agents/agent-types/copilot-cli)
   - [Use the Agents window](https://code.visualstudio.com/docs/agents/agents-window)

2. **VS Code Subagent**
   - [Subagents in Visual Studio Code](https://code.visualstudio.com/docs/agents/subagents)
   - [Agents concepts](https://code.visualstudio.com/docs/agents/concepts/agents)

3. **Copilot CLI `/fleet`**
   - [Running tasks in parallel with the `/fleet` command](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)


