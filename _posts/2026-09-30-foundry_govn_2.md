---
layout:     post
title:      Microsoft Foundry Agent 权限生命周期：谁来批准，何时收回？
subtitle:   Agent Governance 系列（2）
date:       2026-09-30
author:     Bruce Wong
header-img: img/IMG_1861.jpeg
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
    - Agent Governance
    - AI Security
---

[上一篇](/2026/09/21/foundry_govn/)介绍了企业上线 Agent 前要问的五件事。这一篇接着看其中一个问题：**Agent 获得的权限，应该保留多久？**

继续用销售团队成员 Geeker 的会议准备 Agent。它在 Microsoft Foundry 中运行，需要读取客户资料，帮助销售准备会议。假设团队先试用 30 天：谁来批准这份读取权限？试用结束后，还要不要保留？

这就是权限生命周期治理：把“授予访问”变成一件有负责人、有期限、到期会重新判断的事。


## 一、先确认：权限到底给了谁？

在 Foundry 中找到会议准备 Agent 后，Geeker 首先要确认它调用客户资料工具时使用什么身份。本篇讨论的是：**工具使用 Agent 自己的 Entra Agent Identity 访问资源**。如果工具实际使用用户授权或另一套凭据，就要沿着那条授权路径治理，不能只检查 Agent Identity。[Foundry Agent Identity 与工具认证](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity)

Foundry 的新 Agent 模型在创建时就有独立身份和稳定端点。旧模型则可能在开发阶段共享项目身份，发布为 Agent Application 后才有独立身份。两者可能并存，不能单凭“是否发布”判断权限给了谁。[Foundry 新旧模型说明](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/migrate-agent-applications)

本例使用新模型的独立身份。身份确认后，再分清两件事：Agent 可以长期存在，但读取客户资料的权限只批准 30 天。到期收回这份读取权限，不需要同时删除 Agent。

## 二、谁负责使用，谁负责批准？

第一篇提到了 Owner 和 Sponsor。放进这个场景，就容易理解了：

| 角色 | 在会议准备 Agent 中负责什么 |
| --- | --- |
| 技术 Owner，例如 Geeker 或平台工程师 | 核对身份、配置工具、排查访问失败 |
| 业务 Sponsor，例如销售运营负责人 | 说明为什么需要客户资料，判断试用后是否继续使用 |

Owner 和 Sponsor 的分工来自 [Entra Agent ID 的责任模型](https://learn.microsoft.com/en-us/entra/agent-id/agent-owners-sponsors-managers)。人员调岗或离职时，也要有人接手这些责任。

Sponsor 提出业务需要，并不自动拥有批准客户数据访问的权力。在这个示例里，我们另外指定客户数据负责人作为审批人：销售运营解释用途，数据负责人决定是否放行。

## 三、用 Access Package 管一份 30 天的读取权限

**Access Package（权限包）**可以把一组资源权限，以及谁能申请、谁来审批、何时到期等规则放在一起。Agent 获得的那一次分配叫 **Assignment**，可以理解为“这份权限具体给了谁、有效到哪天”的记录。[Access Package 基本概念](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-create)

配置入口在 Entra 管理中心的 **ID Governance > Entitlement management > Access packages**。Foundry 负责运行 Agent；这份访问的审批和期限在 Entra 中管理。

我们先设计一个名为 `Meeting Preparation - Read` 的权限包：

| 设置 | 本例的选择 |
| --- | --- |
| 资源 | 一个客户资料只读安全组的成员资格 |
| 获得访问的身份 | 会议准备 Agent 实际使用的 Agent Identity |
| 申请人 | 销售运营负责人，以 Sponsor 身份代为申请 |
| 审批人 | 客户数据负责人 |
| 有效期 | 30 天 |
| 续期规则 | 允许申请延期，但需要重新审批 |

这里有一个前提：客户资料接口已经支持用这个安全组判断 Agent 的读取范围。**把组放进权限包，不会自动替 CRM 建立授权规则。**具体哪些客户能读、能否写入，仍要由目标系统落实并验证。

面向 Agent 的权限包支持安全组成员资格、允许的 Entra 角色和 OAuth API 权限，但应用角色、SAP 角色和 SharePoint 站点角色不支持；包含这些资源类型的员工权限包也不能直接复用。[Agent Access Package 支持范围与配置](https://learn.microsoft.com/en-us/entra/agent-id/agent-access-packages)

![Access Package 要求审批和申请理由的设置](/img/foundry/access-package-approval.png)

*官方通用配置界面：要求审批，并要求申请人说明理由。来源：[Microsoft Learn](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-create#specify-approval-settings)。*

这个读取包不包含发邮件权限。即使以后批准了发信权限，“是否允许发送这封邮件”仍是另一层运行时审批，需要程序或下游 API 真正拦住未批准的操作。[Agent 共同责任模型](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility-ai-agent)

## 四、30 天到了，会发生什么？

到期前，已设置的 Sponsor 会收到通知。如果策略允许延期，Sponsor 可以说明继续使用的理由并提出申请。本例要求重新审批：客户数据负责人同意后，访问才继续保留。

如果不申请延期，Assignment 会到期，由它管理的访问随之回收。Agent Identity 本身仍然存在。[Agent 访问到期与续期](https://learn.microsoft.com/en-us/entra/id-governance/agent-id-governance-overview#assigning-access-to-agent-identities)

![Access Package 分配的到期设置](/img/foundry/access-package-expiration.png)

*官方通用配置界面展示按天设置到期时间；截图中的 365 天是文档示例，本例应设置为 30 天。来源：[Microsoft Learn](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-create#specify-a-lifecycle)。*

## 五、怎样确认这套规则真的起作用？

Geeker 可以先在测试环境完成一次小范围验证：

1. **授权前**：用实际工具身份请求客户资料，确认访问被拒绝。
2. **授权后**：确认允许范围内的资料可读，范围外的资料和写入操作仍被拒绝。
3. **到期后**：不申请延期，再次调用接口，核对分配状态、组成员资格及实际访问结果。

如果出现 `403`，先查当前身份、权限分配和资源端授权，不要为了“让演示继续跑”直接加一个更大的角色。

排查时，把 Foundry 项目连接到 Application Insights，通过 **Trace** 查看工具调用及结果；再结合 **Entra 登录日志**核对身份的认证活动。两类记录提供不同证据，Trace 本身不会替你收回权限。日志也应限制查看范围，避免暴露客户资料。[Foundry Trace 配置](https://learn.microsoft.com/en-us/azure/foundry/observability/how-to/trace-agent-setup) · [Entra Agent ID 日志](https://learn.microsoft.com/en-us/entra/agent-id/sign-in-audit-logs-agents)

对会议准备 Agent，先把这一份 30 天读取权限管好，就有了可以检查的起点：哪一个身份在用、谁批准、何时到期，以及到期后是否真的不能再读。

下一篇再展开 **Agent Identity 与 Blueprint**：哪些设置属于单个 Agent，哪些影响同一 Blueprint 下的身份。

**You can outsource your thinking, but you cannot outsource your understanding.**

## 官方参考

- [Foundry 新旧 Agent 模型与迁移](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/migrate-agent-applications)
- [Entra Agent ID：Owner、Sponsor 与 Manager](https://learn.microsoft.com/en-us/entra/agent-id/agent-owners-sponsors-managers)
- [为 Agent Identity 配置 Access Package](https://learn.microsoft.com/en-us/entra/agent-id/agent-access-packages)
- [Agent Identity 治理、续期与许可要求](https://learn.microsoft.com/en-us/entra/id-governance/agent-id-governance-overview)
- [创建 Access Package：审批与到期设置](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-create)