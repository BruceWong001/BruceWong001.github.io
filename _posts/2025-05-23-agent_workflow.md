---
layout:     post
title:      学习如何构建高效AI Agent
subtitle:
date:       2025-05-23
author:     Bruce Wong
header-img: img/IMG_6616.WEBP
catalog: true
tags:
    - Agile
    - 敏捷
    - 随笔
    - AI
---

Anthropic的工程师Barry Zhang在最近的AI Engineering大会上分享了《How We Build Effective Agents》的演讲，非常受用，学习之余结合自己的工作实践做了一些总结。以下是我对他演讲的内容的总结和思考。

## 1. 不要为任何事情都构建Agent(Don't build agents for everything)
Barry提到，Agent可以扩展复杂和有价值的任务，但并不是简单一对一的升级所有类型的任务。他提到一个是否需要构建Agent的简单的检查列表：

| 任务足够复杂吗？| No -> Workflows <br> Yes -> Agents |
| 任务有足够有价值？| <$0.1 -> Workflows  <br> >$1 -> Agents |
| 任务的所有部分都可以执行吗？| No -> Reduce scope <br> Yes -> Agents |
| 错误和发现错误的代价是什么？| High -> Read-only/human-in-the-loop <br> Low -> Agents|

非常简单的一个检查表。可以看到Agent更适合不差钱，足够复杂，同时可以完全自动化的任务。另外如果你发现如果Agent执行的任务真的发生了错误，是否能够接受。如果这些回答都是“是”的话，那么就可以考虑构建Agent了。否则，老老实实构建自动化workflow。

**Tips**
用上面的检查表来看一个实际的例子：为什么说写代码的场景最适合用Agent？
- 任务足够复杂吗？是的，代码可以写的非常复杂。
- 任务有足够有价值？是的，好的代码可以节省大量的时间和金钱。当然也可以换回来很多钱。
- 任务的所有部分都可以执行吗？是的，例如Claude就可以执行代码。
- 错误和发现错误的代价是什么？可以控制很低。通过Unit-Tests和持续集成来将错误控制在一个可接受的范围内。

## 2. 保持简单(Keep it simple)
**Agent定义：** Agent是模型在循环里面使用工具。
![Agent定义](/img/AI/agent/agent1.png)

- Env: 代表了Agent运行的上下文环境。可能是一个网页，一台电脑，或者是一个软件系统。
- Tools: 代表了Agent可以使用的工具。可能是一个API，也可能是基于STDIO的本地一个应用。
- System_Prompt：指导模型在一定的环境上下文中，使用工具来完成任务，从而达到目的。

基于上面的三个因素来构建Agent，并且持续优化这三个因素，可以让构建Agent更高效。

**Tips**
当你给系统提示词增加2个以上任务描述要求的时候，你会发现Agent会出现更大几率无法遵循你的指令来工作。因为系统提示词的复杂度增加了。你可以考虑将复杂的任务拆分成多个简单的任务，并使用多个单一任务的Agent来完成任务，这样更高效，同时单个Agent调试也更可控。

## 3. 像你的Agent一样思考(Think like your agent)
看下图，一个Agent的工作循环。可以想象一下，如果你是Agent本身是如何思考并执行的。类似于你闭上眼睛，一片漆黑，你可以获得上下文信息只有128K token，多一些的也就只有1～2M，这里面包含了可以使用的Tool的描述。之后你要在有限的信息中开始尝试执行任务。睁开眼睛看到且仅能看到一个结果的后，再决定做下一个动作。
![Thinkingloop](/img/AI/agent/agent2.png)
你的信息永远都是有限的。这时候你要考虑你作为Agent，如何才能高效的工作？

**Tips**
当你用上面的方式转变你的视角，你会发现简洁高效的系统提示词是多么的重要。只在必要的时候进行推理，毕竟上下文信息有限。这样你可以最大限度的提高Agent的执行准确率和效果。

## 总结与思考

Agent的构建是一个复杂的过程，并且不是所有场景都适合构建Agent。当你需要考虑到如何让Agent在有限的信息中做出正确的决策的时候。保持简单，关注最重要的因素，才能让Agent更高效的工作。

## 视频链接
[视频链接](https://www.bilibili.com/video/BV1dTdfYBEfw/?spm_id_from=333.1391.0.0&vd_source=bc6232dd33c98b2db7fedcfca5f9bf89)
