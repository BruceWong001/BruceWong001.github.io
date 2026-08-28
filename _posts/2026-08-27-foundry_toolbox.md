---
layout:     post
title:      Foundry Toolbox + Tool Search：一个 Endpoint 管好 Agent 的工具
subtitle:   Microsoft Foundry 系列（11）
date:       2026-08-27
author:     Bruce Wong
header-img: img/IMG_1513.WEBP
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
---

大家好，今天我会用一个客户会议准备 Agent 来演示一下 Microsoft Foundry的一个新功能—— Toolbox：原来的 Agent 直接连接 CRM、Work IQ、Web Search 等多个工具；现在，代码只连接一个 Toolbox 的 MCP endpoint（MCP，Model Context Protocol，是一个定义 Agent 如何发现和调用外部工具的开放协议），Toolbox 再帮助 Agent 发现并调用完成任务所需的工具。

先说结论：Toolbox 的价值不只是把多个 Tool “装进一个盒子”。它把原本散落在每个 Agent 里的工具接入，变成一个可以复用、版本化和集中治理的服务层；配合 Tool Search，工具选择还会从“把完整清单都交给模型”变成“先按意图检索，再暴露少量候选工具”。

---
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1zhhV67Esq&cid=41261207200&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen
  title="用 Foundry Toolbox 演示 Agent 工具管理">
</iframe>

---

## 视频里真正改变了哪三件事？

在视频的直接连接版本中，Agent 自己持有多个 Tool 定义和连接。要再创建一个相似 Agent，就需要重复配置工具、凭据、审批规则和错误处理。

改成 Toolbox 后，边界发生了三层变化：

| 层次 | 直接给 Agent 加 Tool | 使用 Toolbox |
|---|---|---|
| 接入层 | 每个 Agent 分别配置多个 Tool | Agent 连接一个 MCP-compatible consumer endpoint |
| 上下文层 | 模型通常看到直接挂载的 Tool 定义 | 开启 Tool Search 后，初始只暴露元工具（即下文的 `tool_search` 和 `call_tool`）和被 pin 的工具 |
| 运维层 | 工具变化可能要求逐个修改 Agent | 在 Toolbox 中集中管理连接、策略和版本 |

Microsoft 将这套生命周期概括为 Build、Discover、Consume、Govern：构建可复用工具集合，按需发现，通过统一 endpoint 消费，再集中应用认证、授权、Guardrail、可观测性和版本管理。Toolbox 创建在 Foundry 中，但消费方不局限于 Foundry Prompt Agent；任何兼容 MCP 的运行时都可以接入，包括自定义 Agent、Microsoft Agent Framework 和 LangGraph。参见官方的 [Toolbox overview](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/toolbox-overview)。

可以把它理解成下面这条链路：

```text
Agent runtime
    -> Toolbox MCP consumer endpoint
        -> tool_search：按任务意图检索候选 Tool
        -> call_tool：调用已发现的 Tool
            -> CRM / Work IQ / Web Search / Azure AI Search / MCP ...

Toolbox 管理面
    -> connections / identity / guardrail / versions / observability
```

这更像一个“受治理的 Tool 服务层”，而不是另一个替 Agent 做业务推理的子 Agent。

## 统一 endpoint 不代表支持所有 Direct Tool

Toolbox 统一的是消费接口和管理方式，不是把任何能力都自动转换成可托管的 Tool。当前支持放入 Toolbox 的类型包括 MCP、Web Search、Azure AI Search、Code Interpreter、File Search、OpenAPI、A2A、Browser Automation、Fabric IQ 和 Work IQ 等；Function Calling 因为需要客户端执行，目前不能放入 Toolbox，Grounding with Bing、Computer Use、Image Generation、直接的 SharePoint Tool 和 Azure Functions 等也仍只支持直接集成。具体清单见官方 overview 的 [Supported tools](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/toolbox-overview#supported-tools)。

这也解释了一个容易从视频画面里看错的细节：我加入 Toolbox 的是 Catalog 中的 `Work IQ SharePoint` MCP entry，不是 Foundry 的直接 SharePoint Tool。两者都带有 SharePoint 名称，但所处的接口层不同。做迁移前应先核对支持矩阵、目标区域、模型兼容性和原工具的认证方式，不能假设 Direct Tool 可以原样搬入 Toolbox。

## 一个 endpoint 与减少 input token，不是同一件事

这是视频中最值得继续展开的地方。

把多个工具放进 Toolbox，首先解决的是 Agent 侧的接入复杂度：消费方只需要知道一个 endpoint。但如果没有开启 Tool Search，MCP 的 `tools/list`（客户端用来获取当前可用工具清单的方法）仍需要把 Toolbox 中的工具提供给客户端。换句话说，“单一 endpoint”本身不等于模型上下文里永远只有一个 Tool schema。

真正改变上下文规模的是 Tool Search。启用 `{"type": "toolbox_search"}` 后，Toolbox 会在初始 `tools/list` 中隐藏普通工具，只暴露两个元工具：

- `tool_search`：模型用自然语言描述需要什么能力；
- `call_tool`：模型按名称调用已经发现的工具。

Tool Search 使用 BM25，根据 Tool 的名称、描述和参数信息做匹配，默认返回最多 5 个候选，最大可设置为 10 个。同一轮里可以多次搜索；已经返回的工具在该轮剩余时间内保持可调用。完整机制见 [Enable tool search in a toolbox](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/tool-search)。

这套隐藏规则只作用于 Toolbox 内的工具。仍然直接挂载在 Agent 上的 Tool 会继续出现在初始列表中，不受 Tool Search 影响。如果迁移后保留了重复的 Direct Tool，接入虽然能运行，预期的上下文缩减却可能没有完全发生。

视频末尾展示的官方页面给出了一个很醒目的数字：在平均 600+ tools、启用 Tool Search 的开源数据集评估中，每次调用的 input tokens 从 313k+ 降到 18k，页面标注降幅 94%；Tool selection accuracy 从 48.2% 变为 52.4%。这里有三个限定词不能省略：600+ tools、Tool Search、特定评估数据集。它不是本次几个工具的会议 Agent 的实测结果，也不保证每个业务都节省 94%。

对只有五到十个稳定工具的 Agent，额外搜索一次可能未必值得。官方建议主要在工具数量超过约 10–15 个，或者不同任务只需要工具集合中的不同子集时使用 Tool Search。小型 Agent 即使不启用它，也仍可因为集中连接、复用和版本治理而使用 Toolbox。

## Tool Search 把工具设计变成了检索设计

Tool Search 不是魔法路由器。既然底层匹配依赖名称、描述和参数，Tool metadata 就从“给开发者看的注释”变成了检索索引的一部分。

以视频里的客户会议 Agent 为例，`get_customer` 如果只写成“Get data”，很难与“查找客户机会、联系人和续约风险”这样的意图稳定匹配。描述至少应该回答：它读取什么业务对象、适合什么任务、关键输入是什么、是否产生副作用。团队还有自己的叫法时，可以用 `additional_search_text` 补充同义词，而不修改 MCP server 原始 schema。

对于几乎每轮都要用、又不希望多一次搜索往返的工具，可以配置 `pin: true`。被 pin 的工具会与两个元工具一起直接出现在 `tools/list` 中。代价也很直接：pin 得越多，初始上下文越大，Tool Search 的收益越小。

下面是官方配置形态的简化示例：

```json
{
  "type": "mcp",
  "server_label": "crm",
  "server_url": "https://crm.example.com/mcp",
  "tool_configs": {
    "get_customer": {
      "pin": true,
      "additional_search_text": "客户 账户 联系人 商机 续约风险 sales opportunity"
    }
  }
}
```

这段配置只控制“如何发现工具”，不授予额外权限，也不会把高风险写操作自动变安全。

## 集中治理不等于自动安全

Toolbox 可以集中凭据注入、OAuth identity passthrough、Guardrail 和版本，但生产设计仍要分清三个不同控制面。

第一，Guardrail 负责内容防护。官方文档说明，绑定到 Toolbox version 的 RAI policy 会在 Toolbox 层检查 Tool input 和 output，并独立于模型本身的内容过滤器运行。它不能代替下游系统的 RBAC、OAuth scope 或业务授权。

第二，Approval 需要运行时执行。当工具配置为 `require_approval: "always"` 时，Toolbox 会在 `tools/list` 返回的 metadata 中标记审批要求；但 MCP endpoint 本身不会阻止客户端直接发起 `tools/call`。真正弹出确认并等待用户批准的是 Agent runtime。自定义运行时如果忽略这段 metadata，审批策略就没有被执行。详见 [Create and manage a toolbox](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/toolbox)。

第三，OAuth 工具可能在首次调用时返回 `CONSENT_REQUIRED`。这不是“Tool Search 没找到工具”，而是当前用户还没有完成连接所需的授权。生产 Agent 要把发现失败、授权失败和业务调用失败分开处理。

## 版本化解决升级问题，也放大升级影响

Toolbox version 是不可变快照。创建第一版时它会成为 `default_version`；之后创建新版本不会自动切换默认版本。官方提供两类 endpoint：

- 带 `/versions/{version}/mcp` 的 developer endpoint，用于测试指定版本；
- 不带版本号的 consumer endpoint，始终跟随 `default_version`。

这让团队可以先在 v2 验证工具清单、Tool Search 命中、OAuth、Approval 和负向用例，再把 v2 promote 为默认版本。连接 consumer endpoint 的 Agent 不需要改代码或重新部署。

但集中化也带来新的 blast radius：如果十个 Agent 共用同一个 consumer endpoint，一次错误的默认版本升级可能同时影响十个 Agent。因此，Toolbox 不应按“全公司所有工具”无限扩张，更适合按业务能力和权限边界拆分，例如 `sales-read-toolbox`、`customer-briefing-toolbox` 和 `crm-write-toolbox`。

## 我会怎样把视频里的 Demo 推向生产？

我会按下面的顺序落地：

1. 先按业务职责和读写风险划分 Toolbox，而不是先把所有工具放进去。
2. 为每个 Tool 写可检索的名称、描述和参数说明，并准备一组真实用户意图做命中测试。
3. 默认隐藏普通工具，只 pin 几乎每轮必用的少数工具；为内部术语补充 `additional_search_text`。
4. 把读操作和写操作拆开，对写、删、发送、外部发布等动作配置 Approval，并验证运行时真的会阻断未批准调用。
5. 用 version-specific endpoint 测试 `tools/list`、正确命中、相似工具误选、OAuth consent、Guardrail 和失败重试，再 promote 默认版本。
6. 监控的不只是最终答案，还包括搜索 query、候选 Tool、实际调用、失败原因、token 使用和版本号。

不要只用一次成功对话判断 Tool Search 是否有效。我会准备一组覆盖常见任务、同义表达、相似 Tool 和无可用 Tool 的测试问题，为每题标记期望工具，然后比较 direct tools 与 Toolbox 两种配置的候选命中率、误选率、input tokens、额外搜索次数和端到端延迟。只有自己的任务分布才能回答“是否值得开启”，官方 94% 只能作为规模化工具集合的参考基线。

视频展示的是“多个工具如何收拢为一个 Toolbox endpoint”；真正决定它能否进入生产的，是 endpoint 背后的发现质量、权限边界、运行时审批和版本发布纪律。

如果只有一个 Agent、少量稳定工具，也没有跨团队复用和集中治理需求，直接挂载 Tool 可能更简单。Toolbox 的收益通常在第二个 Agent、第二个团队和第一次工具升级出现时才真正放大。

**You can outsource your thinking, but you cannot outsource your understanding.**

## 官方资料

- [What is Toolbox in Microsoft Foundry?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/toolbox-overview)
- [Create and manage a toolbox in Foundry](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/toolbox)
- [Enable tool search in a toolbox](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/tool-search)