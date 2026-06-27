---
layout:     post
title:      Foundry IQ 易混淆功能解读
subtitle:   Microsoft Foundry 系列(8)
date:       2026-07-04
author:     Bruce Wong
header-img: img/IMG_1189.webp
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
---

最近在使用Microsoft Foundry IQ构建企业级多源 RAG 知识库时，发现有一些容易混淆的功能和概念：
+ Foundry IQ 与 Azure AI Search 是什么关系？
+ 检索为什么需要 LLM？
+ 它与传统 RAG 有什么不同？

今天就来一一解读这些问题，帮助大家更好地理解和使用 Foundry IQ。

## 先澄清命名：两个入口是不是同一个 Foundry IQ？

**它们指向同一套 Foundry IQ 知识库能力，但属于不同的管理界面。**

- Azure 门户中的 **Foundry IQ（Azure AI Search）** 偏底层资源管理，包括 Search Service、索引、索引器、Knowledge Source、Knowledge Base 和 agentic retrieval。
- Microsoft Foundry 项目中的 **Knowledge（Foundry IQ）** 偏 Agent 开发体验，用于创建、配置和连接 Knowledge Base。

两边最终操作的 Knowledge Base 和 Knowledge Source 都托管在选定的 Azure AI Search Resource 中。如果 Search Endpoint（例如 `https://my-search-service.search.windows.net`）和 Knowledge Base 名称相同，看到的就是同一个底层知识库。[1][2]

部分产品与定价页面会显示 **Foundry IQ（Azure AI Search）**，但这不等于所有 Azure AI Search 技术名称都已被替换。官方技术文档仍将 Azure AI Search 定义为 Foundry IQ 的底层索引与检索基础设施；Index、Indexer、Skillset、Vector Search 和 Search API 也继续沿用 Azure AI Search 名称。[1][3]

```text
Microsoft Foundry
  └─ Knowledge（Foundry IQ）
       └─ Project Connection
            └─ Azure AI Search Resource
            ├─ Knowledge Base / Knowledge Sources
            ├─ Index / Indexer / Vector Search
            └─ Agentic Retrieval
```

一句话概括：**Azure AI Search 负责“搜”，Foundry IQ 负责把搜索组织成可供 Agent 复用的知识服务。**

## Knowledge Base 为什么需要一个 LLM？

![在 Microsoft Foundry 中配置 Knowledge Base](/img/foundry/foundry-knowledge.png)

截图中的 Chat Completions Model 属于 **Knowledge Base 的检索管线**。它不是生成向量的 Embedding Model，也不一定是 Agent 的主模型。目前用于 query planning 的 LLM 仅限 **gpt-4o、gpt-4.1 和 gpt-5 系列**的 Azure OpenAI 部署，具体列表见官方文档。[2][3] 它主要承担三项工作：[2][3]

1. **查询规划**：结合用户问题和对话历史，把复合问题拆成更聚焦的子查询。
2. **查询与知识源规划**：判断应该查询哪些来源并生成子查询；随后由 Azure AI Search 执行关键词、向量或混合检索与语义重排。
3. **可选的答案合成**：当 Output Mode 为 `Answer synthesis` 时，将证据合成为带引用的答案；使用 `Extracted data` 时，则把检索内容和来源交给 Agent 生成最终回答。[2][4]

`Retrieval reasoning effort` 控制 LLM 介入检索的程度，并直接决定可用资源上限：[5]

| 模式 | 核心行为 | 限额（来自官方 FAQ） | 建议 |
|---|---|---|---|
| `Minimal` | 不做 LLM 查询规划，直接搜索 | 最多 10 个 knowledge sources；无 LLM、无答案合成 | 最低成本和延迟；仅支持提取式结果，不支持 Web 知识源 |
| `Low` | 一轮查询规划和知识源选择，默认模式 | 最多 3 个 sources / 3 个子查询；答案合成预算 5,000 tokens | 大多数场景的起点 |
| `Medium` | 首轮结果不足时可修订查询并再检索一次 | 最多 5 个 sources / 5 个子查询；答案合成预算 10,000 tokens | 复杂、跨来源、重视完整性的任务 |

推理强度越高，通常检索覆盖更好，但延迟与 token 成本也会上升。具体限额可参考 [Agentic retrieval limits][8]。

另外，Web 等远程 knowledge source 必须启用 `Answer synthesis` 才能返回结果，因此 `Minimal` 模式（无答案合成）天然不支持 Web 源。[10]

这里还要区分两个模型角色：**Knowledge Base 的 LLM 优化“如何找到证据”，Agent 的 LLM 负责“如何完成任务”。** 两者可以使用相同部署，也可以不同。对于还要调用其他工具并统一组织答案的 Foundry Agent，官方建议优先使用 `Extracted data`，把原始证据和引用交给 Agent 推理，避免知识库和 Agent 重复生成答案；只有在检索结果直接面向终端用户、不需要 Agent 二次加工的场景，才更适合使用 `Answer synthesis`。[3][4]

## 为什么 Blob Knowledge Source 也要配置模型？

![配置 Azure Blob Knowledge Source](/img/foundry/azure%20blob.png)

Blob 页面与 Knowledge Base 页面虽然都有 Chat Completions Model，但它们工作在 RAG 的不同阶段：[7]

| 模型 | 运行阶段 | 作用 |
|---|---|---|
| Blob 的 Chat Model | 文档摄取、建索引或刷新时 | 生成式内容提取，尤其是把图片、图表等视觉内容描述成可检索文本 |
| Blob 的 Embedding Model | 建索引及向量查询时 | 将文档分块和用户查询转换成向量，用于语义相似度检索 |
| Knowledge Base 的 Chat Model | 每次用户检索时 | 理解问题、拆分子查询、选择知识源，以及可选的答案合成 |

创建 Blob Knowledge Source 时，系统会自动生成 Data Source、Indexer、Skillset 和 Search Index。截图中的 `Content extraction mode = Standard` 使用 Azure Content Understanding 做高级文档解析与分块；启用图片描述时，Blob Chat Model 会被生成的 GenAI Prompt Skill 用于把图片等视觉信息转成文本；Embedding Model 则通过 Embedding Skill 和 Vectorizer 分别服务于索引与查询。[7]

## 与 Agent 直接连接 Azure AI Search 有什么不同？

传统 RAG 通常让 Agent 直接连接一个 Search Index，配置查询类型、`top_k` 和过滤条件；Search 返回文档片段，再由 Agent 主模型作答。这种方式路径短、控制直接，适合单索引和稳定查询。[6]

需要说明的是，Knowledge Base 并非只能被 Foundry Agent Service 调用。任何支持 Azure AI Search knowledge base API 的应用、Microsoft Agent Framework 或自建服务都可以调用它；Foundry Agent Service 只是提供了原生集成和托管体验。[1]

Foundry IQ 并没有推翻 RAG，而是把越来越复杂的“检索编排”从每个 Agent 中抽离出来：

| 直接使用 Azure AI Search | 使用 Foundry IQ Knowledge Base |
|---|---|
| 通常面向一个索引 | 可统一组织多个索引型或远程知识源 |
| 通常执行一次配置好的搜索 | 可拆分问题、选择来源、并行查询和迭代检索 |
| Agent 自己处理文档并生成答案 | Knowledge Base 先重排、合并并保留引用 |
| 检索配置分散在各个 Agent 中 | 一个领域知识库可被多个 Agent 共享和独立更新 |
| 成本和延迟更容易预测 | 复杂问题效果可能更好，但会增加模型调用和延迟 |

其核心价值是**关注点分离**：Azure AI Search 负责检索执行，Knowledge Base 负责领域知识与检索策略，Agent 负责对话、工具调用和业务行动。官方基准显示，agentic retrieval 在响应质量上比传统单次 RAG 高约 36%。[9]

## 如何选择？

- 只有一个成熟索引、问题简单、强调精确控制与低延迟：继续直接使用 Azure AI Search Tool。
- 需要跨来源回答复合问题、理解对话上下文，或让多个 Agent 共享同一领域知识：优先考虑 Foundry IQ。
- 生产落地前，用真实问题同时评测答案正确率、引用质量、P95 延迟和单次成本，再决定采用 `Minimal`、`Low`、`Medium` 或直接 Search Tool。
- 演示可以使用 API Key；生产环境优先使用 Microsoft Entra ID、Managed Identity 和最小权限 RBAC。使用 Web 等远程知识源时，还要检查数据边界、许可与合规条款。若使用 Remote SharePoint knowledge source，终端用户需持有有效的 Microsoft 365 Copilot license。[7][10]

最终可以这样理解：

> **Foundry IQ 不是另一个搜索引擎，而是以 Azure AI Search 为基础、面向 Agent 的知识与上下文工程层。**

**You can outsource your thinking, but you cannot outsource your understanding.**

## 参考资料

1. Microsoft Learn：[What is Foundry IQ?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-foundry-iq)
2. Microsoft Learn：[Create a knowledge base in Azure AI Search](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-create-knowledge-base)
3. Microsoft Learn：[Agentic retrieval in Azure AI Search](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-overview)
4. Microsoft Learn：[Connect a Foundry IQ knowledge base to Foundry Agent Service](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/foundry-iq-connect)
5. Microsoft Learn：[Set the retrieval reasoning effort](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-set-retrieval-reasoning-effort)
6. Microsoft Learn：[Connect an Azure AI Search index to Foundry agents](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/ai-search)
8. Microsoft Learn：[Agentic retrieval limits](https://learn.microsoft.com/en-us/azure/search/search-limits-quotas-capacity#agentic-retrieval-limits)
9. Microsoft Tech Community：[Foundry IQ: Boost response relevance by 36% with agentic retrieval](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/foundry-iq-boost-response-relevance-by-36-with-agentic-retrieval/4470720)
10. Microsoft Learn：[Foundry IQ FAQ](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/foundry-iq-faq)

