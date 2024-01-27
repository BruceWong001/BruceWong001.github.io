---
layout:     post
title:      用户思维 VS 技术思维
subtitle:   
date:       2023-09-10
author:     Bruce Wong
header-img: img/patrick-tomasso-fMntI8HAAB8-unsplash.jpg
catalog: true
tags:
    - 敏捷
    - Agile
    - 随笔
    - Scrum
    - ProductOwner
---

工作中经常会遇到用户思维和开发思维的碰撞，尤其是开发和Product Manager之间。今天想来聊聊这个话题。 
## 案例描述  
一个在线教育类产品，会涉及到集成SCORM、xAPI这一类的第三方开发的学习包。而这些多媒体交互应用为了学习体验，一般会使用iframe的方式集成到现有的产品中。最近系统上线后，发现有一些学习包无法正常播放。客户抱怨不已，以往好用的功能突然间无法访问，影响了学生和老师的学习和工作。经过排查，发现是因为内容安全策略（CSP）的问题（文末会有CSP的基本解释）。简单说就是当存在跨域访问的时候需要提前设定允许访问的域名，否则浏览器默认会处于安全考虑阻止一切跨域的访问。在这个偏技术侧的问题的解决方式上，开发和PM的态度是不一样的。今天就来分享一下我所经历的这一讨论过程，以及其中的转变以及思考。   

**问题的效果**  
正常访问应该是如下效果：
![Scorm Works](/img/data/scormworks.PNG)

被CSP拦截后的效果如下：
![Scorm Failed](/img/data/scormfailed.jpeg)

**用户思维（PM思维）**  
在这个案例上，PM的思维是用户思维。站在用户的视角，他们的想法是，用户不应该关心这些技术问题，也不会去听你解释具体的原因，而是应该让用户专注于学习。所以他们的想法是，如何能够避免以后的系统，在添加其他SCORM包的时候，不会再遇到类似的问题。 

**技术思维**  
开发的思维是技术思维，站在技术的角度，他们的想法是，用最小的改动解决这个问题。这个问题是由于第三方的SCORM包的问题，而且这个问题是由于产品的安全策略导致的。那么提供一个手动配置安全策略的交互界面，让用户自己去设置安全策略最简单。一方面能够更灵活的应对各种位置情况，不需要每次都修改应用，同时如果由于修改设置导致系统出现安全漏斗，也可以由客户自己承担。  

双方对于这个问题的解决方式是不一样的，但是站在各自的角度去看都有道理。结果僵持不下。  

## 解决过程   
从上面两种思考视角看，技术思维倾向于让客户自己去处理遇到的问题，因为不同SCORM包需要设置的灵活。前提是客户有足够的技术背景和意愿来手动处理。毕竟软件工具是为了更方便的工作，而不是增加很多手动设置。而PM从用户思维出发，也没有问题，技术本身就应该让工作更简单，而不是需要更多的手动操作。第一轮的讨论显然矛盾突出，无法达成一致。  

接下来PM引导开发团队换位思考。“如果你是用户，你会怎么做？”当然前提是你是个非技术人员。这时候开发提出了一个新的方案——给出提示，让老师在添加SCORM包的时候读取帮助提示，让他主动去设置系统级别的CSP，避免学生使用的时候遇到问题。这个方案从开发的角度比之前的有所改进，不过从PM的角度看，很多系统提示用户并不会看，而且从交互的角度，应该让操作更自然，而不应该依赖提示。第二轮有所改进，但是还是无法达成一致。  

接下来，团队从如何让用户更容易的发现上传的SCORM包需要设置CSP的问题展开讨论。PM提出如果系统能够自动对SCROM包扫描，然后自动设置CSP，那么就不需要用户手动设置了。这个方案从用户的角度看，是最好的，不需要用户做任何操作，系统自动完成。而从技术的角度看，这个方案是可行的，但是需要很多的工作量，而且需要很多的测试，因为不同的公司制作的SCORM包，包的结构不同，所以测试无法覆盖所有情况，同时SCORM包的定义相对宽松，扫描的准确性无法保证。这个方案的投入产出比并不高。第三轮讨论，有了新的idea，但是还是无法达成一致。  

经过前面三轮讨论，可以看到这个问题的核心需求是：如何让老师尽早发现包有问题，进行设置，而不是在学生使用的时候。PM和开发团队达成一致，只要能解决这个需求，在技术可行和工作量能够接受的范围内，双发就能达成一致。基于此团队开始头脑风暴。最后的方案：
1. 上传SCORM包的交互变成一个向导式操作。  
2. 一共两步，都是必做步骤：第一步上传SCORM包，第二步预览。  
3. 在预览时候发现问题，提示用户需要设置CSP。  

至此，双发达成一致，问题解决。  

## 如何换位思考  
从上面的例子可以看到，最后的解决方案根本还是让用户自行设置CSP，但是在如何引导用户去设置的角度，应用程序做了一系列设计，让用户在使用过程中逐步发现需要设置，降低难度。同时，并不是每一次操作都需要设置CSP，也避免了对用户正常工作步骤的干扰。  
不过实际工作中，并不是每一次PM都能说服技术团队换位思考。所以一些工具可以让这个过程更加容易。今天介绍两个工具，可以根据需要在引导团队讨论的时候可以使用：同理心地图和用户旅程地图。  
当然还是那句话，工具不是万能的，他只是帮助团队更好的讨论和思考，避免思维定势和惯性思维。最终的结论还是团队集体智慧的产物。  

### 同理心地图
下图是同理心地图框架，团队将自己的思维切换到用户视角，尝试填写每一个区域的内容。从而从用户视角来思考。  
![Empathy map](/img/data/empthymap.png) 

### 用户旅程地图  
用户旅程地图，顾名思义是用户在旅程过程中的每个关节阶段的描述，如下图是一个孕妇乘坐高铁的旅程描述。将这个概念移动到软件领域，就是用户在使用软件的过程中的每个关节阶段的描述。在不同心里状态下的用户的行为和感受。同时团队可以寻找用户的痛点和机会点，从而设计更好的解决方案。
![Empathy map](/img/data/userjourneymap.jpeg)  

## 总结  
在今天技术差异并不是很大的应用领域，用户思维、业务思维是软件间竞争力的核心之一，软件的核心价值是为了让工作更简单，而不是增加更多的手动操作。所以在解决问题的时候，如果你是技术人员，那么请换位思考，如果你是PM也请换位思考。最好的方案来自于充分理解需求的团队的集体智慧。

## CSP背景知识  
CSP（Content-Security-Policy） 的主要目标是减少和报告 XSS 攻击。XSS 攻击利用了浏览器对于从服务器所获取的内容的信任。恶意脚本在受害者的浏览器中得以运行，因为浏览器信任其内容来源，即使有的时候这些脚本并非来自于它本该来的地方。

CSP 通过指定有效域——即浏览器认可的可执行脚本的有效来源——使服务器管理者有能力减少或消除 XSS 攻击所依赖的载体。一个 CSP 兼容的浏览器将会仅执行从白名单域获取到的脚本文件，忽略所有的其他脚本（包括内联脚本和 HTML 的事件处理属性）。

例如：一个网站管理者允许网页应用的用户在他们自己的内容中包含来自任何源的图片，但是限制音频或视频需从信任的资源提供者，所有脚本必须从特定主机服务器获取可信的代码。那你需要设置如下的CSP策略：
> Content-Security-Policy: default-src 'self'; img-src *; media-src media1.com media2.com; script-src userscripts.example.com  

## 参考引用  

[用户旅程地图](https://zhuanlan.zhihu.com/p/461436110)  
[内容安全策略（CSP）](https://developer.mozilla.org/zh-CN/docs/web/http/csp)  


*践行敏捷实践，让工作变得更美好。欢迎关注我的公众号，交流落地经验。* 