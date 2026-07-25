---
layout:     post
title:      VS Code Subagent 与 Copilot CLI /fleet 实战指南
subtitle:   VS Code 与 Copilot CLI 多 Agent 工作方式指南
date:       2026-07-25
author:     Bruce Wong
header-img: img/brett-jordan-M3cxjDNiLlQ-unsplash.jpg
catalog: true
tags:
    - AI Coding
    - Copilot
    - AI
---

复杂任务不等于多 Agent 任务。真正决定效果的，通常不是启动了多少个 Agent，而是工作能否拆成边界清楚、彼此低冲突、最终可以统一验证的子任务。

> 本文是“VS Code 与 Copilot CLI 多 Agent 工作方式指南”下篇。上篇[《VS Code 多 Agent 名词地图》](https://brucetalk.com/2026/07/18/copilot-fleet/)解释 Local Agent、Subagent、`/fleet` 与 Agents window 的层级；本文不再重复定义，直接讨论怎样拆任务和执行。

本文基于 2026 年 7 月的产品状态。模型、并发、权限和支持范围变化较快，具体限制请以文中的官方链接为准。

---

## 一、先判断任务是否值得并行

在选择 VS Code Subagent 或 Copilot CLI `/fleet` 之前，先回答四个问题：

| 问题 | 适合并行的信号 | 不适合并行的信号 |
|------|----------------|------------------|
| **结果能否独立交付？** | 每个子任务都有单独报告、补丁或测试结果 | 后一步必须持续依赖前一步的具体输出 |
| **输入是否已经稳定？** | 需求、范围和约束已经明确 | 工作过程中需要频繁补充上下文或改变方向 |
| **修改是否低冲突？** | 不同 worker 负责不同模块或只读分析 | 多个 worker 会反复修改同一批文件 |
| **结果能否统一验证？** | 有明确测试、验收标准和汇总负责人 | 合并和复核成本可能超过并行节省的时间 |

只要关键问题的答案是否定的，就应优先使用单 Agent，或先完成前置步骤再顺序推进。多 Agent 更适合任务图中彼此独立的分支，而不是把一条强依赖链硬拆给多个 worker。

因此，正确顺序是：

```text
确认目标 → 画出依赖 → 划分责任边界 → 定义交付物 → 选择执行方式
```

Agent 数量应该由任务结构决定，而不是预先指定“必须启动四个”。

---

## 二、同一个任务，应该怎样拆？

假设目标是检查一个项目的安全、性能、测试和代码质量。下面这种 prompt 看似明确，其实只有分类，没有责任边界：

```markdown
使用多个 Agent 全面检查这个项目。
```

更好的做法是先定义四个只读工作单元：

| Worker | 负责范围 | 交付物 | 禁止事项 |
|--------|----------|--------|----------|
| 安全 | 身份验证、输入验证、凭据和依赖风险 | 带文件位置、影响和修复建议的发现清单 | 不修改文件 |
| 性能 | 热路径、数据库访问、重复计算和资源使用 | 按影响排序的瓶颈与验证方法 | 不做无证据的性能结论 |
| 测试 | 关键流程、边界条件和失败路径 | 测试缺口及建议用例 | 不重写现有测试 |
| 代码质量 | 重复逻辑、错误处理和模块边界 | 可维护性问题及重构优先级 | 不处理纯风格偏好 |

只读评审很适合并行，因为不同 worker 可以独立调查，几乎没有文件冲突。进入实现阶段后，边界要改成具体的 package、module 或文件集合；共享配置、锁文件和公共接口应只交给一个负责人，或放到所有并行任务完成后的整合阶段。

每个子任务至少应该包含以下契约：

```markdown
- 目标：要解决什么问题
- 范围：负责哪些目录、模块或风险类型
- 禁止事项：哪些文件或行为不能碰
- 验证：必须运行哪些测试或检查
- 交付物：返回报告、补丁还是提交
- 异常处理：失败或不确定时如何汇报
```

如果这些内容写不清楚，任务通常还没有准备好进入大规模并行。

对于既要分析又要修改代码的任务，更稳妥的方法是分成两个阶段。第一阶段让各 worker 只读调查，返回证据、影响和建议；主代理去重并确认优先级后，第二阶段再按照不重叠的文件或模块分配实现任务。涉及公共接口、共享配置或锁文件的修改，应指定单一负责人，其他 worker 等待该变更完成后再继续。这样既保留并行调查的速度，也避免多个 Agent 基于不同假设同时改动同一处代码。

---

## 三、任务拆好以后，再选择执行方式

同一份任务契约可以交给不同执行入口。选择标准不是谁“更高级”，而是你希望怎样参与过程：

| 工作状态 | 优先选择 | 原因 |
|----------|----------|------|
| 需求仍然模糊，需要边看代码边调整 | VS Code Local Agent + Subagent | 能结合编辑器上下文持续追问和转向 |
| 计划已经确认，可以批量后台推进 | Copilot CLI `/fleet` | 显式启动并行编排，并通过 `/tasks` 查看后台任务 |
| 同时管理多个顶层任务或 workspace | VS Code Agents window | 统一观察多个独立会话 |
| 任务很小、强顺序或修改高度重叠 | 单 Agent | 避免额外的协调、汇总和冲突成本 |

在 VS Code 中，可以提示 Local Agent 使用多个独立 Subagent，并要求它最终综合结果；这不是直接执行 `agent/runSubagent` 的命令，主代理仍会决定实际委派方式。

在 Copilot CLI 中，则可以显式输入：

```bash
/fleet Review security, performance, test coverage, and code quality in parallel. Do not modify files. Summarize findings by severity.
```

`/fleet` 也不意味着所有步骤都会并行。主代理仍会分析子任务及其依赖，只把适合的部分交给 Subagent。[GitHub 官方文档](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)明确说明，强顺序任务不会因为使用 `/fleet` 自动获得并行收益。

---

## 四、执行前还要算清模型、并发和隔离

### 模型：不要假设所有 Subagent 都与主会话相同

VS Code Subagent 默认继承主会话的 Agent、模型和工具。主代理或 Custom Agent 可以指定其他模型，但不能超过主模型的成本等级；请求更昂贵的模型时会回退到主模型。[VS Code Subagent 文档](https://code.visualstudio.com/docs/agents/subagents)列出了模型选择顺序和成本等级限制。

Fleet 创建的 Subagent 默认使用低成本模型，但可以在 prompt 中为特定工作指定其他可用模型，也可以使用带有模型配置的 Custom Agent。GitHub 当前没有为 `/fleet` 记录与 VS Code 相同的“不得超过主模型成本等级”限制。[Fleet 文档](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)给出的示例允许不同子任务使用不同模型。

有一个例外：如果 Copilot CLI 主会话使用 `Auto`，[CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference) 说明 Custom Agent 的 `model` 字段会被忽略，Subagent 会继承 Auto 最终解析出的会话模型。模型是否可用仍受套餐、组织策略和仓库 allowlist 影响，具体范围见 [Supported AI models](https://docs.github.com/en/copilot/reference/ai-models/supported-models)。

### 并发：请求四个，不代表四个同时运行

VS Code 默认禁止 Subagent 继续嵌套；启用 `chat.subagents.allowInvocationsFromSubagents` 后，最大嵌套深度为 5。Copilot CLI 当前记录的默认最大深度为 6，并发上限随套餐从 2 到 32；只有 usage-based billing 用户可以通过 `subagents.maxDepth` 和 `subagents.maxConcurrency` 覆盖相应限制。[CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)记录了这些数字和适用条件。

并行也不一定更省。每个 Subagent 都可能独立调用模型，增加协调、汇总和 GitHub AI Credits 消耗。更准确的说法是：边界合适的独立任务可能缩短总等待时间，但通常会增加模型交互次数。

### 隔离：worktree 隔离的是会话，不是每个 Fleet worker

从 VS Code 创建 Copilot CLI 会话时，可以选择 Worktree 或 Folder isolation。Worktree 模式把会话修改放在独立 Git worktree 中；Folder 模式直接修改当前 workspace。这是顶层会话的隔离方式，不表示 `/fleet` 会为每个 Subagent 自动创建独立 worktree。

Worktree isolation 会自动使用 Bypass Approvals，且不能切换权限级别。[Copilot CLI 会话文档](https://code.visualstudio.com/docs/agents/agent-types/copilot-cli)说明了两种隔离模式和权限差异。Worktree 主要隔离 Git 工作副本，不等于完整的操作系统或网络安全沙箱；需要限制文件系统和网络访问时，还应使用 [Agent sandboxing](https://code.visualstudio.com/docs/agents/approvals)。

---

## 五、一个从任务出发的决策树

```text
你是在管理多个独立顶层任务吗？
│
├── 是
│   └── 使用 VS Code Agents window 管理多个会话
│
└── 否，是一个复杂任务
    │
    ├── 无法拆成边界清晰、低冲突的工作单元
    │   └── 使用单 Agent，或先顺序完成前置步骤
    │
    └── 可以拆分
        │
        ├── 需要贴着代码交互、频繁调整
        │   └── VS Code Local Agent + Subagent
        │
        └── 计划明确，希望批量后台推进
            └── Copilot CLI `/fleet`
```

这不是产品能力的绝对边界，而是更自然的工作方式。VS Code Subagent 也能批量分析，`/fleet` 也能处理研究任务；真正的分界始终是任务依赖和用户参与方式。

---

## 六、执行与验收清单

启动并行任务前，确认：

1. 每个 worker 都有明确目标、范围和交付物
2. 不同 worker 不会频繁修改相同文件
3. 必要上下文已经写入各自的子任务
4. 测试、检查命令和成功标准已经定义
5. 失败、证据不足或超出范围时必须明确报告

所有 worker 完成后，还需要由主代理或人统一完成：

- 去重和校准发现的严重程度
- 检查不同修改之间的接口和行为是否一致
- 运行项目级测试，而不只相信各子任务的局部验证
- 审查没有证据支持的推断
- 决定哪些结果应该合并、返工或放弃

多 Agent 可以外包执行，但不能外包最终理解。先把任务拆对，再选择 Subagent、`/fleet` 或单 Agent，通常比追求更多并发更重要。

---

**You can outsource your thinking, but you cannot outsource your understanding.**

## 官方参考资料

1. **VS Code Subagent**
   - [Subagents in Visual Studio Code](https://code.visualstudio.com/docs/agents/subagents)
   - [Agents concepts](https://code.visualstudio.com/docs/agents/concepts/agents)

2. **Copilot CLI 与 VS Code**
   - [Copilot CLI sessions in Visual Studio Code](https://code.visualstudio.com/docs/agents/agent-types/copilot-cli)
   - [Use the Agents window](https://code.visualstudio.com/docs/agents/agents-window)
   - [Manage approvals and permissions](https://code.visualstudio.com/docs/agents/approvals)

3. **Copilot CLI `/fleet`**
   - [Running tasks in parallel with the `/fleet` command](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)
   - [Speeding up task completion with the `/fleet` command](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/speed-up-task-completion)
   - [GitHub Copilot CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)

4. **模型可用范围**
   - [Supported AI models in GitHub Copilot](https://docs.github.com/en/copilot/reference/ai-models/supported-models)
