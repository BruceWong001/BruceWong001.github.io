---
layout:     post
title:      基于业务规则拆分用户故事
subtitle:   避免工作局促
date:       2024-07-20
author:     Bruce Wong
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - Agile
    - 敏捷
    - User Story
    - ProductOwner
---

有时候在规定时间内（例如项目周期内，单次迭代）。如果最终交付的工作任务要求高，团队会觉得完成不了而紧张。最简单的解决方案是分阶段交付，如果按照交付用户价值的标准来粉阶段，那么这就是用户故事拆分，因为用户故事拆分都是纵向拆分。不过往往团队会觉得拆分用户故事很难，浪费时间效果还不好，不如咬紧牙关搏一把全做进来。但往往事与愿违。最近参与一个团队为期两周的迭代。有一个类似的事情，分享一下如何使用用户故事拆分来应对。

## 需求描述

一个SaaS产品，有一个模块是一个数据源列表，每一个列表项都是一个数据源，存储对应的设置，例如FTP、SharePoint、Azure Blob都会有不同。现在的需求是，用户希望能够支持导出和导入功能。这样当用户在一个数据中心的部署配置成功后，可以将设置复制到其他数据中心，降低手动设置的麻烦与错误。

## 遇到的问题
这里描述的问题会适当简化难度，方便大家只关注拆分用户故事的方法。例如：
1. 导入数据源的时候，需要判断目的段是否已经存在同名的数据源设置，如果存在。需要考虑冲突解决方案，类似移动同名文件到一个folder内的场景。
2. 导入数据源的时候，如果不存在同名数据源，但是内部设置有冲突，例如数据源名称不同，但是内部却是同一个FTP地址，需要考虑冲突解决方案。
3. 当一次性批量导入多个数据源设置的时候，如果其中一个数据源设置导入失败，需要如何处置？回滚还是继续？

当团队在一周的迭代开发后，发现需要考虑的上面这么多场景，并要在交付时保证质量。顿感压力山大。

## 按照业务拆分用户故事
如果在团队感到焦虑的时候，硬要求团队按时完成，团队也无法保证交付质量，那后果就只能是牺牲客户的满意度。
既然无法完成所有场景，那就考虑完成一部分。但是拆分原则仍然是交付用户能够使用的功能，对用户来说有价值的功能。这时候想起了Mike Cohn的SPIDR用户故事拆分方法中的“R”——规则（[【可参考文章《五种简单高效的拆分用户故事的方法》】](https://brucetalk.com/2020/09/26/split-userstory/)）。这里的规则（”Rule“）就是业务规则，也就是从上面的场景中印发冲突的原因开始思考。于是我建议团队，将这个用户故事拆分成两个大场景：
1. 导入一个数据源设置。解决上面所有冲突的可能。但是指考虑每次一个数据源。
2. 导入多个数据源设置。考虑多个数据源的场景。因为已经覆盖了大部分的冲突场景。只需要考虑多个冲突时候的解决方案。

#1和#2是一个递进的关系，#2是#1的扩展。团队可以在规定时间分别完成有价值的功能，而且彼此互相支撑。有点像TDD思想里面，先开始一个简单的测试用例，然后再逐步扩展。而不是一开始就想着怎么实现一个复杂的测试用例。

## 总结
上面的拆分方式从用户的角度，一个一个导入和批量导入多个，解决的业务痛点是相同的，只是操作的繁杂度不同。在思考和实现的时候，如果同时思考解决方案，会陷入繁杂的实现细节中。通过思考业务规则，结合TDD的从最简单的一小步开始实现的思想，将用户故事进行拆分，先实现简单的，之后再逐步实现繁杂的部分。可以有效的降低工作难度，同时实现的过程还是逐步递进，层层支撑，这可以让团队更有信心来交付高质量的任务，同时也能保证在规定时间内完成。试想一下，如果批量导入多个数据源设置的时候，发现了无法解决的问题，那么团队至少可以交付导入一个数据源设置的功能，这样团队就不会陷入工作局促的困境。

*践行敏捷实践，让工作变得更美好。欢迎关注我的公众号，交流落地经验。*