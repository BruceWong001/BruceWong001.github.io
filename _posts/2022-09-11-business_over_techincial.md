---
layout:     post
title:      ATDD的小妙用
subtitle:
date:       2022-09-11
author:     Bruce Wong
header-img: img/tdd12.jpg
catalog: true
tags:
    - 敏捷
    - Agile
    - 随笔
---

最近在和一个团队的小伙伴们一起，尝试在项目中应用一些工程实践，从而提高项目的完成质量。因为项目是对一个遗留系统做改进，而小伙伴们大部分是从别的团队重新组队过来的，所以遇到了一个很普遍的问题——对代码不熟悉；对业务逻辑也不熟悉的小尴尬。今天分享这期间我们进行的一个小尝试。个人感觉还不错，希望带给小伙伴们启发。

**遇到的困难**
+ 背景是希望改进一个已有功能的逻辑，增加新功能。
+ 开发看code的结构对已有业务逻辑的梳理，结果越看越一头雾水，像递归调用的函数一样，一层一层越看越深入。只见树木不见森林。
+ code本身的特点，并不具备业务描述性，从而导致开发，就算读懂code，也无法获得对应业务逻辑的全貌。更无法进行进一步的业务开发。
+ 如果code以往的开发就带有不准确性，那么从code的角度反而会误导新的开发人员，离正确的道路越来越远。

**采用的办法**
+ PO、QA、Dev一起参加一次1小时左右的会议。
+ 不看代码。大家对着一个屏幕，打开当前的应用程序。对需要修改的业务逻辑部分，用具体的用户角色来操作应用。
+ PO、QA、Dev基于自己当前对需求的理解，用AC(验收标准)的描述形式来演示并介绍自己对需求的理解。
+ 充分讨论，直到三方达成一致，之后产出物是一组AC(既包括已有逻辑的AC，也包括新作业务的AC)。而且可以带着界面截图，方便大家回顾。
+ 基于形成的最终AC，Dev重新看code并制定重构或修改方案。

**注意事项**
+ 一定要用Gherkin语句。它的好处可以参看我之前的一篇文章[《一次ATDD的团队实践》](https://brucetalk.com/2022/01/08/atdd-practice/)
+ 会议期间不考虑代码逻辑，只关注合理的业务逻辑。
+ 最好是结合已有系统的操作，并使用Role带入使用场景讨论，直观形象。

**下面是同一个开发小伙伴，经过两种方式后的梳理产物的对比：**
1. 从code梳理出来，当前部分的业务逻辑：(错综复杂的网状关系让人眼花缭乱)
![code perspective](/img/data/code_refine.png)

2. 通过AC梳理出来，当前部分的业务逻辑：(树形结构简单直观)
![ac perspective](/img/data/ac_refine.png)


看到这里小伙伴们可以发现，这种AC的梳理就是ATDD的过程。没错，其实一开始我也没想到会是这种结果，当时只是实在无法理解code。从开发小伙伴梳理的脑图就能看出来，他当时的内心是多么的凌乱。所以索性不看code，带着他从业务角度重新入手，没想到结果竟然更加简单的结构。这也是ATDD的一个小妙用。不禁感叹AC真的是业务方和团队之间的粘合剂，用好了事半功倍。

*践行敏捷实践，让工作变得更美好。欢迎关注我的公众号，交流落地经验。*