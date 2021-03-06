---
layout:     post
title:      敏捷开发需要内外兼修
subtitle:   
date:       2021-01-16
author:     Bruce Wong
header-img: img/post-bg-mma-3.jpg
catalog: true
tags:
    - Agile
    - 敏捷

---

修炼敏捷和练武功类似，有外家功也有内家功。提到敏捷我们第一时间想到的可能是Scrum的3355、Kanban的拉动式任务流、TOC约束理论、精益控制浪费等，这些框架、思想会指导我们在流程上做出调整。通常只要按照要求执行能够在较短的时间内看到一个改善的效果。他们类似外家功。不过随着持续的进展，大家会发现前进的步子有些迈不动了。就像修炼武功，外练筋骨皮，内练一口气。只有内外兼修才能持续精进，这么看修炼敏捷和修炼武功还真是相似哈。大成的武功都会有内家功，而内家功的练习需要心法，同时需要更长的时间持续的刻意练习、思考，效果不像外家功那么容易看到，不过它属于润物细无声，一旦量变达到质变将是一次飞跃。
说到软件开发的内家功当然是要围绕软件编程、测试等一系列和软件产出相关。极限编程(XP)是敏捷软件开发的一个流派，他提出的12原则中有很多被公认的敏捷软件开发的内家功心法，其中和软件研发相关的如下：
- 简单设计(SOLID)  
- 结对编程  
- 测试驱动开发(TDD)  
- 持续集成(CI)  
- 集体代码所有权  
- 编码标准  

他们中任何一个原则都很好理解，可以说是心法，不过没有认真和长期的实践，很难的到其精髓。今天就想来聊一下我们为什么要重视修炼这些内功。 
### 迭代 VS 增量
下面这个图大家应该都不陌生。很好的方式来讲增量开发和迭代开发的不同。
![interation vs incremental](/img/scrum/mona-lisa.png)   
我们经常用这个图来解释敏捷开发和瀑布开发的区别。从功能交付还是按照业务价值交付角度看，传统软件开发是增量交付，而敏捷是迭代方式。迭代交付我们需要保证整个业务价值可用，每一次交付都要保证新增加的部分和之前部分的完全融合，就像这幅画，每一次客户看到的都是一副完整的画作。而功能交付希望它只需要保证单个部分可用，最后合到一起给客户就是一个完整的画作(这里有一个假设几个功能交付可以无缝合并，不过现实往往无法做到)。所以在传统软件发布之前还是需要一次调整，这个就是传统软件开发中的在开发结束之后的集成测试，还有最终交付之前的用户UAT的过程。也就是说敏捷开发要想持续的高效交付，需要保证每次迭代的全量交付，因为取消了交付前的大规模集中测试了。
那么这两种交付和内家功有什么关系呢？我用下面的图解释一下。迭代的工作形式就是外家功，例如采用Scrum的方式。但是没有自动化测试等工程实践，也就是缺少内家功。如果严格按照迭代交付标准的话会有一个怎样的趋势：(下图是Ethan Huang 老师手绘作品,版权归Ethan老师哈哈)  
![practise](/img/scrum/iterationrate.png)   
+ 当Sprint1的时候要做10个story，这个时候在一个2周TimeBox的迭代周中。DEV和QA的时间分配比例大概是60%和40%。  
+ 按照迭代开发的方式(请看上面的蒙娜丽莎)，在做一次迭代的任务，无论做多少，都要保证新增加的和Sprint1的内容整体可用。 
+ 第二次迭代如果保证Sprint 1的内容都完成，那么至少还是需要40%的测试时间，能多测的任务时间就会挤占开发的60%区域，所以，开发不太可能还开发10个story。可能只能做6个，DEV和QA的比例可能是40%和60%。整个团队保证16个story的价值交付。  
+ 以此类推，第三次的迭代内容需要保证前两次的都没有问题。可能只能做3个story，而DEV和QA的分配降低到20%和80%。整个团队保证总共19个story的价值交付。  
+ 按照这个趋势，很快超不过几次迭代就会面临迭代内只能测试功能而无法开发新任务的局面。  

为了进一步说明上面的过程对迭代的影响，我画了一个系统循环图来说明一下在迭代中几个因素的关系。
![practise](/img/scrum/systemcycle.jpg)  

可以看到，红色和蓝色回路是系统平衡回路。他们虽然能够让整个系统趋于自平衡。不过前提是"编码任务量"的减少能够带来"代码质量"的提升。而且这个过程是有一定延迟的，会导致系统改变被延迟，如果有恶化会被持续一阵并放大影响。黑色回路属于增强回路，他的恶性循环会影响蓝色和红色循环，最终导致黄色的"迭代价值"降低。这是我们最不愿意看到的。所以要想尽办法让黑色回路变成良性循环的增强回路，这样才能让迭代价值持续保持。可以看到为了达到这样的增强回路，绿色的箭头是可以进行改进的地方，通过这些改变来影响整个系统回路的调整。这也正是敏捷内家功发挥威力的地方。  
> 系统回路相关内容请参考"系统思考"相关资料  

## Bruce有话说  
敏捷圈内一直流传着一个"诅咒"，没有工程实践的敏捷团队活不过3次迭代。以往软件工程中的集成测试和UAT等测试工作，往往是集中优势兵力在固定的时间段内做集团作战，这让很多开发团队在前期一心只想做功能，同时不管这些功能的质量只在乎数量。因为大家认为在最后发布之前会有集中的时间段测试和修改bug。在这个瞬息万变的VUCA时代，每次迭代恨不得都可以做一次发布，哪里还有机会给你集中测试。敏捷开发的迭代每一次更像是一次为正式发布而做的冲刺。软件团队一方面需要硬核的技术层面的实践，例如 TDD，BDD，自动化测试，DevOps等诸多实践。类比武功这就是内功心法，需要长期的练习和思考才能达到较高的境界。另一个方面是软技能方面的实践，例如 流程改进，与人的沟通合作等。类比武功就是外功套路。敏捷软件开发想落地并持续其生命力，需要内外兼修，这样才能走得稳，走的的持久。
