---
layout:     post
title:      Foundry Agent 加工具时，Configured 和 Catalog 到底有什么区别？
subtitle:   Microsoft Foundry 系列(5)
date:       2026-06-20
author:     Bruce Wong
header-img: img/aaron-burden-NXt5PrOb_7U-unsplash.jpg
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
---

如果你和我一样，最近在 Microsoft Foundry 里给 Agent 添加 Tool，那你一定会看到各种 Work IQ 工具。尤其是在 `Configured` 以及 `Catalog` 这两个 Tab 下面都会看到，特别在 `Catalog` 下面，更有细化的 Work IQ MCP、Work IQ Mail、Work IQ Teams。它们是不是同一个能力？

先看两张图。

![Foundry Add tool - Configured tab](/img/foundry/configuredtool.jpeg)

`Configured` 里有 File search、Code interpreter、Azure AI Search、Work IQ、SharePoint 等。

![Foundry Add tool - Catalog tab](/img/foundry/catalogtool.jpeg)

`Catalog` 里搜索 `iq`，又能看到 Work IQ Copilot、Work IQ Mail、Work IQ MCP 等。

我的理解是：**它们不是重复，而是不同接入形态。**

## 三个入口

`Configured` 更像你已经完成配置的工具列表，Foundry 的内置/托管工具（比如 File search、Code interpreter、Azure AI Search、Work IQ）通常不需要额外 endpoint，配置后会直接出现在这里。

`Catalog` 是公共或组织级 Tool Catalog，用于发现尚未配置的工具，很多工具是 Remote MCP，比如 Work IQ MCP、Work IQ Mail、第三方 SaaS。

`Custom` 是你自己的工具入口，比如 MCP server、OpenAPI、Function；在 Catalog 里也有一种 `Custom` 类型，指从 Azure Logic Apps connectors 转换而来的 custom MCP server。

Microsoft Foundry 文档说明，Agent 可以连接 MCP server 来扩展外部工具和数据源。参考：[Connect agents to Model Context Protocol servers](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/model-context-protocol)。

## Configured 里的 Work IQ

`Configured > Work IQ` 是 Foundry 内置的 Work IQ tool。你的 Agent 调用它后，把任务交给 Work IQ：`Foundry Agent -> Work IQ -> M365 工作上下文 -> 返回结果`。

Work IQ 官方文档提到，它支持 A2A、MCP 和 REST；其中 REST API 在文档中目前标注为 **coming soon**。A2A 更适合 agent-to-agent communication 和任务委托。参考：[Work IQ API overview](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/api-overview)。在 Foundry 里，`Configured > Work IQ` 对应 `work_iq_preview` 工具，使用 **A2A 协议 + On-Behalf-Of (OBO) 认证**把自然语言任务委托给 Work IQ 这个 peer agent，并不是你自己部署的 Foundry Agent。

适合场景：会议准备、邮件总结、项目上下文梳理。

## Catalog 里的 Work IQ MCP

`Catalog > Work IQ MCP` 则把 Work IQ 作为 Remote MCP server 暴露给 Agent。

官方 Work IQ MCP 文档说明，它提供 generic tools，让 Agent 读取、创建、更新、删除 M365 entities，也可以调用 Copilot 或指定 agent。参考：[Work IQ MCP overview](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/mcp/overview)。

通用 Work IQ MCP 官方列出 10 个 tools：`fetch`、`create_entity`、`update_entity`、`delete_entity`、`do_action`、`call_function`、`ask`、`list_agents`、`get_schema`、`search_paths`。参考：[Work IQ MCP tool reference](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/mcp/tool-reference)。

重点是：`create_entity`、`update_entity`、`delete_entity`、`do_action` 都可能产生写操作或副作用。

## Work IQ MCP 和 Mail / Teams

以下是我基于 UI 呈现和命名习惯的推断，官方文档目前尚未明确说明 `Work IQ Mail / Teams / Calendar` 与通用 `Work IQ MCP` 之间的严格关系：

```text
Work IQ MCP = 通用入口，覆盖多类 Microsoft 365 resource paths
Work IQ Mail / Teams / Calendar = 更窄的 workload-scoped MCP
```

不过，官方在 Foundry Toolbox 的“Add the Work IQ Tool”界面中，已经把选项明确区分为走 A2A 的 **Work IQ Chat** 和其余走 MCP 的 workload 选项（包括 Teams、Word、Outlook Calendar、Outlook Mail、M365 user profile、SharePoint、OneDrive）。参考：[Connect agents to Microsoft 365 with Work IQ](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/work-iq)。

但不要说它们是严格的“父子包含关系”。公开文档明确的是通用 Work IQ MCP 有 10 个 tools；对于 Work IQ Mail、Work IQ Teams，要以实际 `tools/list`、UI 选择、OAuth scope、租户策略和 approval 为准。

## Work IQ Copilot 和 ask

`Work IQ MCP` 里的 `ask` 可以向 Microsoft 365 Copilot 或指定 agent 提问。不传 `agentId` 时默认调用 built-in Microsoft 365 Copilot agent；传 `agentId` 时路由到指定 agent；`list_agents` 可以列出可用 agents。

如果你想从 Foundry Agent 显式调用 M365 Copilot 里的自定义 agent，公开文档里最明确的路径是 `list_agents + ask(question, agentId)`。对于 `Work IQ Copilot`，我没有看到公开文档说明它是否支持 `agentId`。

## 权限和风险

Work IQ 使用 Microsoft Entra ID delegated authentication，请求运行在 signed-in user 上下文中，并自动执行 Microsoft 365 permissions、sensitivity labels 和 compliance policies。参考：[Work IQ API overview](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/api-overview)。它不是用管理员权限扫描全租户，而是访问当前用户有权访问的 M365 资源。

但工具风险要单独看。`WorkIQAgent.Ask` 权限描述提到，它可能涉及 read and write access。参考：[Work IQ API permissions reference](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/permissions)。

在 Foundry 中使用 Work IQ 时，还有几点需要管理员提前准备：

- **Microsoft 365 Copilot license**：目前 Foundry Work IQ 集成要求调用者拥有 M365 Copilot license。参考：[Connect agents to Microsoft 365 with Work IQ](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/work-iq)。
- **Bring your own Entra app + admin consent**：需要 Entra Global Administrator 先在租户中 provision Work IQ service principal，再为应用注册授予 `WorkIQAgent.Ask` 的 admin consent，否则 Foundry 中不会出现 Work IQ 连接选项。
- **不支持 VNet/Private Endpoint**：当前 Foundry Work IQ 集成不支持虚拟网络隔离；如果项目是 VNet-restricted endpoint，则无法使用。
- **数据驻留**：Work IQ 检索数据不离开 Microsoft 365 tenant，数据位置遵循 M365 tenant 的数据驻留配置，而不是 Foundry project region。

我的判断：Work IQ 不应越权读取用户看不到的数据；Work IQ Mail 不一定只读，具体读写能力取决于该工具实际暴露的 resource paths 和当前用户在 M365 中的权限；写操作要当成高风险能力。

## 我会怎么选

快速总结 M365 工作上下文，选 `Configured > Work IQ`。只想问 Copilot，选 `Work IQ Copilot` 或 `Work IQ MCP ask`。需要指定可见 Copilot agent，用 `list_agents + ask(agentId)`。只需要邮件或 Teams，选 `Work IQ Mail` / `Work IQ Teams`。需要跨 Mail、Calendar、Teams、OneDrive、SharePoint，再考虑通用 `Work IQ MCP`。接公司内部系统，走 `Custom` MCP、OpenAPI 或 Function。

## 小结

`Configured` 更偏 Foundry 内置和托管体验，`Catalog` 更偏工具目录和 Remote MCP，`Custom` 是你自己的工具入口。放到 Work IQ 这个例子里：`Configured > Work IQ` 像任务委托入口；`Catalog > Work IQ MCP` 像 MCP 工具服务器；`Work IQ Mail / Teams / Calendar` 是更窄的 workload-scoped MCP；`Work IQ MCP ask(agentId)` 支持指定可见 agent。

不要只看名字，要看调用形态、工具范围和权限治理。能选窄工具，就不要直接选全域工具；涉及写操作时，最好先生成 draft，再让用户确认执行。

> 注：Work IQ 服务及 Work IQ MCP APIs 已于 2026 年 6 月 GA；Microsoft Foundry Agent Service 及 Foundry 工具目录核心框架也已 GA。但 Microsoft Foundry 中的 `work_iq_preview` 内置工具集成、Foundry MCP Server 云端托管版以及工具目录中的部分工具（如 Browser Automation、Computer Use 等）仍处于 public preview。本文基于 2026 年 6 月的公开文档，具体 UI、工具列表和实现细节可能随时间变化。

**理解 AI，用好 AI，让 AI 帮助自我进化，加油。**

## 相关链接

- [Work IQ overview](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/)
- [Work IQ API overview](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/api-overview)
- [Work IQ MCP overview](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/mcp/overview)
- [Work IQ MCP tool reference](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/mcp/tool-reference)
- [Work IQ API permissions reference](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/permissions)
- [Connect agents to Microsoft 365 with Work IQ](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/work-iq)
- [Agent tools overview for Microsoft Foundry Agent Service](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/tool-catalog)
- [Connect agents to Model Context Protocol servers](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/model-context-protocol)
