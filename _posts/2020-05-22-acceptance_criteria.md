---
layout:     post
title:      验收标准如何写？
subtitle:   PO的正确姿势
date:       2020-05-22
author:     Bruce Wong
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Agile
    - 敏捷
    - Acceptance Criteria(AC)
    - 验收标准
---

作为Product Owner(PO)的你是否挣扎在和团队讲解需求的各个会议中？是否惊讶为什么同样的需求已经讲过多遍但团队依然不理解？是否惆怅希望有更多的时间思考需求和产品路线图而现实情况是分身乏术经常被团队成员打断思路要解释某个功能？今天我们讲讲PO正确姿势的一种——验收标准如何写。  
![Retrospective Meeting](../../../../img/scrum/Agile.jpg )  
验收标准作为PO和Team之间的一种契约，是对迭代交付的用户故事的唯一检验标准。以往的经验认为PO是需求的管理者，他来编写验收标准是最适合不过的，不过往往实际效果是，PO很努力的讲解用户故事，分析需求原因，Team在下面玩手机；PO拿出验收标准，大家看都不看一眼继续玩手机。当实际迭代开始做的时候，QA会直接拿验收标准当作测试用例，好一些的会有些扩展，开发基本不考虑全都等着QA验证。
> 记得某位大牛说过: 软件的质量在你写code的那一瞬间就已经注定了，并不是测试能改变的。 

既然验收标准是一个契约，而契约本来就是人与人合作之间的标准，我们何不从他入手做一些文章？让团队在PO介绍需求之后，Team一起合作写出他们认为当前功能需要的验收标准(AC)。PO 通过查看AC来验证是否Team理解了他的需求。同时在Team写出验收标准的时候PO会惊奇的发现，以往你千辛万苦地编写AC在这个时候才思泉涌般的被Team贡献出来，有时你还会发现一些意想不到的新点子。最后AC作为一种文档保留给Team在迭代中用于扩展测试和开发自测的标准。

我的Team应用了这个实践之后有如下的效果反馈：
- 动手写AC比单纯听一遍PO印象更深刻。
- AC促使Team参与需求讨论，增加参与感。
- Team开始思考需求的原因而不是仅仅实现功能。
- Team更早的开始考虑前提条件和依赖关系，而不是到做的时候才考虑。
- 尽早达成一致，识别任务大小，减少误判。
- 故事足够小的时候（AC）估算会更容易。
- Team和PO成为伙伴关系。彼此更加理解。  

解放PO，让PO去做更有价值的思考思考，让团队更多的参与到功能讨论中，释放潜能，让Idea呈现涌现状态(一个人和一群人出主意的区别 :smile: )，这才是敏捷应该走在的方向上。这也是PO的正确姿势之一。

> Tip：有很多团队认为AC就是测试case，或者就是一句话“实现需求要的功能”。这就失去了上面说的诸多好处。最后附带一个AC的编写模板，团队可以从模板开始找找感觉：
+ Given:[a pre-condition]
+ When: [user inputs]
+ Then: [user gets expected results]


