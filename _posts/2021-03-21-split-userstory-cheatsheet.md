---
layout:     post
title:      用户故事拆分速查手册(译)
subtitle:   Story Splitting Cheat Sheet
date:       2021-03-21
author:     Bruce Wong
header-img: img/patrick-tomasso-Oaqk7qqNh_c-unsplash.jpg  
catalog: true
tags:
    - 敏捷
    - Agile
    - User Story
    - 译文
---

当我们拆分用户故事的时候总会遇到各种各样的情况，有没有一些简单的例子让我们借鉴呢？最近看到了一个国外的速查手册，简单易懂，分享给大家。
## INVEST 模型  
用户故事应该遵循如下约定: 
+ Independent: 故事之间尽可能彼此独立的/低依赖的  
+ Negotiable: 故事并不是需求文档，如果有需要，可以进一步讨论的  
+ Valuable：有业务价值的  
+ Estimable：可以估算的  
+ Small：大小足够在允许时间内完成  
+ Testable：可以测试的  

## 用户故事拆分模式  
#### 按工作流步骤拆分  
作为一个网站内容管理员，我可以在公司网站上发布一篇新闻。  
##### 可拆分成：
...我可以直接在公司网站发布新闻。  
...我可以发布带有编辑评论的新闻。  
...我可以发布带有法律评论的新闻。  

#### 按业务规则拆分  
作为一个用户，我可以灵活地选择日期来搜索航班信息。 
##### 可拆分成：  
...可以选择从第X到第Y天之间。  
...可以选择12月的某个周末。  
...可以选择从X天加减N天。    
#### 按照主工作量拆分  
作为用户，我可以用VISA、万事达卡、大来卡或美国运通卡支付机票费用。  
##### 可拆分成：  
...我可以用其中一种信用卡付款(VISA、万事达卡、大来卡、美国运通卡)。  
...我可以用全部四种信用卡付款(VISA、万事达卡、大来卡、美国运通卡)。  

#### 按业务简单或复杂的方式拆分  
作为一个用户，我可以搜索两个目的地之间的所有航班。  
##### 可拆分成： 
...指定目的地之间最大停靠的站数。  
...包括目的地附近的机场。  
...使用灵活的日期。  
...等等。  

#### 按不同的数据拆分  
作为网站内容管理员，我可以创建新闻。  
##### 可拆分成：  
...使用英语。  
...使用日语。  
...使用阿拉伯语。  
...等等。  

#### 按输入数据的方式拆分  
作为一个用户，我可以搜索两个目的地之间的航班。
##### 可拆分成： 
...通过简单的输入日期。  
...通过漂亮的日历控件选择日期。  

#### 通过延迟性能拆分  
作为一个用户，我可以搜索两个目的地之间的航班。  
##### 可拆分成： 
... 能获得搜索结果，不过显示缓慢。会显示“正在搜索”的等待动画。  
...在5秒钟得到搜索结果。  

#### 按照操作来拆分(例如：增删改查)  
作为一个用户，我可以管理我的账户。  
##### 可拆分成： 
...我可以注册一个账户。  
...我可以修改我账户的设置。  
...我可以注销我的账户。  

#### 研究一个新领域  
作为一个用户，我能够用信用卡支付。  
##### 可拆分成： 
...研究信用卡处理流程。  
...实现信用卡处理(作为一个或多个故事)。  

Copyright ©2009 Humanizing Work, LLC  

### 参考链接
- [Story Splitting Cheat Sheet](https://3p72t53zzqr23puj171gfblg-wpengine.netdna-ssl.com/wp-content/uploads/2021/01/Story-Splitting-Cheat-Sheet.pdf)
