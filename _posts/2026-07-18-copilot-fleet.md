---
layout:     post
title:      从“哪个更厉害”到“哪个更合适”：多 Agent 入口决策指南
subtitle:
date:       2026-07-18
author:     Bruce Wong
header-img: img/IMG_1189.webp
catalog: true
tags:
    - AI Coding
    - Copilot
    - AI
---

VS Code 有 Agent Mode、Subagent 和新的 Agents window，Copilot CLI 又有 `/fleet`。这些名称看起来都在做“多 Agent”，但它们并不处于同一个层级。本文从使用者视角厘清它们的关系，并回答一个最实际的问题：面对同一个任务，到底应该留在 VS Code，还是打开 Copilot CLI？

---

## 一、先说结论

VS Code Subagent 和 Copilot CLI `/fleet` **不是两套完全无关的技术，也不是同一个功能换了一个界面**。

它们共享相似的多代理工作模式：

1. 一个主代理理解整体目标
2. 主代理把部分工作交给多个子代理
3. 子代理使用独立上下文完成各自任务
4. 子代理把结果返回给主代理
5. 主代理汇总、验证并继续推进

但它们的产品入口和编排方式不同：

- **VS Code Subagent**：主代理在工作过程中按需调用子代理，重点是上下文隔离和灵活委派。
- **Copilot CLI `/fleet`**：用户显式启动并行模式，由 CLI 将计划拆成任务并调度多个子代理，重点是批量执行和任务状态管理。
- **VS Code Agent Mode**：让一个主代理在编辑器里自主读代码、修改文件和运行命令；它本身不等于多代理，但主代理可以进一步调用 Subagent。
- **VS Code Agents window**：管理和观察多个 Agent 会话的 UI；它甚至可以承载 Copilot CLI 会话，因此“使用 VS Code UI”和“使用 Copilot CLI runtime”并不总是二选一。

最简单的选择原则是：

> 需要贴着代码交互、随时调整方向，优先 VS Code；已经有清晰计划，需要把独立任务批量并行执行，优先 Copilot CLI `/fleet`。

---

## 二、为什么这些名字容易混淆？

因为我们常把三个不同维度混在一起：

| 维度 | 它回答的问题 | 例子 |
|------|--------------|------|
| **交互界面** | 我在哪里查看和控制 Agent？ | VS Code Chat view、Agents window、CLI 终端 |
| **代理运行方式** | Agent 在哪里、以什么方式运行？ | VS Code local agent、Copilot CLI、cloud agent |
| **编排方式** | 一个任务如何交给多个 Agent？ | Subagent 委派、CLI `/fleet`、多个独立会话 |

这三个维度可以组合。

例如，你可以：

- 在 VS Code Chat view 中使用本地 Agent，并让它调用多个 Subagent
- 在终端中运行 Copilot CLI，然后使用 `/fleet`
- 在 VS Code Agents window 中启动或查看 Copilot CLI 会话
- 在 Agents window 中同时管理不同 workspace 的多个独立会话

因此，“VS Code 还是 CLI”有时是在选择交互体验，有时是在选择运行时和编排能力。先分清问题属于哪个维度，很多困惑会自然消失。

---

## 三、四个概念分别是什么？

### 3.1 VS Code Agent Mode：一个主代理自主完成任务

在 VS Code Chat view 中选择 Agent 后，主代理可以围绕一个目标反复执行：

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

适合它的场景包括：

- 需求还不完全清楚，需要边讨论边实现
- 需要利用当前编辑器中的文件、选区、错误和测试结果
- 希望随时查看改动并调整方向
- 需要使用 VS Code 扩展提供的工具或 MCP server

### 3.2 VS Code Subagent：主代理按需委派

当主代理认为某个子任务适合隔离处理时，可以调用 `agent/runSubagent`：

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

每个 Subagent 有独立上下文，只把最终结果返回主代理，避免把大量搜索过程和中间信息塞进主会话。多个 Subagent 也可以并行运行。

Subagent 通常由主代理自主触发，但用户可以在 prompt 中明确要求并行分析：

```markdown
请并行运行三个 subagent：
1. 检查身份验证相关的安全风险
2. 检查错误处理的一致性
3. 检查单元测试覆盖缺口

最后将结果合并成一份按优先级排序的报告。
```

Subagent 还可以使用 Custom Agent 中定义的专用指令、工具和模型。默认情况下，Subagent 不能继续创建下一层 Subagent；VS Code 可以通过设置允许嵌套，最大深度为 5。

### 3.3 Copilot CLI `/fleet`：显式启动批量并行编排

`/fleet` 是 Copilot CLI 的交互命令。用户明确告诉 CLI：“请考虑把这个任务交给多个子代理并行完成。”

```bash
/fleet Refactor each SDK package independently, run its tests, and summarize the changes
```

需要注意：用户显式触发了 `/fleet`，但并不代表用户必须亲自完成所有任务拆分。主代理仍会分析 prompt、判断哪些工作可以并行，并负责调度 Subagent。

Fleet 使用显式任务状态协调工作。可以把它简化理解为：

```text
总体目标
  ↓
主代理生成任务列表
  ├── todo-1：重构 auth package  ── Worker A
  ├── todo-2：重构 api package   ── Worker B
  └── todo-3：重构 utils package ── Worker C
  ↓
跟踪 pending / in_progress / done / blocked
  ↓
父会话汇总并验证结果
```

Fleet 可以跟踪任务依赖，但依赖越多，可并行的部分就越少。它最适合可以预先拆分、文件所有权明确、相互等待较少的任务。

### 3.4 VS Code Agents window：多会话控制台

Agents window 是 VS Code 提供的 agent-first UI。它与传统 Chat view 的侧重点不同：

- **Chat view** 是 code-first：围绕当前打开的 workspace 和代码工作
- **Agents window** 是 agent-first：跨 workspace 创建、查看和管理多个 Agent 会话

Agents window 可以管理 Copilot CLI、Copilot cloud 和部分第三方 Agent 会话。它解决的是“如何同时观察多个会话”，而不是定义一种新的 Subagent 编排算法。

这里还要区分两种并行：

- **多个独立会话并行**：你分别启动多个顶层任务，每个会话有自己的目标
- **一个会话内部的 Subagent 并行**：主代理为了同一个目标委派多个子任务

两者都叫“多 Agent”，但管理边界和结果汇总方式不同。

---

## 四、底层技术是一样的吗？

准确答案是：**概念模型相似，产品实现和公开的编排接口不同；不能据此断言它们使用完全相同的内部代码。**

### 相同点

| 能力 | VS Code Subagent | CLI `/fleet` |
|------|------------------|--------------|
| 主代理负责整体目标 | ✅ | ✅ |
| 子代理独立上下文 | ✅ | ✅ |
| 支持多个子代理并行 | ✅ | ✅ |
| 子代理返回结果给父会话 | ✅ | ✅ |
| 支持 Custom Agent | ✅ | ✅ |
| 可以为子任务选择模型 | ✅ | ✅ |

### 不同点

| 维度 | VS Code Subagent | Copilot CLI `/fleet` |
|------|------------------|----------------------|
| **入口** | Agent 在对话中调用 `runSubagent` | 用户显式输入 `/fleet`，或从 Plan Mode 选择 Fleet |
| **主要目的** | 隔离上下文、灵活委派研究和分析 | 将清晰计划拆成独立工作项并批量调度 |
| **调度模型** | 主代理发起一个或多个 Subagent 调用 | 父会话维护 todo、状态和依赖并调度 worker |
| **用户观察方式** | Chat 中的可折叠 Subagent 工具调用 | 终端输出及 `/tasks` 后台任务列表 |
| **嵌套** | 默认关闭，可配置最多 5 层 | 不应与 todo 依赖链混为一谈 |
| **环境优势** | 编辑器上下文、VS Code 工具和扩展 | 终端工作流、显式批量执行和后台任务管理 |

所以更准确的说法不是“两个完全不同的多 Agent 系统”，而是：

> 它们采用相似的主代理—子代理模式，但通过不同产品入口和不同调度方式服务不同的交互习惯。

---

## 五、同一个任务，在两种入口中有什么不同？

假设任务是：检查一个项目的安全、性能、测试和代码质量问题。

### 使用 VS Code

```markdown
请分析当前项目，并行运行四个 subagent，分别检查：
1. 安全风险
2. 性能瓶颈
3. 测试覆盖缺口
4. 代码质量问题

先只输出综合报告，不修改代码。
```

你的体验会更像与 Tech Lead 协作：主代理决定如何调用 Subagent，你可以在 Chat 中展开每个调用、查看结果，然后继续追问或选择要修复的问题。

### 使用 Copilot CLI

```bash
/fleet Review security, performance, test coverage, and code quality in parallel. Do not modify files. Summarize findings by severity.
```

你的体验更像启动一批工作：CLI 将工作交给后台 Subagent，你可以通过 `/tasks` 查看任务、进入某个任务看详情或终止任务，最后由父会话汇总。

在这个只读分析案例里，两种方式都合理。真正决定选择的通常不是“谁的模型更聪明”，而是你希望怎样参与过程：

- 希望边看代码边讨论：VS Code 更顺手
- 希望发出一个清晰批量任务后在终端观察：CLI `/fleet` 更顺手

---

## 六、到底应该选哪个？

### 优先 VS Code Agent Mode + Subagent

当你符合以下情况：

- 日常工作本来就在 VS Code 中
- 任务仍有模糊之处，需要频繁补充上下文
- 需要根据编辑器诊断、选中代码或测试结果即时调整
- 主要目标是研究、评审、探索多个方案
- 希望以可折叠 UI 查看 Subagent 的 prompt、工具调用和结果

### 优先 Copilot CLI `/fleet`

当你符合以下情况：

- 日常工作偏向终端
- 已经有经过确认的实施计划
- 任务能拆成多个边界清晰、互不冲突的工作项
- 希望显式启动批量并行，并使用 `/tasks` 查看后台任务
- 例如每个 worker 分别负责一个 package、module 或独立评审范围

### 优先 VS Code Agents window

当你的主要问题不是“一个任务如何拆分”，而是：

- 想同时跟踪多个顶层任务或多个 workspace
- 想在一个 UI 中管理 Copilot CLI、cloud agent 等不同会话
- 想在 agent-first 与 code-first 工作方式之间切换

### 两者都不该用

以下情况不要为了“多 Agent”而并行：

- 一个 Agent 很快就能完成的小任务
- 后一步必须持续依赖前一步具体输出的强顺序任务
- 多个 worker 会频繁修改相同文件
- 任务边界模糊，需要不断共享最新推理
- 并行后的合并和评审成本可能超过节省的时间

---

## 七、一个更实用的决策树

```text
你是在管理一个任务，还是多个独立会话？
│
├── 多个独立会话 / 多个 workspace
│   └── VS Code Agents window 或 GitHub Mission Control
│
└── 一个复杂任务
    │
    ├── 任务很小、强顺序或会修改相同文件
    │   └── 使用单 Agent
    │
    ├── 需要贴着代码交互、频繁调整
    │   └── VS Code Agent Mode，需要时调用 Subagent
    │
    └── 已有清晰计划，可以拆成独立工作项
        └── Copilot CLI `/fleet`
```

这不是硬性限制。VS Code Subagent 也能处理批量分析，`/fleet` 也能并行研究；决策树表达的是更自然的入口，而不是产品能力的绝对边界。

---

## 八、常见误区

### 误区 1：Agent Mode 就是多 Agent

不是。Agent Mode 首先表示一个 Agent 能自主使用工具完成多步任务。只有当主代理进一步调用 Subagent 时，才形成会话内部的多代理协作。

### 误区 2：`/fleet` 是另一种与 Subagent 无关的技术

不是。`/fleet` 会把任务分配给多个 Subagent。区别在于它提供了更显式的批量并行入口和任务协调方式。

### 误区 3：在 VS Code 中就不能使用 Copilot CLI

不是。新的 Agents window 可以创建和管理 Copilot CLI 会话。UI、运行时和编排方式是可以组合的。

### 误区 4：输入 `/fleet` 后，所有步骤都会并行

不是。主代理仍会判断哪些工作适合交给 Subagent；存在依赖的任务可能等待，不能有效拆分的部分也未必并行。

### 误区 5：并行一定更快、更省

不是。并行有启动、协调、汇总和冲突处理成本；每个 Subagent 还会独立调用模型，可能消耗更多 GitHub AI Credits。准确说法是：**合适的独立任务可能缩短总等待时间。**

### 误区 6：任务依赖等于 Subagent 嵌套

不是。任务依赖表示 B 等待 A；Subagent 嵌套表示一个子代理继续创建下一层子代理。这是两个不同概念。

---

## 九、多 Agent 真正重要的不是“开几个”，而是如何分工

无论使用 VS Code 还是 `/fleet`，都建议遵守以下原则：

1. **每个子任务有明确目标和交付物**
2. **不同 worker 尽量不要修改重叠文件**
3. **把必要上下文写进每个子任务，不要假设子代理知道主会话全部历史**
4. **要求每个 worker 汇报修改内容、验证方式和阻塞项**
5. **最终结果必须由主代理或人统一验证**

一个好的并行 prompt 应明确：

```markdown
- 每个 worker 负责哪些文件或模块
- 哪些文件禁止修改
- 应运行哪些测试
- 最终需要返回什么格式
- 失败或不确定时如何报告
```

如果任务无法清楚回答这些问题，通常说明它还没有准备好进入 Fleet 或大规模并行执行。

---

## 十、总结

| 名称 | 它本质上是什么 | 什么时候优先使用 |
|------|----------------|------------------|
| **VS Code Agent Mode** | 一个主代理在编辑器中自主使用工具 | 贴着代码、交互式开发 |
| **VS Code Subagent** | 主代理按需委派的上下文隔离能力 | 研究、分析、多视角评审 |
| **Copilot CLI `/fleet`** | 基于多个 Subagent 的显式批量编排 | 计划清晰、边界独立的并行任务 |
| **VS Code Agents window** | 跨 workspace 和 Agent 类型的会话管理 UI | 同时管理多个顶层会话 |

它们不是简单的竞争关系：Agent Mode 负责“让主代理干活”，Subagent 负责“让主代理委派”，`/fleet` 负责“显式批量编排”，Agents window 负责“统一查看和管理会话”。

真正的选择不是“哪个功能更高级”，而是：

> 这个任务是否适合并行，以及你希望以多大的粒度参与、观察和控制它。

---

## 官方参考资料

1. **VS Code Agent 与界面**
   - [Build with agents in VS Code](https://code.visualstudio.com/docs/agents/overview)
   - [Local agents in Visual Studio Code](https://code.visualstudio.com/docs/agents/agent-types/local-agents)
   - [Use the Agents window](https://code.visualstudio.com/docs/agents/agents-window)

2. **VS Code Subagent**
   - [Subagents in Visual Studio Code](https://code.visualstudio.com/docs/agents/subagents)
   - [Agents concepts](https://code.visualstudio.com/docs/agents/concepts/agents)

3. **Copilot CLI `/fleet`**
   - [Running tasks in parallel with the `/fleet` command](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)
   - [Speeding up task completion with the `/fleet` command](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/speed-up-task-completion)
   - [Best practices for GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-best-practices)

4. **跨仓库和多会话编排**
   - [How to orchestrate agents using mission control](https://github.blog/ai-and-ml/github-copilot/how-to-orchestrate-agents-using-mission-control/)

