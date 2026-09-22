---
layout:     post
title:      Microsoft Foundry Agent 治理入门：企业上线前先问五件事
subtitle:   Agent Governance 系列（1）
date:       2026-09-21
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

Agent 开始在企业里真正落地，治理问题紧跟着来：这些 Agent 由谁管、能访问什么、出了事怎么查？从这一篇开始，我会用一个系列来分享在 Microsoft Foundry 中如何做 Agent 的企业治理。

整个系列会以一个实际的企业 Agent 为背景，讲清楚具体怎么做，以及为什么这么做。

**Scenario：**销售团队的成员 Geeker 做了一个会议准备 Agent：它能读客户资料、整理摘要，也能调用工具发送邮件。演示成功后，能直接给全公司使用吗？

我们会先考虑五件事：
- 找不找得到它，谁负责；
- 谁能调用它，它能访问什么；
- 哪些动作要审批；
- 出了问题能否看清原因；
- 最后能否停用和收回权限。

这就是企业治理 Agent 的起点。

## 一、我们知道有哪些 Agent、谁负责吗？

Foundry 的管理视图（Control Plane）提供 **Operate > Assets > Agents** 清单，可查看所选 Azure 订阅中、受支持平台的 Agent，包括它属于哪个项目、当前版本和状态。平台团队可以先用这份清单找出“已经上线却没人再管”的 Agent。清单受订阅、平台支持和查看权限限制；没看到某个 Agent，不等于它不存在。[Foundry Agent 资产清单](https://learn.microsoft.com/en-us/azure/foundry/control-plane/how-to-manage-agents)

![Foundry Control Plane 中 Operate > Assets > Agents 的 Agent 清单](/img/foundry/control-plane-agents-inventory.png)
*图片来源：[Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/control-plane/how-to-manage-agents)*

找到以后，还要填上两个人：技术 **Owner** 负责配置和故障处理；业务 **Sponsor** 判断它是否还值得运行、是否仍需要访问客户数据。会议准备 Agent 的 Sponsor 可以是销售运营负责人，而不一定是写代码的人。[Entra Owner 与 Sponsor](https://learn.microsoft.com/en-us/entra/agent-id/agent-owners-sponsors-managers)

如果公司还有 Copilot Studio 或其他平台的 Agent，Agent 365 Registry 提供更广的组织级盘点视角；它与 Foundry 按订阅查看的清单不是一回事。[Foundry 与 Agent 365 集成](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-365-integration)

## 二、谁能调用它，它又能读写什么？

这里有两道门。第一道门是 **谁能调用 Agent**：Foundry 端点可以用 Microsoft Entra 身份和 Foundry 权限控制调用方。第二道门是 **Agent 能访问什么**：读 CRM、写 Storage 等操作仍由 Agent 自己的身份（Agent Identity）或用户授权，以及目标系统的权限决定。允许销售人员调用 Agent，不等于允许 Agent 读取所有客户资料。[Foundry 端点授权](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/configure-agent) · [Agent Identity](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity)

还要确认用户接触的是哪个版本。Foundry 的新 Agent 模型在创建时就有稳定端点和独立身份，端点默认使用最新版本。若生产变更需要先评估、再放行，应把端点固定到已批准的版本，否则一次新建版本就可能改变用户实际得到的行为。[配置 Agent 版本](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/configure-agent)

> **旧项目提醒：**早期 Foundry Agent 使用另一套发布模型：开发中的 Agent 可能共享项目身份，发布后才获得独立身份。现在新旧模型可能并存。检查权限或迁移时，先确认 Agent 实际属于哪一类，不要按“发布了吗”猜它的身份。[官方迁移指南](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/migrate-agent-applications)

![Foundry 中查看 Agent 身份和端点的示意图](/img/foundry/identity_endpoint.png)

## 三、它能调用哪些工具，哪些动作要人同意？

Foundry **Toolbox（工具箱）** 可以集中管理 Agent 使用的工具及其认证方式。对会议准备 Agent，可以把“查客户资料”和“发邮件”看成两种不同风险：前者需要限制读取范围；后者可能把信息送出企业，应核对收件人，并对高影响操作设置真正能暂停执行的审批。[Foundry Toolbox](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/toolbox-overview)

在提示词里写“发邮件前先问我”，不等于邮件接口真的会等待批准。最终仍要由运行程序或下游 API 执行授权与审批。连接外部工具时，还应查清数据会送到哪里、由谁保存。[微软的 Agent 共同责任模型](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility-ai-agent)

![微软的 Agent 共同责任模型](/img/foundry/ai-agent-shared-responsibility.svg)
*图片来源：[Microsoft Learn](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility-ai-agent)*

## 四、出问题时能查到原因吗？

Foundry **Trace（运行轨迹）** 记录 Agent 运行过程，能帮助查看出错位置和工具调用。要使用它，需要把 Foundry 项目连接到 Application Insights，并安排好日志的查看权限与敏感数据处理。只看到 Agent 最终答了什么，通常不足以解释它为什么读取了某位客户的资料。[Foundry Trace 配置](https://learn.microsoft.com/en-us/azure/foundry/observability/how-to/trace-agent-setup)

![Foundry 中查看 Agent 运行轨迹的 Traces 视图](/img/foundry/agent-trace-detail.png)
*图片来源：[Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/control-plane/how-to-manage-agents)*

上线前也要试几次“应该失败”的情况：让没有权限的人请求其他区域的客户资料，拒绝一次发信审批，或给 Agent 一段带恶意指令的检索内容。Foundry 评估和 AI Red Teaming 可帮助发现问题；CRM 自己的权限检查仍需单独验证。[Foundry AI Red Teaming](https://learn.microsoft.com/en-us/azure/foundry/concepts/ai-red-teaming-agent)

## 五、需要停用时，能停在哪里？

如果 Agent 的回答或工具调用出现异常，新模型的 Foundry Agent 可以禁用端点，阻止新的调用；Entra 中禁用 Agent Identity，则阻止这个身份继续获得访问令牌。两者作用不同。旧的 Agent Application 还有停止部署的操作，所以应在上线时就写清楚：发生事故时，谁有权执行哪一种停用。[Foundry 停用方式](https://learn.microsoft.com/en-us/azure/foundry/control-plane/govern-agent-infrastructure-entra-admin) · [Entra Agent 管理](https://learn.microsoft.com/en-us/entra/agent-id/manage-agent-identities-admin)

![Control Plane 中通过 Update Status 停止 Agent](/img/foundry/control-plane-agent-stop.png)
*图片来源：[Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/control-plane/how-to-manage-agents)*

停用之外，还要避免权限长期留着没人复核。Entra ID Governance 的 **Access Package（权限包）** 可以给 Agent 的访问设置审批和到期时间；业务 Sponsor 再决定是否申请延期。这部分会在下一篇展开。[Agent Identity 治理概览](https://learn.microsoft.com/en-us/entra/id-governance/agent-id-governance-overview)

这五件事可以写在同一张上线记录里：Agent 和版本、Owner 与 Sponsor、调用与数据权限、工具审批、Trace、停用负责人。团队若无法填出其中某一项，就知道下一步该补什么，而不必先搭建一套庞大的治理流程。

**You can outsource your thinking, but you cannot outsource your understanding.**

## 官方参考

- [Microsoft Foundry Control Plane](https://learn.microsoft.com/en-us/azure/foundry/control-plane/overview)
- [Foundry 新旧 Agent 模型](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/migrate-agent-applications)
- [配置 Foundry Agent](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/configure-agent)
- [Microsoft Entra Agent ID 治理](https://learn.microsoft.com/en-us/entra/id-governance/agent-id-governance-overview)
- [AI Agent 共同责任模型](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility-ai-agent)