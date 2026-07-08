---
layout:     post
title:      从一个 Agent 到一支 AI 开发团队：GitHub Copilot CLI '/fleet' 实战
subtitle:
date:       2026-07-08
author:     Bruce Wong
header-img: img/IMG_1189.webp
catalog: true
tags:
    - AI Coding
    - Copilot
    - AI
---

过去使用 Coding Agent，通常是把任务交给一个 Agent：读需求、改代码、跑测试，再给出结果。面对同时涉及后端、前端、测试和文档的完整需求时，它仍要沿一条时间线逐项执行。

GitHub Copilot CLI 的 `/fleet` 改变的不只是“同时多开几个 Agent”。主 Agent 会转为编排者：分析目标、拆分任务、识别依赖，把当前可执行的工作交给多个 subagent 并行处理，最后汇总和验证结果。GitHub 对它的官方定义也是：将复杂请求拆成较小任务并并行运行，以提升效率和吞吐量。[GitHub Docs：Running tasks in parallel with `/fleet`](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)

## 视频里发生了什么
---
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1WoMG6wEvn&cid=39767576050&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen
  title="用 /fleet 并行构建 FeedHub RSS 阅读器">
</iframe>

---
演示目标是从 PRD 和架构文档出发，构建一个 FeedHub RSS 阅读器：后端使用 FastAPI + SQLite，前端使用 React + TypeScript。

视频没有直接运行 `/fleet`，而是先进入 Plan 模式。Copilot 阅读 PRD、Architecture 和 Roadmap 后，将工程拆成 22 个任务，建立 30 条依赖关系：后端 scaffold、model、schema 是 discovery、parser 和 router 的前置任务；前端 scaffold 可以与后端基础设施并行；API 和 hooks 完成后，layout、feed、article、toolbar 等组件才能展开；搜索、OPML、测试和文档则位于更靠后的层级。

确认计划后，视频输入：

```text
/fleet please follow up the plan to implement
```

Fleet 首先找到两个没有前置依赖的起点：`backend-scaffold` 和 `frontend-scaffold`，将它们并行派发给后台 subagent。随着任务完成，新的工作被逐批解锁。这不是把 22 个任务同时启动，而是“能并行的并行，有依赖的等待，完成一层再调度下一层”。

最后，主 Agent 收回各分支结果，运行完整后端测试和前端构建。录屏中共有 41 个后端测试通过，前端成功构建，并生成了完整的项目结构与实现说明。

## `/fleet` 如何工作

按照 GitHub 官方说明，Fleet 会：

1. 将目标拆成带依赖的独立工作项；
2. 判断哪些可并行、哪些必须等待；
3. 派发后台 subagent；
4. 轮询结果并调度下一批任务；
5. 验证输出并合成最终结果。

每个 subagent 有独立的上下文窗口，但共享同一个文件系统；subagent 之间不能直接交流，只能由主 Agent 协调。[GitHub Blog：Run multiple agents at once with `/fleet`](https://github.blog/ai-and-ml/github-copilot/run-multiple-agents-at-once-with-fleet-in-copilot-cli/)

因此，Fleet 的价值不取决于 Agent 数量，而取决于任务依赖图中有没有足够宽的“并行层”。例如，前后端可以同步搭建，但依赖 schema 的 router 必须等待；不同 UI 组件可以并行，但统一页面组装应放在它们之后。

独立上下文也能让负责 parser 的 Agent 专注 RSS 解析，让负责 layout 的 Agent 只加载 UI 所需信息，减少无关上下文干扰。

### 真正的瓶颈是任务图，而不是模型速度

把一个十小时的串行任务交给十个 Agent，并不会自动缩短到一小时。如果十个步骤首尾依赖，后一个必须等待前一个产出，Fleet 仍只能串行推进。真正适合并行的计划，通常具备三类结构：

- **按技术层拆分**：前端、后端、基础设施在接口确定后分别推进；
- **按模块拆分**：多个相互独立的包、页面或服务各有明确所有者；
- **按工作类型拆分**：实现、测试、文档可以在输入稳定后分轨完成。

拆分时还要警惕“看起来独立，实际共享状态”。例如多个 Agent 分别开发页面，却都需要修改同一个路由入口；后端和前端同步开发，却没有先固定字段名称和错误响应。好的计划会把这类协调点提取成前置任务：先确定 API contract、公共类型和目录结构，再展开并行工作。视频中 Copilot 识别出前后端唯一的主要协调点是 schema 定义的 API contract，这正是后续多条工作线能够安全展开的基础。

主 Agent 的职责也不只是派活，还要持续检查状态、读取结果、判断依赖是否满足，并在最后做跨模块验证。

## 为什么建议先 Plan，再 Fleet

虽然可以直接输入 `/fleet <目标>`，复杂工程更适合先 Plan、后 Fleet。Plan 的作用不是生成一份漂亮清单，而是提前确认：交付物是否具体、任务是否真正独立、依赖是否合理、最终由谁集成和验收。

GitHub 官方推荐的典型流程也是：按 `Shift+Tab` 进入 Plan 模式，完成实现计划后选择 Autopilot + `/fleet`，或退出 Plan 再输入 `/fleet implement the plan`。[GitHub Docs：Speeding up task completion](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/speed-up-task-completion)

这里需要区分：

- `/fleet` 解决“谁能同时做什么”，核心是并行编排；
- Autopilot 解决“是否需要用户逐步确认”，核心是持续自主执行。

两者可以组合，也能独立使用。视频是在 Autopilot 下启动 Fleet，因此主 Agent 可以持续派发后续工作。

## 怎样写好 Fleet Prompt

一句“帮我把项目做完”很难形成有效并行。更可靠的 Prompt 应明确四件事：交付物、文件边界、依赖关系、验收标准。

```text
/fleet Implement FeedHub according to docs/PRD_2026_6.md.

Tracks:
1. Backend owns backend/app/** and backend/tests/**.
2. Data layer owns frontend/src/api/** and frontend/src/hooks/**.
3. UI owns frontend/src/components/** after hooks are ready.
4. Docs are updated after implementation and validation.

Constraints:
- Do not modify files outside each assigned area.
- Preserve the frontend/backend API contract.

Done means:
- Backend tests pass.
- Frontend type-check and production build pass.
- Report changed files, validation results and remaining risks.
```

如果项目已有 PRD、`.github/copilot-instructions.md`、`.github/agents/<name>.md` 或架构文档，应让 subagent 明确读取它们。subagent 看不到主 Agent 的完整对话历史，派发给它的任务必须自包含，或引用可读取的上下文文件。GitHub 官方也强调，应把工作映射到具体文件或测试，并写清边界、限制和验证条件；目标过于模糊，执行很可能退化为串行。[GitHub Blog：Write prompts that parallelize well](https://github.blog/ai-and-ml/github-copilot/run-multiple-agents-at-once-with-fleet-in-copilot-cli/#write-prompts-that-parallelize-well)

“实现功能”只描述了动作，没有定义可验证结果。完成标准应具体到新增文件、必过命令、需验证的用户路径和待报告的风险，避免各 subagent 对“完成”产生分歧。

## 最大风险：多个 Agent 同时写一个文件

subagent 共享文件系统，但没有文件锁。如果两个 Agent 同时修改一个文件，后完成的一方可能静默覆盖先前结果，并不会自动出现合并冲突提示。[GitHub Blog：Avoiding common pitfalls](https://github.blog/ai-and-ml/github-copilot/run-multiple-agents-at-once-with-fleet-in-copilot-cli/#avoiding-common-pitfalls)

因此最好遵守“单一文件所有者”原则：按目录或文件划分责任，把 `README.md`、路由注册表、依赖清单等公共文件留给主 Agent 统一集成。视频中的前后端适合并行，正是因为目录边界清晰，协调点集中在 API contract，没有争抢核心文件。

## 如何观察和干预

使用 `/tasks` 可以查看本会话的后台任务。当前官方文档中的操作是：方向键选择、`Enter` 查看详情、`k` 终止任务、`r` 清理任务、`Esc` 返回。[GitHub Docs：Monitoring progress](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/speed-up-task-completion#monitoring-progress)

录屏版本显示的是 `x kill`，与当前文档不同。CLI 快捷键可能随版本变化，应以本机界面和 `/help` 为准。执行过程中也可以继续指挥主 Agent，例如要求优先处理失败测试、列出活跃 subagent，或规定只有 lint、类型检查和测试全部通过才能标记完成。

观察 Fleet 时可以检查三个信号：计划是否被拆成多条工作线；任务面板是否同时出现多个后台 subagent；进度更新是否来自不同模块。如果只有一条长任务持续运行，通常说明目标太模糊、文件边界重叠或依赖声明过重。此时与其盲目增加 Agent，不如暂停并要求主 Agent 先重新拆分独立 tracks，逐项报告所有者、输入、输出和阻塞条件。

## 什么时候不该使用 `/fleet`

视频结尾有个很好的反例：应用生成后，实际运行发现“添加网站 URL 时无法输入”。问题最终定位到 `AddFeedModal.tsx` 中 effect 的依赖项导致状态被反复重置。它目标集中、修改范围小、调查路径基本串行，交给普通单 Agent 更合适。

Fleet 适合跨模块重构、API/UI/测试并行开发、多份独立文档更新、多个包或服务的批量处理。严格串行、集中在单文件、强共享状态或规模很小的任务，普通 Copilot CLI 通常更简单。Fleet 本身有协调成本，只有存在真实可分配的并行工作时才值得使用。[GitHub Blog：When to use `/fleet`](https://github.blog/ai-and-ml/github-copilot/run-multiple-agents-at-once-with-fleet-in-copilot-cli/#when-to-use-fleet-and-when-not-to)

更多 subagent 也意味着更多独立模型交互，可能消耗更多 GitHub AI Credits。录屏中的 AIC 只代表本次会话，不能作为其他项目的固定基准；任务规模、模型、重试次数和 Prompt 质量都会影响消耗。[GitHub Docs：Points to consider](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet#points-to-consider)

## 最后

真实项目中，可以从一个有 3～5 条自然工作线的任务开始：

1. 先让 Plan 列出任务、依赖、文件所有者和预期产物；
2. 人工检查并行任务是否会修改同一文件，公共接口是否已经固定；
3. 在 Fleet Prompt 中写明禁止修改范围和依赖顺序；
4. 把单元测试、类型检查、构建和 smoke test 设为统一质量门禁；
5. 通过 `/tasks` 检查是否真的形成并行，而不是只生成了更长的计划；
6. 结束后检查 diff、运行产品，并重点验证跨模块连接处。

这里最后一步不能省略。subagent 可以分别证明自己的局部任务完成，却未必能发现组合后的交互问题；单元测试也可能覆盖业务逻辑，却遗漏焦点、输入、响应式布局等真实体验。视频结尾发现输入框问题，正说明最终验收应从“代码是否生成”走到“用户是否能完成关键路径”。

FeedHub 演示既展示了 Fleet 的能力，也说明了它的边界：多 Agent 可以快速完成从后端、前端到测试文档的多线工作，但“构建成功、测试通过”不等于产品体验没有问题。

`/fleet` 的意义不是让开发者退出流程，而是让我们从逐项执行者转变为任务设计者和质量负责人。从一个 Agent 到一支 AI 开发团队，关键不是拥有更多 Agent，而是学会组织它们。

## 参考资料

- [GitHub Docs：`/fleet` 概念与注意事项](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)
- [GitHub Docs：使用 `/fleet` 加速任务](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/speed-up-task-completion)
- [GitHub Blog：`/fleet` 原理与实践](https://github.blog/ai-and-ml/github-copilot/run-multiple-agents-at-once-with-fleet-in-copilot-cli/)

