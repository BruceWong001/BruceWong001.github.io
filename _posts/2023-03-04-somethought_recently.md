---
layout:     post
title:      再见二月，谢谢你带给我的思考
subtitle:
date:       2023-03-04
author:     Bruce Wong
header-img: img/ewan-buck-4WdWzfV4KNE-unsplash.jpg
catalog: true
tags:
    - 敏捷
    - Agile
    - 随笔
---

短暂又紧张的二月份在不知不觉中过去了。接触了不同风格的团队，引发了一些思考。在这里做个总结，也算对2月份的一个告别。


+ 有时候强制添置一些约束条件。  
    例如：将迭代的Timebox缩短（从两周缩短到一周），能引发团队进一步拆分任务的欲望，更进一步是引发对核心业务需求的讨论。更关注本质，去繁求简。
+ 重新认识验收条件（AC）和Task的关系。  
    如果团队通过AC来梳理需求，并能够分离出核心需求和非核心需求对应的AC，那么每一个AC可以作为一个Task分配给团队。
+ 计划越完美越会让自己相信这是可行的。  
    忘记从哪里看到了这句话。觉得很对就分享到这里了。想一下确实会这样，有时候完美的计划会蒙蔽我们，忘记了当计划制定完的那一刻就已经过时了。它只代表在当时的上下文中我们的一种预判。计划需要能够随时应对变化，并能够随时做出调整。
+ 多团队合作的时候，不同团队文化的融合是关键。  
    塔克曼团队模型中风暴期（storming）在多团队融合初期尤为突出。团队文化的碰撞有时候会很激烈。最好让这种多团队融合在项目或者产品开发早期，这样能有更多的时间调整度过这一时期。否则会影响团队效率。
+ 任何一个事情发展到现在都有它独有的经历和原因。在没有全面了解之前，不要进行评判。同时需要尊重已有的事实和所有人付出的努力。  
+ 康威定律的思考。  
    《高效能团队模式》这本书让我进一步了解了康威定律，并让我认识到好的软件架构需要考虑团队的各个方面：团队拓扑结构，团队交互模式。  
    团队边界最好的划分方式是通过领域划分，高内聚低耦合。这也是领域驱动设计（DDD）希望达到的。看来好的软件设计思维也一样能够构建好的团队模式。  
+ 好的微服务架构都是从单体演化而来的。而不是从一开始设计而来的。  
    这也是我目前为止的一家之言，纯属实际感受。最近接触过的几个号称基于微服务设计的产品或者项目，都存在各种服务划分不合理，导致性能效率甚至可维护性低的问题。有时候团队抱怨这样的设计不如单体架构容易和高效。好的微服务设计一定是基于对业务领域的理解，并能够很好的基于领域边界来进行切割。简单的基于技术组件进行划分可能会导致强依赖，最终仍然是一个分布式单体。终于理解了那句话：架构需要是不断演进而来的，而不是从一开始设计而来的。
    
*践行敏捷实践，让工作变得更美好。欢迎关注我的公众号，交流落地经验。*