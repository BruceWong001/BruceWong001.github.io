---
layout:     post
title:      Microsoft IQ 与企业级 Context Engineering：四层上下文解析
subtitle:   Microsoft Foundry 系列(4)
date:       2026-06-14
author:     Bruce Wong
header-img: img/IMG_1513.WEBP
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
---

最近在用 Microsoft Foundry 开发 Agent 时，我发现一个容易混淆的地方：微软有一系列带 IQ 的概念，比如 Work IQ、Fabric IQ、Foundry IQ、Web IQ。它们都像是“给 Agent 提供上下文”，但补的上下文并不一样。

这其实指向了企业 Agent 工程里的核心问题：**context engineering**。如果说 prompt engineering 是“怎么问模型”，那 context engineering 就是“**给模型什么上下文**”。Microsoft IQ 可以看作是微软在这个方向上的企业级实现：把组织内外分散的上下文，封装成四层可被 Agent 消费的智能层。

先说结论，Microsoft IQ 包含四个能力：

- **Work IQ**：理解员工如何工作。
- **Fabric IQ**：理解业务数据和业务实体。
- **Foundry IQ**：理解企业知识、政策和权威文档。
- **Web IQ**：理解外部世界的新鲜信息。

## 四层上下文分别解决什么问题？

企业 Agent 的上下文通常来自四类来源，Microsoft IQ 每层对应一类：

| 上下文来源 | 对应 IQ | 核心问题 | 典型表示 |
|---|---|---|---|
| 人的活动和关系 | Work IQ | 谁、在什么时候、和谁、做了什么 | Microsoft Graph、协作信号 |
| 业务事实和指标 | Fabric IQ | 业务当前是什么状态 | Semantic model、ontology、Graph |
| 企业知识和规则 | Foundry IQ | 组织里权威资料怎么说 | Knowledge base、agentic retrieval |
| 外部实时信息 | Web IQ | 外部世界正在发生什么 | Web search、fresh content |

## Work IQ：理解“人是怎么工作的”

Work IQ 是 Microsoft 365 侧的 workplace intelligence layer。它理解 work context、relationships 与 patterns，让 Agent 比单纯 connector 更快、更准确地回答问题。

从 context engineering 角度看，Work IQ 的核心是**以“人 / 关系 / 时间”为中心的活动信号**。它回答的不是“文档里写了什么”，而是“**谁、在什么时候、和谁、做了什么**”。

比如：最近和客户有哪些邮件往来？明天会议前应该看哪些文档？Teams 里大家围绕某个项目讨论了什么？

Work IQ 依托 Microsoft 365 tenant data，包括 SharePoint、OneDrive、Outlook、Teams 以及 Microsoft Graph 中的协作信号，并尊重 Microsoft 365 权限和组织边界。它特别适合员工生产力类 Agent：会议准备、邮件总结、项目上下文梳理等。

## Fabric IQ：理解“业务当前是什么状态”

Fabric IQ 是 Microsoft Fabric 里的 IQ workload。官方文档说：Fabric IQ provides context on the state of your business。它不是简单让 Agent 查表，而是把数据提升到业务语言层，让人和 Agent 围绕业务概念、目标和规则理解数据。

从 context engineering 角度看，Fabric IQ 解决的是“**业务数据如何被 Agent 正确理解**”。企业数据原本在表、字段、schema 里，但业务关心的是：Customer 是谁，Shipment 和 Order 是什么关系，某个 KPI 怎么算，异常时该触发什么动作。

Fabric IQ 通过 OneLake、Power BI semantic models、ontology、Graph、data agent、operations agent 等，把这些业务概念、关系、规则和动作组织起来。它适合分析型和运营型 Agent：经营分析、供应链异常判断、指标解释、实时运营响应。

## Foundry IQ：理解“企业知识在哪里”

Foundry IQ 更偏向 Agent 的托管知识层。Agent 需要来自分散企业内容的上下文，但模型有知识截止时间，也不能自己访问企业私有数据。Foundry IQ 可以创建 configurable、multi-source knowledge base，让 Agent 基于组织数据给出 permission-aware responses。

从 context engineering 角度看，这和 RAG 很接近，但不是手写一个简单向量库。Foundry IQ 的知识库可以连接 Azure Blob Storage、SharePoint、OneLake 和 public web data 等来源，并用 agentic retrieval（查询改写、多轮检索、来源排序、引用生成等）返回带引用的 grounded answers。

它解决的是“**Agent 如何可靠访问企业知识**”的问题：公司政策在哪里？产品文档怎么解释？历史方案里有哪些相关内容？回答能不能给出引用？

它适合知识问答、政策助手、产品文档助手、售前方案助手等场景。

### 容易混淆：Foundry IQ 和 Fabric IQ 都提到 OneLake，区别在哪？

- **Fabric IQ** 处理的是 OneLake 里已经被建模为业务语义的数据（表、指标、Graph、语义模型）。它让 Agent 懂“业务事实”。
- **Foundry IQ** 处理的是 OneLake 里作为企业知识来源的文档、模板、方案等非结构化内容。它让 Agent 懂“企业说法 / 规则”。

一个是“业务数据层”，一个是“企业知识层”。

## Web IQ：理解“外部世界发生了什么”

Web IQ 是第四类能力。官方 Microsoft IQ 文档说，Web IQ provides AI systems and agents with fresh, real-world intelligence from across the web。

从 context engineering 角度看，它解决的是“**外部正在变化的信息**”：最新行业新闻是什么？竞争对手发布了什么？新政策、新法规、新漏洞有什么影响？

如果说前三者更多在组织内部找上下文，Web IQ 就是在组织外部补充变化。

### 容易混淆：Foundry IQ 也能接 public web data，和 Web IQ 不是重复吗？

不重复。

- **Foundry IQ 里的 web 数据**是企业主动 curate 过的外部资料，比如被抓取进知识库的竞品白皮书、监管页面、行业报告。它已经被纳入企业知识边界。
- **Web IQ** 是实时、开放、无需预收录的通用网络情报，补充的是“此刻外部世界正在发生什么”。

一个是“被企业管起来的外部资料”，一个是“实时外部世界”。

## 四者的区别

| IQ | 关注点 | 典型问题 | 典型数据 | Agent 作用 |
|---|---|---|---|---|
| Work IQ | 员工如何工作 | 我昨天和客户聊到哪了？ | 邮件、会议、Teams、文件、人员关系 | 理解工作上下文 |
| Fabric IQ | 业务当前状态 | 这个客户的续约风险是多少？ | OneLake、语义模型、Ontology、Graph、实时/分析数据 | 理解业务概念和指标 |
| Foundry IQ | 企业知识在哪里 | 我们的服务协议对这类问题怎么说的？ | 文档、知识库、SharePoint、Blob、OneLake、企业 curate 的 Web 资料 | 获得可引用的权威知识 |
| Web IQ | 外部世界发生什么 | 客户最近有没有被行业负面新闻波及？ | Web 信息、新闻、公开网页、实时外部资料 | 获得新鲜的外部上下文 |

## 一个 context engineering 实例

假设做一个客户会议准备 Agent：

> 帮我准备明天和 Contoso 的会议。

这不是一次简单的 prompt，而是一次**多层上下文的编排**：

1. 用 **Work IQ** 拉取最近和 Contoso 相关的邮件、会议、Teams 讨论，建立“人”和“时间线”上下文。
2. 用 **Fabric IQ** 查询 Contoso 的业务状态，比如 ARR、订单、风险、续约、服务工单趋势。
3. 用 **Foundry IQ** 查询产品文档、客户方案、合同政策和交付模板。
4. 如果需要，用 **Web IQ** 补充 Contoso 最近的公开新闻、行业动态或竞争信息。
5. 最后由 Agent 整理成 briefing、风险点和建议问题。

关键不是“哪个 IQ 更强”，而是它们补的上下文不同。例如，如果你只接 Foundry IQ 查文档，却不知道客户最近在 Teams 里表达过强烈不满，Agent 的回答很可能是“正确但脱离上下文”的。

## 小结

Agent 真正难的地方，不只是模型会不会推理，而是它有没有正确的上下文。微软把这些上下文拆成不同层次：

- **Work IQ** 让 Agent 像理解同事一样理解工作。
- **Fabric IQ** 让 Agent 像业务专家一样理解数据。
- **Foundry IQ** 让 Agent 像知识管理员一样找到权威资料。
- **Web IQ** 让 Agent 像研究员一样关注外部变化。

从 context engineering 的角度来看，Microsoft IQ 的核心价值在于：**把企业里原本分散、异构、权限复杂的上下文，封装成 Agent 可以直接消费的标准化能力**。你不需要自己写 Graph connector、自己搭语义层、自己维护向量库、自己对接网络搜索，而是按场景选择合适的 IQ 层。

如果企业 Agent 只接了模型，很容易变成一个“很会说话但不了解组织”的助手。当它能同时理解工作、业务、知识和外部变化时，才更接近真正可落地的企业 Agent。

---

## 相关链接

- [Microsoft IQ documentation](https://learn.microsoft.com/en-us/microsoft-iq/)
- [Work IQ overview](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/)
- [What is Fabric IQ?](https://learn.microsoft.com/en-us/fabric/iq/overview)
- [What is Foundry IQ?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-foundry-iq)
- [OneLake overview](https://learn.microsoft.com/en-us/fabric/onelake/onelake-overview)
