---
layout:     post
title:      Microsoft Foundry Agent 身份治理入门：Agent Identity、Blueprint 与 Managed Identity
subtitle:   Microsoft Foundry 系列（9）
date:       2026-08-09
author:     Bruce Wong
header-img: img/IMG_1513.WEBP
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
---

今天聊聊Microsoft Foundry如何对AI Agent进行权限治理的，当 Agent 从“回答问题”走向“调用工具、访问数据和执行动作”时，还有一个更基础的问题：**这次操作究竟是谁做的？**

很多团队会先关注模型、Prompt 和 Tool 是否好用，等到准备上线才开始补权限。结果往往走向两个极端：要么权限不够——开发环境能运行，生产环境出现 `403`；要么权限过多——为了赶进度不断加权限，Agent 拿着远超任务需要的身份运行。前者至少会报错，反而容易被发现；后者不报错，直到某天 Agent 把不该读的数据读了出来、写进了不该写的系统，或者替用户发出了不该发的邮件。而到那时再追查，往往连"这次操作代表用户还是代表系统"都说不清。

这篇先不讲配置步骤，而是给刚接触 Agent 治理的读者建立一张身份地图。

## 为什么 Agent 也需要身份治理？

普通聊天机器人主要生成内容，企业 Agent 还可能读取文件、更新 CRM、发送邮件，甚至串联多个 Tool 完成任务。模型不仅在回答，也在一定范围内决定“下一步调用什么”。

因此，企业不能只问“Agent 能否完成任务”，还要回答五个问题：

1. 谁发起了这次请求？
2. Agent 以谁的身份访问下游系统？
3. 这个身份最多可以做什么？
4. 哪些具体动作必须有人确认？
5. 出现异常时，能否审计、撤权和停用？

这五个问题分别对应 caller、identity、authorization、approval，以及 audit 与 lifecycle。Prompt 可以约束行为倾向，却不是权限边界；真正的边界仍要由身份系统、下游授权和确定性的控制来执行。

可以用一个简单例子理解。员工让 Agent “准备明天的客户会议”，这句话背后至少可能发生三件事：读取员工自己的邮件、从团队知识库查产品资料、把结果写回 CRM。三次调用看起来属于同一个任务，业务责任却不同：第一步是在代表员工，第二步可能代表组织，第三步会改变业务系统状态。如果只给 Agent 一把通用 API key，这些边界就会被压扁成“都能调用”。

更麻烦的是，“都能调用”的失败不是 `403` 式的报错。权限不足时系统会拒绝你；权限过多时系统会配合你——Agent 用同一把 key 读出其他员工的邮件、把内部资料写进客户可见的字段，下游服务只会照常返回 `200`。数据泄露在日志里看起来和正常调用一模一样。身份治理的目的，正是把这些边界重新拆开，让每一类操作都有自己的权限上限和审计归属。

## Foundry 中最容易混淆的四个身份概念

Microsoft Entra Agent ID 把 Agent 当作可以独立识别、授权和审计的主体。Foundry 会在 Agent 生命周期中创建和管理相关对象。围绕一次 Tool 调用，最容易混淆的是下面四个概念：

| 概念 | 它代表什么 | 主要作用 | 权限治理关注点 |
|---|---|---|---|
| End User Identity | 当前登录用户 | 提供交互式任务中的用户上下文 | 用户原有权限、delegated scope、consent 和租户策略 |
| Agent Identity | 运行中的 Agent | 让 Agent 作为独立主体取得 token、访问 Tool 或资源 | 给该 Agent 分配多大的 RBAC、app permission 或下游权限 |
| Agent Identity Blueprint | 一类 Agent 的治理对象 | 创建和管理相关 Agent Identity，承载分类与生命周期关系 | Blueprint 的 owner、策略、允许创建哪些 Agent Identity |
| Project Managed Identity | Foundry Project 的托管身份 | 支撑项目及平台操作，也可通过 federated credential 认证 Blueprint | 不要把项目级基础设施权限与 Agent 业务权限混为一谈 |

这里最重要的不是背名词，而是理解它们不在同一层：

```text
Project Managed Identity --认证--> Agent Identity Blueprint
Agent Identity Blueprint --创建和治理--> Agent Identity
Agent Identity --取得目标 audience 的 token--> Tool / Downstream Resource
```

这是对 Foundry Agent Identity token exchange 的简化图。按照[官方身份文档](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity)，在这条链路中，Project Managed Identity 用来认证 Blueprint；真正需要目标资源 RBAC 的 principal 是 Agent Identity。

但也不要把这个结论扩大成“Project Managed Identity 永远不能访问 Tool”。Foundry 的 MCP connection 也支持显式选择 `project-managed-identity` 认证；如果采用这种连接方式，就要把相应权限授予 Project Managed Identity。判断权限给谁，最终要看**当前 Tool connection 实际选择的认证方式**，而不是只看 Project 或 Agent 的名字。

## Connection 中的 key 是不是另一种 Agent Identity？

不是。Foundry Project Connection 可以保存 API key、bearer token、OAuth 配置或选择某种 Entra 身份认证。它解决的是“这个 Tool 如何完成认证”，但不一定为 Agent 提供独立、可治理的 Entra 身份。

例如，十个 Agent 共用同一把第三方 API key，下游日志可能只能看到这把 key，无法仅凭该日志区分是哪个 Agent 发起调用。此时至少还需要在应用层记录 Agent、Tool、参数和 correlation ID，并定义 key 的存储、轮换与撤销流程。能使用 Agent Identity 或用户委托身份时，通常更容易建立独立权限和审计边界；只能使用 key 时，则要把共享凭据的 blast radius 当作显式风险管理。

## Agent Identity 和传统应用身份有什么不同？

从 Microsoft Entra 的实现看，Agent Identity 仍然是 service principal，但它被明确标记和治理为 AI Agent 身份。[Microsoft Entra Agent ID 文档](https://learn.microsoft.com/en-us/entra/agent-id/what-are-agent-identities)强调，它的价值在于把 Agent 操作与员工、客户和普通 workload 的操作区分开，并支持对大量、可能快速创建和销毁的 Agent 做统一治理。

所以，Agent Identity 的意义不只是“又多了一个登录账号”，而是让组织可以明确：

- 这是 AI Agent 产生的操作，不是某个员工本人直接执行；
- 这个 Agent 有自己的 owner、sponsor、权限和生命周期；
- 可以单独查看、禁用或撤销它，而不是轮换一把被多个应用共用的 key；
- 不同 Agent 可以拥有不同权限边界和审计边界。

治理层面上，Microsoft Entra admin center 提供 Agent identities 清单，可以查看租户中的 Agent Identity，并进入 Blueprint 或禁用相关身份：

![Microsoft Entra admin center 中的 Agent identities 清单页面](/img/foundry/entra-admin-center-agent-identities.png)

*图片来源：[Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity)*

## 开发身份和生产身份为什么还要分开？

身份治理不仅决定“当前能不能调用”，还管理 Agent 从创建、测试、发布到停用的整个生命周期。

在同一个 Foundry Project 中，未发布的 Prompt Agents 默认共享 Project 的 Agent Identity。这适合快速试验，却意味着多个开发中 Agent 共用一套权限边界。如果某个 Agent 需要不同权限、独立审计或准备进入生产，就不应继续停留在共享身份上。

这套共享身份并不是抽象概念。在 Azure 门户打开 Project 的 Resource JSON，就能看到 `agentIdentity` 字段，以及它关联的 `agentIdentityId` 与 Blueprint ID：

![Foundry Project 的 Resource JSON 中的 agentIdentity 字段](/img/foundry/azure-agent-identity-json-view.png)

*图片来源：[Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity)*

发布为 Agent Application 后，Foundry 会为它创建独立 Blueprint 和 Agent Identity。新的身份不会自动继承共享开发身份的 RBAC。这种变化体现的正是治理原则：生产 Agent 应有清晰的权限与生命周期边界。至于发布后角色怎样迁移、新身份与共享身份有何不同，会放到下一篇的实战中展开。

## OBO 和独立身份，是两种“代表谁”的答案

Agent Identity 支持两类常见运行方式：

- **Attended / delegated access**：用户在场，Agent 通过 OBO（On-Behalf-Of）代表当前用户访问资源。有效权限还受到 delegated scope、consent、Conditional Access 和租户策略约束。
- **Unattended / application-only access**：没有用户在场，Agent 以自己的 Agent Identity 工作，权限由该身份的 RBAC、Microsoft Graph application permissions 或目标系统策略决定。

两者不是“安全”和“不安全”的区别，而是业务责任不同：读取“我的邮件”通常应保留用户上下文；后台处理队列则不能假装某个用户一直在线。

## 身份、权限和审批不是一回事

企业 Agent 的治理至少要拆成五层：

| 层次 | 回答的问题 | 典型机制 |
|---|---|---|
| Identity | 谁在调用？ | User Identity、Agent Identity、Managed Identity |
| Authorization | 最多能做什么？ | Azure RBAC、Graph permissions、下游 ACL |
| Tool policy | 能使用哪些动作？ | Tool allowlist、参数校验、read/write 分离 |
| Approval | 这一次是否应该执行？ | 人工确认、JIT elevation、业务审批流 |
| Observability & lifecycle | 做了什么，如何撤销？ | audit log、trace、owner、expiration、disable |

例如，Agent 拥有“创建工单”的权限，只能说明调用在授权边界内；它不能证明当前批量创建工单符合用户意图。Microsoft 的[最小权限建议](https://learn.microsoft.com/en-us/security/zero-trust/sfi/least-privilege-for-ai-agents)因此把身份、scope、Tool allowlist、approval、日志和撤销测试放在同一个治理框架中。

## 刚开始时，最少应该设计到哪一步？

可以用下面这棵简单的决策树判断：

```text
Agent 是否访问外部数据或调用 Tool？
├─ 否：先按普通 AI 应用保护入口、数据和日志
└─ 是
   ├─ 是否应该继承当前用户权限？是 -> 设计 OBO / delegated access
   └─ 是否代表系统后台执行？是 -> 使用独立运行身份和最小权限
       └─ 是否会写入、外发、删除或提权？是 -> 增加 Tool policy 与 approval
```

无论走哪条路径，只要进入生产，都应有明确 owner、权限清单、日志字段和停用办法。治理不是要求第一天就建立一套庞大的审批平台，而是避免用共享账号和宽权限，把未来必须回答的问题暂时藏起来。

## 三个常见误解

**误解一：有了 Agent Identity，就不需要 OBO。** 事实上，一个 Agent 仍可以在交互式场景中代表用户；Agent Identity 提供 Agent 自身的可识别性，OBO 则保留用户授权上下文。

**误解二：Project Managed Identity 就是 Agent Identity。** 前者是 Project 级托管身份，后者代表具体 Agent。二者可能都参与认证，但用途、scope 和生命周期不同。

**误解三：在 system prompt 中写“操作前先询问”，就完成了治理。** Prompt 不是强制授权机制。身份、RBAC、Tool policy 和 approval gate 必须在模型之外真正执行。

## 小结

理解 Foundry Agent 治理，可以先记住一句话：**Identity 说明谁在行动，Authorization 限制它最多能做什么，Approval 决定这一次该不该做，Observability 与 Lifecycle 负责事后说清楚并及时收权。**

权限治理处理的其实是两类方向相反的故障：权限太少，表现为 `403`，系统会大声报错；权限太多，表现为数据泄露，系统往往一声不吭。工程排错解决的是前者，身份治理真正要防的是后者。

**You can outsource your thinking, but you cannot outsource your understanding.**

## 相关阅读

- [Agent identity concepts in Microsoft Foundry](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity)
- [What are agent identities?](https://learn.microsoft.com/en-us/entra/agent-id/what-are-agent-identities)
- [Least privilege for AI agents](https://learn.microsoft.com/en-us/security/zero-trust/sfi/least-privilege-for-ai-agents)
- [Connect agents to MCP server endpoints](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/model-context-protocol)