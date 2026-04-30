---
layout:     post
title:      Microsoft Foundry可观测性-Evaluation
subtitle:   Microsfot Foundry 系列(2)
date:       2026-04-30
author:     Bruce Wong
header-img: img/IMG_1513.WEBP
catalog: true
tags:
    - Microsoft Foundry
    - AI
---

Agent落地企业，Tracing解决"看清发生了什么"，Evaluation解决"判定做得对不对"——两者缺一不可。
最近在用 Microsoft Azure AI Foundry 做 Agent Evaluation，它的价值不只是打分，而是把评估变成了工程化的质量门禁：

+ ✅ 评估对象灵活：Agent、Model、Dataset 都能测.
+ ✅ 数据不愁：没有黄金数据集时，Foundry 能自动合成测试数据.
+ ✅ 评估器开箱即用：40+ 默认 evaluator 覆盖 Agent 全生命周期.
+ ✅ 工程集成：SDK 输出评估分数，直接接入 CI/CD 做质量门禁.

从"看着不错"到"测了才算"，这一步决定了 Agent 能不能进生产环境。

---
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1xaoCBzEKh&cid=37883874120&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen>
</iframe>

---
## 为什么是 Azure AI Foundry？
企业做 Agent Evaluation 往往卡在三个地方：

1. **没有数据** — 黄金数据集标注成本高
2. **不知道测什么** — 评估维度定义困难
3. **测完没法用** — 分数和工程流程脱节

Foundry 用三个设计解决：

| 痛点 | Foundry 解法 |
|--------|-------------|
| 没数据 | **合成测试数据** — 基于场景自动生成 |
| 不知道测什么 | **40+ Built-in Evaluator** — 覆盖 Agent 全生命周期 |
| 测完没法用 | **SDK + CI/CD 集成** — 分数直接变成质量门禁 |
---
# 40+ Built-in Evaluator 覆盖什么？
这是 Foundry 的核心资产。开箱即用，不用自己定义评估逻辑。
### 三大类评估维度
| 类型 | 代表 Evaluator | 解决什么问题 |
|------|---------------|-------------|
| **基础质量** | Groundedness, Relevance, Coherence | 回答是否准确、切题、连贯 |
| **Agent 能力** | Tool Call Accuracy, Parameter Correctness, Session Completeness | 工具调用对不对、参数准不准、任务有没有完成 |
| **安全合规** | Harmful Content, Jailbreak Resistance, Protected Material | 有没有幻觉、有没有被注入、有没有泄露 |
---

**你的团队开始做AgentEvaluation了吗？要不要尝试一下Microsoft Foundry?**

