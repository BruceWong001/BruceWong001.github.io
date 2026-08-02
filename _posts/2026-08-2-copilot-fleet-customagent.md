---
layout:     post
title:      Fleet + Custom Agents：如何把专业角色绑定到并行任务
subtitle:
date:       2026-08-02
author:     Bruce Wong
header-img: img/IMG_1189.webp
catalog: true
tags:
    - AI Coding
    - Copilot
    - AI
---

> Fleet 负责拆分与调度，Custom Agent 负责专业能力；要让两者真正配合，关键不是只创建几个 Agent 文件，而是显式绑定每一条工作线。

先说结论：在本次演示是在 GitHub Copilot CLI 中，稳定工作的配置是显式声明 `user-invocable: true` 和 `infer: true`，同时不设置 `disable-model-invocation`。随后再在 `/fleet` Prompt 中使用 `@CUSTOM-AGENT-NAME`，把具体工作线绑定给指定角色。

## 视频如下：

---
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1kG3Q6LEyj&cid=40515602906&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen
  title="用 /fleet 并行构建 FeedHub RSS 阅读器">
</iframe>

---

> **视频勘误：**这段视频采用分段录制。前段展示 Agent profile 时仍保留了 `disable-model-invocation: true`；准备实际执行 Fleet 任务时，我们才通过复测发现需要移除它，并增加 `user-invocable: true`、`infer: true`。由于没有重新录制前段画面，视频中展示的 profile 与后续实际执行配置并不完全一致，请以本文配置为准。

## 视频里的任务图，比功能本身更重要

视频定义了三个项目级 Custom Agents：后端工程师、前端工程师和只读 Reviewer。它们没有同时无序启动，而是按下面的任务图工作：

```text
                  ┌─ @feedhub-backend ──┐
固定接口与业务规则 ──┤                     ├─ @feedhub-reviewer ── 主 Agent 验收
                  └─ @feedhub-frontend ─┘
```

后端和前端拥有不同文件范围，可以并行；Reviewer 依赖两侧的集成结果，必须等待；最后的完整测试、构建和结果汇总仍由主 Agent 完成。

录屏中可以看到三个关键证据：

- `/agent` 列出了三个仓库级 Custom Agents，说明配置已被 CLI 加载；
- `/fleet` 启动后，前后端任务同时进入后台执行，`/tasks` 中显示的执行者正是指定的 Agent；
- 两个实现任务结束后才启动 Reviewer，最终主 Agent 报告 49 个后端测试通过，前端 build 和 lint 完成，并继续做真实页面验证。

这些结果只能说明本次演示环境中的执行成功，但它们足以展示一个可复用的分工模式：**并行实现、串行审查、集中验收**。

## Fleet 默认会调度，但不会天然理解你的业务分工

GitHub 对 `/fleet` 的定义是：主 Copilot Agent 分析目标，拆成较小任务，判断依赖，并尽可能交给多个 subagents 并行执行。每个 subagent 有独立上下文，主 Agent 负责整体编排。[GitHub Docs：Running tasks in parallel with `/fleet`](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)

如果仓库中存在 Custom Agents，Fleet **可能**根据任务和 `description` 选择它们；但官方文档同时明确指出，如果你已经知道某项工作最适合哪个 Agent，应在 Prompt 中使用 `@CUSTOM-AGENT-NAME` 指定。[GitHub Docs：Fleet specialization](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet#benefits-of-using-fleet)

这里有三个容易混在一起的动作：

| 动作 | 解决的问题 | 关键配置 |
| --- | --- | --- |
| 定义 Custom Agent | 它懂什么、能做什么、不能做什么 | `.github/agents/*.agent.md` |
| 允许 Fleet 调用 Agent | 是否允许主 Agent 自动委派给该角色 | `infer: true` |
| 绑定 Fleet 工作线 | 这一次任务究竟交给谁 | `/fleet` Prompt 中的 `@agent-id` |

因此，如果目标是避免 Fleet 为关键工作线自动分配通用 Agent，不能只写一个很准确的 `description`。在当前 CLI 实践中，更稳妥的组合是：**让 Custom Agent 保持可委派状态，同时在 Prompt 中显式绑定任务。**

## 第一步：让 CLI 正确发现 Custom Agent

项目专用 Agent 应放在仓库的 `.github/agents/` 下，并使用 `.agent.md` 扩展名。例如：

```text
.github/agents/
├── backend-engineer.agent.md
├── frontend-engineer.agent.md
└── integration-reviewer.agent.md
```

GitHub 官方建议文件名使用小写字母和连字符，便于在命令行中稳定引用。创建后需要重启 CLI，再输入 `/agent` 检查是否已加载。[GitHub Docs：Creating and using custom agents for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli)

需要注意，官方文档说明 `/agent` 列表不包含 CLI 的默认 Agents。因此，视频里只显示三个自定义角色，只能证明它们已经加载，不能证明通用 Agent 已被关闭。

这里还要区分显示名和 Agent ID：

```yaml
name: Backend Engineer
```

`name` 是界面中的显示名称；在 Prompt 中使用的稳定 ID 通常来自文件名。因此，`backend-engineer.agent.md` 应使用 `@backend-engineer`。视频里显示的是 `FeedHub Backend`，实际引用的是 `@feedhub-backend`，也是这个原因。

## 第二步：允许 Fleet 调用，同时保留手动选择

一个适合 Fleet 使用的实现类 Agent，可以从下面的最小配置开始：

```markdown
---
name: Backend Engineer
description: Implements this repository's API, service, persistence, and backend tests.
user-invocable: true
infer: true
tools: [read, search, edit, execute]
---

You are the backend engineer for this repository.

Before changing code, read `.github/copilot-instructions.md`
and `docs/ARCHITECTURE.md`.

Ownership:
- You may modify only `backend/app/**` and `backend/tests/**`.
- Never modify frontend files or another agent's files.

Validation:
- Run focused tests first, then the complete backend test suite.
- Report changed files, commands run, and integration risks.
```

为了让调用意图更清楚，建议显式写出：

```yaml
user-invocable: true
infer: true
```

`user-invocable: true` 让 Agent 可以被用户选择和显式引用；`infer: true` 允许主 Agent 将任务自动委派给它。二者默认值都是 `true`，所以并非每次都必须显式书写；这里保留它们，是为了让团队能直接看懂调用策略，也便于复现本次验证结果。当前 GitHub Copilot CLI command reference 仍把 `infer` 定义为“允许主 Agent 自动委派”。[GitHub Docs：Custom agent frontmatter fields](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#custom-agent-frontmatter-fields)

这里存在一个需要特别注意的版本差异。GitHub 的统一 Custom Agents 配置页说明，`disable-model-invocation: true` 会禁止模型自动使用该 Agent，并且与 `infer` 同时出现时拥有更高优先级。[GitHub Docs：Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration) 在本次 Copilot CLI v1.0.77 复测中，只要保留 `disable-model-invocation: true`，Fleet 就无法按预期调用这些 Custom Agents；移除后，指定角色能够正常参与调度。

因此，面向 Copilot CLI `/fleet` 的 Agent 配置不要混用这两个方向相反的字段。本次验证采用 `infer: true` 和 `user-invocable: true`，完全省略 `disable-model-invocation`。这解决的是“Agent 是否处于可调用状态”；下一步的 `@agent-id` 负责表达“这项工作应交给谁”。

## 第三步：在 Fleet Prompt 中把任务绑定到 Agent

下面是一份可以替换业务内容后复用的 Prompt：

```text
/fleet Implement the feature described in docs/FEATURE.md.

Run @backend-engineer and @frontend-engineer in parallel.
After both implementation agents finish, run @integration-reviewer
against the integrated result.
Use these named custom agents for the three named tracks.
After dispatch, report the actual agent ID used for each track.

Fixed contract:
- Define the API path, request fields, response shape, and error behavior here.

Ownership:
- @backend-engineer owns backend/app/** and backend/tests/** only.
- @frontend-engineer owns frontend/src/** only.
- @integration-reviewer is read-only and starts only after both are complete.

Done means:
- the main agent runs backend tests successfully;
- the main agent runs frontend build, type-check, and lint successfully;
- report changed files, validation results, and remaining risks.
```

这段 Prompt 做了四件事：

1. 用 `@agent-id` 指定每条工作线的执行者；
2. 用 `in parallel` 和 `after both finish` 描述并行与依赖；
3. 用 ownership 避免多个 Agent 争抢同一文件；
4. 要求报告实际 Agent ID，并把最终质量门禁留给主 Agent。

如果 Prompt 只写“使用我的自定义 Agent 完成任务”，Fleet 仍要猜测任务如何拆分、哪个 Agent 对应哪一部分，以及哪些工作可以并行。显式写出映射和依赖，才是在设计可执行的任务图。不过，Prompt 表达的是调度意图，不是硬性的权限策略；执行后仍要由用户输入 `/tasks`，核对实际使用的 Agent。

## 专业 Agent 不只是换一个角色名称

“你是一名资深后端工程师”只能提供很弱的角色暗示。真正提高专业领域适应能力的，是把团队平时依赖的隐性规则写进 Agent：

| 应写入的内容 | 实际作用 |
| --- | --- |
| 必读上下文 | 指向架构、编码规范、领域词汇和接口文档 |
| Ownership | 明确允许修改和禁止修改的目录或文件 |
| 业务不变量 | 固化事务、权限、兼容性、数据一致性等规则 |
| 工具范围 | 让实现者可编辑执行，让 Reviewer 只能读取搜索 |
| 验收标准 | 规定测试、构建、lint、smoke test 和失败处理 |
| 汇报格式 | 要求返回变更文件、执行命令、风险和验证缺口 |

视频中的 Reviewer 是一个很好的例子。它使用：

```yaml
tools: [read, search]
```

这让它可以检查前后端契约，却不能顺手修改代码或运行命令。实现和审查因此形成了真实的能力差异，而不是三个名字不同、权限相同的通用 Agent。官方配置参考也说明，`tools` 未设置时默认开放全部工具，因此 Reviewer 这类角色尤其需要显式收紧。[GitHub Docs：Custom agent tools](https://docs.github.com/en/copilot/reference/custom-agents-configuration#tools)

不过，`tools` 只控制可用工具，并不是目录级文件锁。`backend/**` 这样的 Ownership 仍属于指令约束，主 Agent 最后要通过 changed-file list 或 diff 检查是否越界。

## 还需要注意三个 Fleet 风险

第一，subagents 共享文件系统，但没有文件锁。两个 Agent 同时修改同一个文件时，后完成的修改可能静默覆盖前者。因此并行工作必须尽量拥有互斥的文件边界，公共入口文件留给主 Agent 集成，或显式改为串行。[GitHub Blog：Avoiding common `/fleet` pitfalls](https://github.blog/ai-and-ml/github-copilot/run-multiple-agents-at-once-with-fleet-in-copilot-cli/#avoiding-common-pitfalls)

第二，subagent 看不到主 Agent 的完整对话历史。重要规则要直接写进 Fleet Prompt，或者让 Agent 读取仓库内的规范文件。不要假设“前面已经聊过，它应该知道”。

第三，未绑定的任务仍由 Fleet 判断如何执行。关键业务工作线要逐一写明 `@agent-id`；探索、收尾和跨模块验收则可以保留给主协调 Agent。使用 `/tasks` 打开后台任务详情，核对显示的 Agent 名称、并行关系和阻塞状态，而不是只看最终回答。

## 什么时候不值得使用这套方案

如果任务只修改一个文件、调查路径严格串行，或者多个角色必须频繁改同一处共享状态，普通单 Agent 往往更简单。Fleet 有拆分、调度、上下文和 AI Credits 成本；官方也提醒，只有能分解成相对独立子任务的工作，才真正适合并行。[GitHub Docs：When to use `/fleet`](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet#when-should-you-use-fleet)

可以先从一个自然形成三条工作线的需求开始：两个互不重叠的实现角色，加一个后置 Reviewer。等接口、Ownership 和质量门禁稳定后，再扩展更多专业 Agent。

## 最后：让 Fleet 从“多开 Agent”变成“组织专业团队”

要让 `/fleet` 稳定使用符合业务需求的 Custom Agents，可以按这张清单检查：

1. Agent 文件是否位于 `.github/agents/`，文件名是否便于用 `@id` 引用；
2. 重启 CLI 后，能否通过 `/agent` 看到它；
3. 是否让 `infer` 与 `user-invocable` 保持为 `true`，并移除 `disable-model-invocation`；
4. Fleet Prompt 是否把每条关键工作线绑定到具体 `@agent-id`；
5. Agent 是否写明领域规则、文件所有权、工具范围和验收标准；
6. `/tasks` 中是否真的出现指定 Agent，以及正确的并行和依赖关系；
7. 主 Agent 是否在最后执行跨模块验证，而不是只汇总 subagent 的自我报告。

Fleet 的价值不只是同时运行更多 Agent。它提供的是编排能力；Custom Agents 提供的是可复用的专业能力。把发现、调用策略、任务绑定和验收连起来，Agent 集群才会从“通用劳动力”变成真正适应项目和业务的专业团队。

**You can outsource your thinking, but you cannot outsource your understanding.**

## 官方参考

- [GitHub Docs：Running tasks in parallel with `/fleet`](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)
- [GitHub Docs：Creating and using custom agents for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli)
- [GitHub Docs：Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [GitHub Blog：Run multiple agents at once with `/fleet` in Copilot CLI](https://github.blog/ai-and-ml/github-copilot/run-multiple-agents-at-once-with-fleet-in-copilot-cli/)
