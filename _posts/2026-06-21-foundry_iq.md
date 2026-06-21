---
layout:     post
title:      如何用Foundry IQ为 Foundry Agent 构建企业级多源 RAG 知识库
subtitle:   Microsoft Foundry 系列(6)
date:       2026-06-21
author:     Bruce Wong
header-img: img/IMG_1189.webp
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
---

企业知识往往分散在内部文档、业务系统和公开网页中的实时信息。下面的短视频演示了如何使用 **Foundry IQ**，把 Azure Blob Storage 中的法规文件与实时 Web 信息组合成一个 Knowledge Base，再将它连接到 Foundry Agent，为回答提供可追溯的依据。

## 视频中完成了什么？

整个过程可以概括为五步：

1. 在 Microsoft Foundry 的 **Knowledge（Foundry IQ）** 中连接一个已有的 Azure AI Search Resource。
2. 创建 Azure Blob Knowledge Source，导入 GDPR 和 EU AI Act 文档。
3. 添加 Web Knowledge Source，用于补充最新法规、新闻和执法案例。
4. 配置检索指令、回答格式与 Retrieval Reasoning Effort。
5. 将 Knowledge Base 添加为 Foundry Agent 的知识工具，并通过引用验证两个数据源都被正确使用。

这样得到的是一个可供多个 Agent 复用的领域知识库。在 `Low` 或 `Medium` 检索强度下，LLM 负责规划查询和选择来源；Azure AI Search 执行并行检索、语义重排与结果合并，并返回证据和引用。[1][2]

## Foundry IQ 与 Azure AI Search 是什么关系？

目前在不同门户中可能同时看到 **Foundry IQ（Azure AI Search）** 和 **Knowledge（Foundry IQ）**。它们不是两套独立的知识库，而是同一套能力的不同管理界面：

- Azure 门户侧重管理 Search Service、Index、Indexer、Knowledge Source 和 Knowledge Base。
- Microsoft Foundry 侧重为 Agent 创建、配置和连接 Knowledge Base。

如果两边连接的是同一个 Search Endpoint，并使用相同的 Knowledge Base 名称，操作的就是同一个底层知识库。Azure AI Search 仍提供索引和检索基础设施；Foundry IQ 则提供面向 Agent 的知识编排体验。[1][3]

## 为什么 Blob 和 Knowledge Base 都要配置 Chat Model？

它们使用同类模型，但工作阶段不同：

| 配置位置 | 主要职责 |
|---|---|
| Blob Chat Model | 在索引或刷新时进行图片描述等生成式内容处理 |
| Blob Embedding Model | 在索引和查询时把文档分块与查询转换为向量 |
| Knowledge Base Chat Model | 在检索时规划查询、选择来源，并可选地合成答案 |

一句话记忆：**Blob Chat Model 让文件更容易被搜索；Embedding Model 计算语义距离；Knowledge Base Chat Model 决定问题应该怎样搜索。**[2][4]

## 相比传统 RAG，变化在哪里？

传统方案通常让 Agent 直接查询一个 Azure AI Search Index，再整理返回的文档。Foundry IQ 则把多源路由、查询规划、结果合并和引用管理抽离为可独立维护、可被多个 Agent 共享的知识层。

对于单索引、简单查询和低延迟场景，直接使用 Azure AI Search Tool 仍然简单有效；对于复合问题、多数据源或多个 Agent 共享知识的场景，Foundry IQ 更有价值。

## 总结

Azure AI Search 让企业内容可搜索；Foundry IQ 把这些内容组织成 Agent 可规划、可组合、可共享的知识服务。这正是视频中法规顾问 Agent 能够同时引用内部 PDF 与外部最新案例的原因：答案不只来自模型记忆，而是建立在动态检索、来源引用和企业知识之上。

---
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1Q7jb63EP7&cid=39284639585&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen>
</iframe>

---

**理解AI，用好AI，让AI帮助自我进化，加油。**

## 参考资料

1. Microsoft Learn：[What is Foundry IQ?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-foundry-iq)
2. Microsoft Learn：[Agentic retrieval in Azure AI Search](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-overview)
3. Microsoft Learn：[Create a knowledge base in Azure AI Search](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-create-knowledge-base)
4. Microsoft Learn：[Create a Blob knowledge source for agentic retrieval](https://learn.microsoft.com/en-us/azure/search/agentic-knowledge-source-how-to-blob)
5. Microsoft Learn：[Connect a Foundry IQ knowledge base to Foundry Agent Service](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/foundry-iq-connect)
