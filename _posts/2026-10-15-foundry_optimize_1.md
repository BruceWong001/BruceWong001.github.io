---
layout:     post
title:      Microsoft Foundry Agent 优化实操：从评估结果到 Model Router 配置
subtitle:
date:       2026-10-15
author:     Bruce Wong
header-img: img/IMG_1861.jpeg
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI

---

Agent 能正常运行之后，还可以继续改进提示词、模型选择和工具描述。Microsoft Foundry 的 Agent Optimizer 把这几类尝试放进同一套评估流程：生成候选配置，与原版本比较，再由我们决定是否采用。

下面的视频用 Foundry Agent 演示了完整操作，本文不复述每一步界面，只补充视频没有展开、但决定成败的几个知识点。

---
<iframe
  src="//player.bilibili.com/player.html?bvid=BV14ypd6FEQt&cid=42683730597&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen
  title="演示 Foundry Agent Optimizer 优化流程"
>
</iframe>

---

## 先分清三种模型，再谈优化

优化任务里出现的模型有三个角色，混淆它们会误读结果：

| 界面中的模型 | 负责什么 |
| --- | --- |
| Agent 当前模型与候选模型 | 执行业务任务，是被比较的对象 |
| Optimization model | 分析结果、生成改进配置 |
| Evaluation model | 按评估标准为回答打分 |

更换 Evaluation model 是换了评分者，并不等于替 Agent 换模型；具体支持列表以[Agent Optimizer 文档](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-optimizer-overview#models)为准。另外，向导 Target 步的 **Maximum candidates（最大候选数）**直接决定耗时和费用：每个候选都要完整跑一遍数据集。

## 优化会真实执行你的工具

这是最容易被忽略的一点：优化不是对配置的静态分析。优化器会把基线和每个候选逐条跑完整个数据集，Agent 的外部工具调用（API、数据库、第三方服务）在每次评估中都会真实执行，规模约为（候选数 + 1）× 数据集条数，计费、状态变更、限流都会随之放大。文档的建议是：为会产生副作用的工具准备测试端点，或在优化期间使用 mock 实现。[优化执行说明](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-optimizer-overview#how-the-agent-optimizer-works)

对视频里的数据合规 Agent，这意味着扫描类工具若连着生产数据源，优化期间会被反复执行。

## 数据集和评分标准定义了“更好”

Optimizer 只能向数据集看齐：数据集提供任务，评估器（Criteria）决定什么表现值得更高分。用 traces 生成数据集时，要检查代表性和失败样本是否覆盖；上传数据时注意列名必须与评估器要求一致，当前向导不支持列映射。[数据集与评估标准配置](https://learn.microsoft.com/en-us/azure/foundry/agents/quickstarts/quickstart-optimize-prompt-agent#select-an-evaluation-dataset)

## 怎么读优化结果

视频中的最佳候选 candidate_3 与基线对比：

| 指标 | Baseline | candidate_3 |
| --- | ---: | ---: |
| 评估分数 | 0.542 | 0.607 |
| 平均 Token 数 | 944.5 | 1.9K |
| 平均延迟 | 2.0 秒 | 2.9 秒 |

复合分数（composite score）是各评估器原始分缩放到 0–1 后取的平均；按文档说明，评分模型对每条任务按每条标准各评一次、返回二值分数，所以小幅度波动应视为噪声。官方文档给出了增幅解读标尺：[分数增幅解读](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-optimizer-overview#interpret-score-improvements)

| 增幅 | 解读 |
| --- | --- |
| 小于 0.03 | 噪声，不视为有意义的改进 |
| 0.03 – 0.10 | 中等改进，值得部署 |
| 0.10 – 0.20 | 显著改进 |
| 大于 0.20 | 重大改进，通常说明基线本身较弱 |

按这个标尺，视频中的 0.065 属于“中等改进、值得部署”，前提是接受 Token 与延迟的增加。Token 变多也不等于费用变贵，因为候选模型单价可能不同；Review 页的费用估算（Min/Est/Max）同样不是支出上限。[优化费用与 Token 说明](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-optimizer-costs)

### 应用前先看 View Changes

推荐方案同时改写了系统提示词，并把模型从 `gpt-4o` 换成 `gpt-6-luna`——所以本次提升不能全部归因于“换了模型”。点击 Promote candidate 会创建新版本；如果 Agent 固定在某个版本（pinned），还要手动切换 active version 才会接收流量。[候选推广与版本行为](https://learn.microsoft.com/en-us/azure/foundry/agents/quickstarts/quickstart-optimize-prompt-agent#promote-a-candidate)

## 工具描述优化的边界

视频后半段的 Tool description 优化任务报错：

```text
AgentCandidate must set at least one tool for mutate_tools to work
```

这正是能力范围的体现：这项优化只改写 Function calling 工具及参数的描述，不修改参数类型、默认值和必填字段；MCP、Toolbox 的描述不在支持范围内。但使用 MCP 工具的 Agent 仍可优化提示词和模型。顺带一提，Optimizer 还支持代码定义的 Hosted Agent，走 Foundry Toolkit / azd CLI 而非门户向导，并多了 Skill improvement 目标。[优化目标与 Agent 类型](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-optimizer-overview#optimization-targets)

## Model Router 是另一层：运行时选模型

Agent Optimizer 在一次离线评估里比较配置；**Model Router 在运行时按请求选择底层模型**，会参考系统消息、用户输入、工具定义和对话历史。同一 Agent 的不同请求可能由不同模型处理。[Model Router 工作方式](https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/model-router-how-it-works)

视频展示了 Router 的部署界面，但没讲这些设置的取舍：

- **Routing mode**：Balanced（默认，兼顾质量与成本）、Quality、Cost。模式表达的是路由偏好而非保证，每次只改一个变量，比较结果才好解释。[模式配置](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/model-router#optional-change-the-routing-mode)
- **Models subset**：Route to all supported models 的“全部”指 Router 支持且满足部署条件的模型池，不是项目里已部署的所有模型；选子集可限制参与路由的模型，新支持的模型不会自动进入子集。通常无需为 Router 单独部署底层模型，Claude 是例外。[子集配置](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/model-router#optional-route-to-a-model-subset)
- **版本与部署限制**：长上下文任务按子集中最小的上下文容量规划；TPM 和内容过滤是部署级设置，视频中的数值只是演示；固定 Router 版本号不代表底层模型池永远不变。[部署设置](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/model-router#optional-customize-deployment-settings)

验收方法：记录响应中的 `model` 字段查看实际选中的模型；保持提示词、工具和测试集不变，先验收 Optimizer 推荐的固定模型版本，再加入 Router 做对照，才能判断变化来自配置还是路由。视频没有展示 Router 与固定模型的评测对比，不能据此得出启用 Router 就一定更便宜或更准确。[如何公平评估 Router](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/evaluate-model-router)

## 总结

**视频精华** —— 完整走了一遍优化闭环：

- 用 Agent Optimizer 为数据合规 Agent 生成并复评候选，candidate_3 以 0.542 → 0.607 胜出，Promote 为正式版本。
- 尝试 Tool description 优化，因缺少可变更的工具而报错。
- 部署 Model Router，把模型选择交给运行时。

**文章补充** —— 视频没展开、但决定成败的取舍：

- Optimizer 离线比较候选配置，Router 运行时按请求选模型——两层机制，验收用同一套对照方法。
- 增幅小于 0.03 是噪声，0.065 属于“中等改进、值得部署”；但提示词与模型同时改写（先看 View Changes），Token 与延迟一并上涨，归因和取舍要谨慎。
- “更好”完全由数据集和评估器定义——评估流程有多可信，优化结果才有多可信，投入优先花在这里。
- 优化会真实执行工具，约（候选数 + 1）× 数据集条数次，先 mock 或测试端点再提交。
- 工具描述优化只覆盖 Function calling 工具，MCP、Toolbox 不在列；pinned 版本 Promote 后要手动切 active version，候选全部落败时保留现状。

数据集太小、缺少失败样本或评分不稳定时，先手工迭代、从 traces 积累数据更划算。优化器放大的是评估流程的价值——评估流程本身不可信时，它只会更快地产出不可信的分数。

**You can outsource your thinking, but you cannot outsource your understanding.**