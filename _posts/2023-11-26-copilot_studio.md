---
layout:     post
title:       Copilot Studio使用初体验
subtitle:
date:       2023-11-26
author:     Bruce Wong
header-img: img/bg1.jpg
catalog: true
tags:
    - AI
    - 随笔
    - Microsoft
---


继OpenAI Dev Days之后的又一个大型AI落地发布会非Microsoft Ignite 2023莫属。这次微软推出了一系列AI的落地功能，进一步拉低了AI应用的入门门槛，更多普通人可以通过No code或者Low code将AI融入到自己的工作和生活中。

![Untitled](/img/AI/Copilot_Studio/Untitled.png)

微软宣布Copliot（副驾驶/助理）作为新的AI交互方式，全面通过Office全家桶，Windows 11等多个途径铺开落地了。今天想介绍一下我最近的试用体验，作为普通人如何应用Copilot Studio这个工具来制作属于你自己的AI助手。

![Untitled](/img/AI/Copilot_Studio/Untitled.jpeg)

## 使用前提

目前的Copilot Studio还是Preview版本。你需要有自己的Microsoft Azure账户，需要有Power Platform的订阅，因为Copilot Studio是基于以前Power Virtual Agents平台做的改进，可以说将以前的Q&A Bot基于这波OpenAI热潮重翻新了。微软这方面确实很有一手。

**注意：**目前创作的Copilot只支持英语。

## 创建自己的AI助理（Copilot）

话不多说，直接来看看创作一个Copilot的过程。今天我打算做一个最简单的敏捷教练小助手，他可以基于敏捷研发相关的书籍、网站、博客等内容，回答敏捷新手在日常工作中的一些问题，并能给出回答内容的引用地址。

### 创建一个Copilot

打开Studio的首页，可以看到你已经创建的Copilot。

![Untitled](/img/AI/Copilot_Studio/Untitled%201.jpeg)

直接点击创建，输入名字。下面的web Site是Copilot的生成对话回复内容的默认来源。后面还有更多设置，这里只是一个default设置。

![Untitled](/img/AI/Copilot_Studio/Untitled%202.jpeg)

最下方的高级设置，还可以设置Copilot图标，数据存储内容，以及数据隔离的Schema。还可以开启语音功能，我暂时没有尝试。（小心：越高级的功能，用的很爽，扣钱也是如流水哦。哈哈）

![Untitled](/img/AI/Copilot_Studio/Untitled%203.jpeg)

### Copilot首页

进入新创建的Copilot主页之后，可以看到三个区域：左侧是所有设置内容选项；中间Test copilot是类似于Play Ground，所见即所得的测试你的机器人行为。例如你做了一些调整后，马上可以看到效果，直到满意为止；右侧区域是你可以对当前机器人的行为做设置，例如“Go to Generative AI”就是设置生成式AI相关设置，”Create toipic“是原来Power Virtual Agents机器人设置方式；另外两个仍然在Preview的，是第三方插件，可以和OpenAI等其他第三方的Plugins进行集成。

![Untitled](/img/AI/Copilot_Studio/Untitled%204.jpeg)

### Generative AI设置

进入Generative AI的设置，是不是很眼熟，最开始创建Copilot的Website就在这里，同时你可以加入更多的Website。虽然当前的限制是URL只能到两级路径，不过让Copilot直接读取Web站点这一点已经很方便了。同时你还可以上传文字为主要内容文档，用于给Copilot作为知识库。当前限制是每个文件3M。

![Untitled](/img/AI/Copilot_Studio/Untitled%205.jpeg)

### 设置Topics

这个是传统的Q&A bot部分，也就是传统的Power Virtual Agents。如果你眼尖，可以看到上面Generative AI设置最下面有一个“Advanced configuration”部分，内容是当Bot无法生成回复内容的时候设定一个Topic，也就是最后可以将对话都引导到一个固定的Topic。相当于编程语言中的If else。当生成式AI无法回答的时候，都统一走一个固定的回复，避免回复不了的不好体验。

Topic可以理解成当前的Copilot可以预设的一些对话主题。例如下面的这些主题，每个里面会有一些语义相近的词或者短语，作为触发某一个Topic的触发点。

![Untitled](/img/AI/Copilot_Studio/Untitled%206.jpeg)

可视化的设置每一个Topic里面的逻辑处理路径，例如下图，给客户返回的内容会有三个选项，当客户选择每一个内容的时候会有对应的新的回复路径。可以看到，在GPT出现之前，这种能够理解语义并可以做一些逻辑选择的聊天机器人，能够应对一些简单的对话机器人场景，例如我们平时常用的银行客服、宽带客服、电信运营商客服等。和当今的GPT相比，确实死板（笨）一些。不过由于GPT3.5/4的强大，Topic这种类型的配置几乎可以忽略了。我这次就没有配置。

![Untitled](/img/AI/Copilot_Studio/Untitled%207.jpeg)

### 发布到特定渠道

到目前为止不需要编写任何代码，一个简单的人工智能助理Copilot就做完了。接下来就是发布给大家试用。点击发布之后就可以选择渠道了。

![Untitled](/img/AI/Copilot_Studio/Untitled%208.jpeg)

作为一个聊天机器人，能通过什么平台来使用聊天机器人？微软结合azure bot service。很方便的集成已有的通讯频道。例如，Teams，Skype，Line，Email，Slack等。当然必须有Microsoft自己的Copilot无缝集成。你可以在office和windows系统的Copilot里直接使用你的个人AI助理喽。

![Untitled](/img/AI/Copilot_Studio/Untitled%209.jpeg)

甚至微软还提供了可以是你自己的网站嵌入一个web聊天对话框。

![Untitled](/img/AI/Copilot_Studio/Untitled%2010.jpeg)

用于看效果的Demo Website更是可以直接发给小伙伴play。

![Untitled](/img/AI/Copilot_Studio/Untitled%2011.jpeg)

打开上面这个链接，可以直接看到web页面，并和你的机器人直接对话。

![Untitled](/img/AI/Copilot_Studio/Untitled%2012.jpeg)

### 分析仪表盘

对于自己编写的Copilot，微软还提供了贴心的仪表盘，帮助你了解使用情况，从不同的视角让你观察工作效果，方便进行调整。

![Untitled](/img/AI/Copilot_Studio/Untitled%2013.jpeg)

## 小展望

1. 问答类的事务性工作肯定会逐步被替换，例如：HR回答员工日常事务性的工作。产品在线客户服务的第一级工作。企业能够节省大量的人力成本，同时提高服务时常，能够做到24小时在线，服务态度不受任何情绪影响，保持稳定。
2. 微软的Copilot、OpenAI的GPTs Agent这类低代码平台让人门使用AI的门槛几乎抹平。年初还在谈论的提示词工程，现在似乎也不是最重要的了，真的要进入技术平权时代了！那么在平权时代，技术已经不是核心竞争力，那么如何结合对企业业务的理解，合理的利用AI和工作结合，提高企业竞争力，将成为更重要的技能。也就是每个开发人员都是一个超级个体，Product Owner/BA +开发+测试+运营。
3. 用开放的心态，积极的在工作和生活中尝试使用AI的相关工具，让自己的生活和工作与AI工具结合，适应这种新的工作方式。应对未来的机会和挑战。

*践行敏捷实践，让工作去繁从简。欢迎关注我的公众号，交流落地经验。*