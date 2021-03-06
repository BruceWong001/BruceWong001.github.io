---
layout:     post
title:      Scrum Patterns(65):准备就绪的标准 DoR(译)
subtitle:   A Scrum Book——The Spirit of the Game
date:       2021-01-31
author:     Bruce Wong
header-img: img/DefinitionOfReady_Head.jpg  
catalog: true
tags:
    - 敏捷
    - Agile
    - 译文
    - Scrum Patterns
---


##  Bruce有话说   
团队在采用Scrum工作方式的时候，都知道完成标准(DoD)的重要性，因为它可以在迭代结束的时候作为团队工作的整体验收标准。在Scrum中还有一个类似的标准叫准备就绪标准DoR(Definition of Ready)，往往会被大家忽略。它定义了产品待办列表项能够进入Sprint的门槛。如果没有准备好是不能开始Sprint。所以每次Sprint 计划会上PO宣讲的PBI(Product Backlog Item)都是应该达到DoR的，这也保证了PBI的质量以及计划会的质量。
有人会问，如果达不到DoR的PBI由于各种原因团队还是开始了会有什么问题呢? 我可以告诉你——风险很大。我这边就遇到过以下的情况：
1. 当此迭代任务，本团队有一个服务依赖于第三方团队完成才能开始本次迭代功能。而本团并没有对第三方服务的状态进行完整评估，也就是对于本团队来说当前任务并没有准备好，就开始了迭代。  
2. 带有技术可行性风险的任务并没有就绪就开始开发，导致迭代中发现技术问题。  

上面两个都是我经历的真实场景，最后的结果都是迭代没有被团队的完成。今天翻译的这篇Scrum模式文章就是对DoR的原因、目的、如何实施DoR等进行阐述。希望对你有帮助。

## 正文  

你有一个产品待办事项列表，它是一个细化的产品待办事项列表，或者差不多，你开始计划一个Sprint。出于需要，产品待办事项列表上的一些条目对于开发团队来说仍然太模糊，仍然无法实现。然而Sprint Backlog包含的工作项必须能够达成Sprint目标，同时能够指导开发的工作。因此，Sprint Backlog中的项目必须是具体的；他们不能含糊不清。那么说到底，对于开发团队完成他们工作来说，多“具体性”才够呢?  

✥       ✥       ✥ 

如果开发团队不能准确地理解产品待办事项列表(Product Backlog)，开发工作(和时间)可能会激增，这反过来会导致Sprint错过Sprint目标或不能交付干系人所期望的内容。  

![DefinitionOfReady_Pre](/img/scrum/DefinitionOfReady_Pre.jpg)

开发团队面临的挑战是获得新想法并且能够把他们开发出来。这是一个思维方式的改变：一个想法在一开始可能非常抽象，开发团队要实现它需要能够具象化到具体场景。如果你想要一个完备的计划(同时你也想要一个短期的能够给你的过程注入智慧的计划)，那么对于抽象想法的探索我们需要细化它，这样才可以回答任何程序设计或交互设计相关的问题。那么细化到什么程度呢？细化到Team有信心完成进一步的计划。  

> 我曾经为一家小公司做过一些外包任务。有一次，一个CEO让我写一些软件，并提出了一个固定的价格。“我们达成一致了吗?”他问道。“不,”我说。“这个软件的定义不够精确，我不知道需要多长时间完成它。”  

换句话说：在市场和设计的世界中间有一个断层。他们各自创造了不一样的演进方式：市场反应偏慢，而设计往往会快速聚焦（参考《实现规范Enabling Specifications》).他们之间需要一个组织级的边界。否则你就会在市场和设计两方面失去聚焦。理论上，你可以通过一个流程来实现这种分离。但是组织人员的关系会贯穿整个过程，因此你仍然面临这样一个问题：关注业务和价值领域，还是关注产品和技术领域。在这两者之间，人们还是左右为难。反过来，你可以从时间上来划分人员在这两个领域之间的关注来解决该问题。但是这会导致人们在任务上下文之间的切换，这会导致效率上的降低。Scrum将这些问题从组织层面区分开来，分成产品负责人(PO)和团队。而这两个领域相遇的接触点对于成功至关重要。 

当你处于计划的困境中时，有一种强烈的诱惑让你想要快速做出假设，并将棘手的问题（例如详细的估算）推迟到以后。而当一个小组一起计划时，往往会面临一些隐形的同僚压力，为了使计划过程能够顺利进行而将困难的问题推迟。要解决这个问题，团队必须达成一致，最先解决最棘手的问题。为了避免在不确定情况下开始工作，团队对截止时间点需要有一个明确的共识，并共同遵守这个客观标准。  

在精益思想中，不确定是造成浪费的一个原因。如果开发团队没有充分的理解PBIs的真正意思，或者如何开发它， 如何估算他都无法完成，那么对于Sprint的成果增加了不确定的风险，同时这种工作还可能会导致成本、风险和不确定性增加。  

#### 因此：

在Sprint计划会中，能够成为开发团队领取的当次迭代备选的PBIs，必须符合下面的条件：

1. 团队可以立即对PBI采取行动。  
2. 计划的交付是有价值的。  
3. 产品负责人和开发团队已经对其进行过讨论。  
4. 开发团队已经对他进行了评估。  
5. 它是可测试的，并且PO已经准备了专门的测试说明(译者认为这里说的就是AC)。
6. Scrum Team已经把当前的项目裁剪到适合的大小(参看Small Items).  
(译者注：这里6点其实就是INVEST原则。Independent Negotiable Valuable Estimable Small Testable)
如果PBIs满足上面的条件并且可以填满一次Sprint的需要，那么这个Product Backlog就是准备就绪了。
![DefinitionOfReady_Post](/img/scrum/DefinitionOfReady_Post.jpg)

一个好的准备就绪标准能够帮助团队处理好外部依赖。如果一个工作项的完成的控制权在外部团队，那么把他放到Sprint Backlog中会极大地增加Sprint结束时的交付风险，与此同时你还无能为力！把依赖关系分析做到BPIs层面，而不要仅仅在常规产品增量这个层面；团队最终需要理解PBIs层面的依赖关系，才能安排产品开发过程中的工作顺序。请将外部依赖作为准备就绪标准的考虑因素。  

在准备就绪和实现规范之间有一个重要的相互作用。那就是下一次Sprint的备选的PBIs最晚在Sprint计划会结束前必须变成准备就绪的状态。在计划会结束的那一刻，PBIs以及PO对需求的的说明还有问题的澄清都能够让团队毫无顾忌地开始实现他们才行。  

✥       ✥       ✥ 

Scrum团队的目标就是要满足所有就绪标准，因为他们致力于精炼产品待办列表，同时正确在Sprint计划会之前制定实现规范。这个活动的目标就是让PBI不被任何阻碍的通过“就绪关口”，每个代办项仅需要遵守简短的检查清单即可。虽然这个列表对于未开始开发的团队能够“叫停生产线”来说是很重要的，但更大的好处是可预期的约束条件，以及提前筹划PBIs，在Sprint开始前完全准备好他们，这样整个流程就不会受到阻碍。  

注意，在Product Backlog列表中并不是所有的项目都准备就绪了，他们通过在Backlog的从下往上的移动，应该朝着就虚的状态逐步演进。准备就绪定义了哪些PBIs可以进入Sprint，与之相对应的，完成标准定义了可以在Sprint结束的时候交付的BPIs的标准。  

如果一个PBI没有准备就绪会发生什么？虽然处理PBIs是整个团队的工作，不过PO的职责是在Sprint计划会来临前让团队所有备选项目都没有障碍。在大都数情况下，如果存在没有准备就绪的项目，那就意味着，PO必须回去组织更多的分析(译者注：这里我翻译成组织而不是PO做，因为我认为一些研究型的工作并不是PO来做，而是团队)，并在最后日期前再次把PBI拿到团队面前。Scrum的传统是，如果PBI在Sprint计划会的时候没有准备就绪，那么团队有权“去海滩玩吧”，就如同Jeff Sutherland在《Scrum：一半的时间内完成两倍工作的艺术》(137页“完成定义”)中描述的一样。这让产品负责人没有把他们的待办列表准备好这件事显而易见。如果你经常在"海滩上玩"，那说明你们的工作出现了问题。对产品代办列表的准备就绪这个事你还能做哪些事情？团队还能如何帮助PO做些什么呢？  

为了区分Scrum中的就绪概念与语言中“就绪”的不同，Scrum有时会使用短语“就绪的就绪（Ready-Ready）”，而不是单个次“就绪”。理查德.克隆法尔特在2008年发布的关于“就绪定义”的正式描述。  

非常感谢Peter Gfader的评论！
Picture credits: https://www.rawpixel.com/image/8657/hands-holding-bangkok-thailand-travel-guide-book-map-floor (under CC0 license).


## 原文引用
- [ScrumBook.org——Definition of Ready](http://scrumbook.org.datasenter.no/value-stream/product-backlog/definition-of-ready.html)
