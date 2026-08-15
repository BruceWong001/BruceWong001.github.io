---
layout:     post
title:      Microsoft Foundry Agent 权限治理：用 OBO、RBAC 与 Approval 设计会议准备 Agent
subtitle:   Microsoft Foundry 系列（10）
date:       2026-08-15
author:     Bruce Wong
header-img: img/IMG_1513.WEBP
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
---

上一篇[《Microsoft Foundry Agent 身份治理入门：Agent Identity、Blueprint 与 Managed Identity》](/2026/08/09/foundry_identity/)建立了 User Identity、Agent Identity、Agent Identity Blueprint 与 Project Managed Identity 的身份地图。这一篇把身份模型落到权限账本、发布迁移和负向测试，用一个具体场景回答三个实践问题：

1. 什么时候应该用 OBO，什么时候应该用 Agent Identity？
2. 为什么开发环境正常，发布后可能突然 `403`？
3. 已经有 RBAC，为什么高风险 Tool 仍需要 approval？

## 场景：客户会议准备 Agent

假设销售团队希望做一个 Agent，完成三类任务：

- 销售主动发起：“帮我准备明天与 Setoro 的会议”，Agent 读取该用户能访问的邮件、日历和文档。
- 由企业调度器每天凌晨触发后台任务，通过支持 Agent Identity 的 MCP Tool 生成团队级客户摘要，并写入指定 Azure Storage 容器。
- 用户确认后，把会议摘要发送给参会人，或更新 CRM 记录。

如果把所有能力交给一个共用 service principal，功能也许能跑，但权限边界会非常模糊。我会按“谁代表谁、读什么、写什么、谁批准”拆成三条路径：

| 任务 | 运行身份 | 权限来源 | 是否需要审批 |
|---|---|---|---|
| 读取当前销售的邮件和会议 | OBO：User + Agent 上下文 | 用户权限、delegated scope、consent 与租户策略的交集 | 通常不逐次审批，但应限制为只读 Tool |
| 后台生成团队摘要并写 Storage | Agent Identity | Tool connection 支持 Agent Identity，且目标资源 RBAC 授给当前 Agent Identity | 窄范围自动执行；异常或越界参数应阻断 |
| 外发邮件、批量更新 CRM | OBO 或专用写入身份，取决于业务责任 | 用户委托权限或专用 application permission | 建议对高影响动作显式审批 |

这张表不是部署后的文档，而应该在配置 Tool 之前就写出来。每新增一个动作，都先补齐 principal、resource、action、scope、approval 和 audit 六个字段。如果团队无法回答其中任何一项，说明当前设计还没有形成可执行的权限边界。

表中的身份是目标设计，不代表每一种 Tool 都支持对应认证方式。真正配置时，还要核对 Tool 文档和 connection 的 auth type；如果只能使用 key、OAuth passthrough 或 Project Managed Identity，就应把共享范围、轮换、撤销和审计风险写入同一份权限账本。

## 路径一：用户在场时保留用户边界

交互式任务可以简化为：

```text
Sales User Token
    -> Foundry Agent Service
    -> OBO Token（Agent + delegated user permissions）
    -> Microsoft 365 / Downstream Service
```

OBO 是 On-Behalf-Of，也就是“代表用户”。它不等于把用户所有权限复制给 Agent。最终有效权限还要经过应用申请的 delegated scope、用户或管理员 consent、Conditional Access 和下游系统授权。前提是调用方以用户的 Entra token 调用 Agent API——带 OBO Tool 的 Agent 不接受 API key 调用。

在这个例子里，销售 A 不应因为使用 Agent 就读到销售 B 的私人邮件。OBO 的价值，是让 Agent 的读取边界继续跟随当前用户，而不是为了方便统一换成一个高权限后台账号。

## 路径二：后台任务使用独立 Agent Identity

由企业调度器触发的汇总任务没有登录用户，不能依赖 OBO。它应以自己的 Agent Identity 运行，并且只获得完成任务所需的权限。这里假设 Agent 通过支持 Agent Identity 认证的 MCP Tool 访问下游资源；如果采用其他 Tool 类型，应重新核对它实际支持的认证方式。

以 Prompt Agent 的 Agent Identity token exchange 为例，可以简化为：

```text
Project Managed Identity -> Blueprint -> Agent Identity Token
Agent Identity Token -> target audience token -> MCP / Downstream Resource
```

按照[Foundry Agent Identity 文档](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity)，在这条链路中，目标资源 RBAC 应授予 Agent Identity，而不是用于认证 Blueprint 的 Project Managed Identity。

例如，该 Agent 确实需要读写某个 Storage Account 时，可以按官方示例把角色授给当前 `agentIdentityId`：

```bash
az role assignment create \
  --assignee "<agentIdentityId>" \
  --role "Storage Blob Data Contributor" \
  --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

若新建的 Agent Identity 因 Graph 同步延迟导致 `--assignee` 解析失败，可改用 `--assignee-object-id "<agentIdentityId>" --assignee-principal-type ServicePrincipal`。

[`Storage Blob Data Contributor`](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/storage#storage-blob-data-contributor) 不只是“写入”权限，它包含对 Blob 容器和 Blob 的读取、写入与删除。若任务只允许追加报告、不允许读取或删除历史文件，应该继续评估更窄的容器 scope、带条件的角色分配、业务 API 或 custom role，而不是因为示例使用了内置角色就直接照搬到生产。

这里还有一个容易被概念图隐藏的例外：MCP connection 也支持显式选择 `project-managed-identity`。如果连接配置选择的就是这种 auth type，目标资源的 RBAC 自然要授给 Project Managed Identity。排错时应查看 Tool connection 的实际认证方式。

## 从开发到上线，我会按什么顺序配置？

为了避免权限越加越乱，我会把部署过程固定成下面五步：

1. **列出 Tool contract**：记录每个 Tool 暴露的动作、参数和认证方式，区分 read、write、delete、export 与 admin action。
2. **确定 principal**：用户数据选择 OBO；后台任务选择 Agent Identity；只有 Tool connection 明确要求时才考虑 Project Managed Identity、key 或其他 OAuth 方式。
3. **先给最小 scope**：从资源、容器或业务 API 边界开始，不因为测试方便就直接给订阅级角色。
4. **发布后重新绑定**：取得生产 Agent 的新 `agentIdentityId`，再按权限账本逐项迁移，而不是复制开发环境的全部角色。
5. **验证成功和失败路径**：不仅测试正确用户能否访问，还要确认错误用户、越界资源、未审批写入和撤权后的调用会被拒绝。

这套顺序把“能不能跑”放在权限设计之后。它会让第一次配置多花一点时间，但可以避免用一个不断扩权的身份掩盖模型、connection 和 RBAC 之间的真实问题。

## 为什么开发正常，发布后却 403？

同一个 Foundry Project 中，未发布、开发中的 Prompt Agents 默认共享 Project 的 Agent Identity。这样可以减少原型阶段的重复配置，但也扩大了共享权限边界。

当 Agent 发布为 Agent Application 时，Foundry 会创建与应用绑定的独立 Blueprint 和 Agent Identity。新 `agentIdentityId` 不会自动继承开发阶段共享身份的 RBAC，所以发布后必须重新检查并分配必要角色。

对于开发阶段的共享身份，在 Azure portal 打开 Project，进入 **Overview → JSON View**，选择最新 API version，可以核对 `agentIdentityId` 和 Blueprint ID。

发布后应改为打开对应的 **Agent Application resource → Overview → JSON View**，再次复制新的 `agentIdentityId`。如果这里看到的 ID 仍与开发阶段相同，应先确认自己打开的是 Agent Application，而不是 Project；不要继续把生产角色授给旧身份。

Hosted Agent 的生命周期又不同：每个部署版本的 Hosted Agent 会自动获得专属 Agent Identity 和 endpoint；Project Managed Identity 用于项目级基础设施操作，例如让平台从 Azure Container Registry 拉取镜像，而不是 Agent 的运行身份。外部业务资源仍要单独授权给 Hosted Agent 的身份。可参考 [Hosted Agent 身份说明](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agents)。

## 一张 403 排查顺序表

当 Tool 调用失败时，我会按下面顺序排查，而不是直接增加角色：

1. **确认实际 principal**：共享 Project Agent Identity、发布后的独立 Agent Identity、Hosted Agent Identity、OBO user，还是 connection 指定的 Project Managed Identity？
2. **确认授权对象**：RBAC 是否真的授给当前 principal？发布后是否还在查看旧的 `agentIdentityId`？
3. **确认权限平面**：Azure `Owner` 或 `Contributor` 主要提供 ARM control plane 权限，并不自动包含 Foundry 或目标服务所需的 data plane 权限。可参考 [Hosted Agent permissions reference](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions)。
4. **确认 scope**：角色是在订阅、资源组、资源、容器还是数据对象范围生效？是否过宽或过窄？
5. **确认 audience**：Storage 应使用 `https://storage.azure.com` 这类目标 resource identifier，而不是 MCP Server URL。
6. **确认 Tool 支持**：Foundry 官方文档说明目前只有部分 Tool 支持 Agent Identity，并要求逐项检查 Tool 文档；当前明确给出 `AgenticIdentityToken` connection 配置的是 MCP 与 A2A。其他 Tool 可能使用 key、OAuth passthrough 或不同 connection 类型。

这套顺序的重点是先找到“谁在调用”，再讨论“缺什么权限”。否则，加再多角色也可能只是加给了错误身份。

## 路径三：有权限不等于应该自动执行

外发邮件和更新 CRM 都会产生业务影响。RBAC 只能说明 Agent 有资格调用接口，不能判断当前参数是否被 prompt injection 污染，也不能判断这次批量更新是否符合用户真实意图。

因此，我会把控制拆成三层：

```text
Identity：谁在调用
Authorization：它最多能做什么
Approval：这一次是否执行
```

Foundry MCP Tool 可以设置 `require_approval: "always"`（缺省即为 `always`），要求调用方在执行前检查 server、tool 和 arguments。例如，Toolbox 定义可以显式写成：

```yaml
resources:
  - kind: toolbox
    name: meeting-prep-tools
    tools:
      - type: mcp
        server_label: crm
        server_url: https://<your-mcp-server>/mcp
        require_approval: always
        project_connection_id: crm-connection
```

这段配置只是声明审批策略。对于 Toolbox，`tools/list` 返回的 `_meta.tool_configuration` 会携带 `require_approval`，但 Toolbox MCP endpoint 本身不会阻断 `tools/call`；Hosted Agent 或自定义 runtime 必须读取策略，在调用前展示待执行动作并等待用户确认。如果只想对高风险 Tool 强制审批、只读 Tool 保持自动执行，MCP Tool 还支持 `{"always": ["send_mail", "update_crm"]}` 这类按工具名单的取值。可参考 [MCP Tool 审批流程](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/model-context-protocol)和 [Toolbox 审批说明](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/toolbox)。

我的实践原则是：查询、总结等低风险只读动作可以在窄权限和参数校验下自动执行；外发、批量修改、删除、导出、权限变更等动作，应使用模型之外的 deterministic control。不要只在 system prompt 中写一句“请先询问用户”。

## 发布前检查清单

- [ ] 每个生产 Agent 有明确 owner、sponsor、用途和停用方式。
- [ ] 每条任务路径都明确选择 OBO、Agent Identity 或 connection 指定的其他身份。
- [ ] Tool 已区分 read、write、delete、export 和 admin action，并设置 allowlist。
- [ ] RBAC 授给实际 principal，scope 收窄到完成任务所需的边界。
- [ ] Prompt Agent 发布后，已用新的 `agentIdentityId` 复核全部角色。
- [ ] Hosted Agent 的项目基础设施权限与运行时业务权限已经分开。
- [ ] 高风险 Tool 使用可执行的 approval gate，而不只是 Prompt 约定。
- [ ] 日志能关联 caller、Agent、Tool、参数、资源、approval 和结果。
- [ ] 已测试撤销角色、禁用身份和阻断 Tool 后的实际行为。
- [ ] 对 key、connection secret 和 OAuth token，已定义存储、轮换与审计策略。

## 不要只验证 Happy Path

权限治理最有价值的测试往往是“它应该失败时是否真的失败”。至少可以加入四个负向用例：让没有文档权限的用户发起同一问题；把 Storage scope 改到未授权容器；拒绝一次外发邮件审批；撤销生产 Agent 的角色后再次调用。

预期结果不只是返回一个错误，还要确认日志能关联 caller、Agent Identity、Tool、resource、approval decision 和 correlation ID。否则系统虽然拒绝了动作，团队仍然无法快速解释为什么拒绝、谁受到了影响，以及是否需要停用 Agent。

## 小结

这个会议准备 Agent 最终不是一个“万能身份”，而是三条清晰的执行边界：用户数据用 OBO，后台任务用独立 Agent Identity，高影响动作再加真正可执行的 approval gate。

上线时最值得坚持的顺序是：**先确认谁在调用，再授予最小权限，最后为具体高风险动作设置审批和审计。**

**You can outsource your thinking, but you cannot outsource your understanding.**

## 相关阅读

- [Agent identity concepts in Microsoft Foundry](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity)
- [Hosted agents in Foundry Agent Service](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agents)
- [Hosted agent permissions reference](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-permissions)
- [Connect agents to MCP server endpoints](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/model-context-protocol)
- [Create and manage a toolbox in Microsoft Foundry](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/toolbox)
- [Azure built-in roles for Storage](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/storage)
- [Least privilege for AI agents](https://learn.microsoft.com/en-us/security/zero-trust/sfi/least-privilege-for-ai-agents)