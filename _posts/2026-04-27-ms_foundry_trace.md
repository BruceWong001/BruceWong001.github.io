---
layout:     post
title:      Microsoft Foundry可观测性-Tracing
subtitle:   Microsfot Foundry 系列(1)
date:       2026-04-27
author:     Bruce Wong
header-img: img/IMG_1513.WEBP
catalog: true
tags:
    - Microsoft Foundry
    - AI
---

如果你在做企业级的AI Agent开发，那你一定无法避免的一个问题就是：**可观测性**。AI Agent的开发和传统的软件开发有很多相似之处，但也有一些独特的挑战：
+ 不确定性 — 相同的输入, 不同的输出结果
+ 多步骤推理 — 链条中的一个错误会导致整个过程失败
+ 调用工具 — 即使参数正确，选择错误的工具仍然会失败

Microsoft Foundry作为一个专注于AI Agent开发的平台，在一个地方提供了完整的Agent开发可观测性所需要的所有工具和能力，我会用接下来一个系列来介绍这些概念和知识。不过你完全可以找到对应的开源代替工具，从而使用你熟悉的技术栈。

Foundry Tracing（追踪）能做什么：
-  Tracing让你更好地理解Agent的行为和决策过程。
-  Tracing帮助你发现Agent的瓶颈和性能问题。
-  Tracing Log可以构建评估用的黄金数据集。

<iframe
  src="//player.bilibili.com/player.html?bvid=BV1HFoRBcENv&cid=37820760656&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen>
</iframe>

## 小结

Tracing意味着记录Agent执行的每一个步骤：每一个提示词，每一次工具调用还有每一个决策。没有Tracing，就像在黑箱中调试Agent，你无法看到内部，也无法找到问题。

