---
layout:     post
title:      Microsoft Foundry可观测性-Red Team
subtitle:   Microsfot Foundry 系列(3)
date:       2026-05-04
author:     Bruce Wong
header-img: img/IMG_1513.WEBP
catalog: true
tags:
    - Microsoft Foundry
    - AI
---

## 一句话总结

**评估质量只是第一步。发现那些你根本没想到要评估的盲区，才是 AI Agent 安全的生死线。**

---

## 为什么需要红队测试？

你的 AI Agent 已经能：
- 理解用户意图 ✅
- 调用外部工具 ✅
- 执行多步骤任务 ✅

但它是否也能：
- 被一句精心构造的提示词"越狱"？❓
- 在诱导下执行被禁止的操作？❓
- 泄露敏感数据给不该看到的人？❓

**传统 QA 测的是"功能对不对"。红队测试测的是"坏人能不能让它做错"。**

---

## Microsoft Foundry 的红队测试：三步走

### 1. 选风险分类

Foundry 覆盖 **9 大类风险**：

| 风险类型 | 适用对象 | 测什么 |
|---------|---------|--------|
| **Hateful and Unfair Content** | Model + Agent | 仇恨、偏见内容 |
| **Sexual Content** | Model + Agent | 色情、性暴力内容 |
| **Violent Content** | Model + Agent | 暴力、武器相关内容 |
| **Self-Harm-Related Content** | Model + Agent | 自残、自杀内容 |
| **Code Vulnerability** | Model + Agent | 代码安全漏洞（注入、SQLi 等） |
| **Ungrounded Attributes** | Model + Agent | 无根据的推断（人口统计、情绪状态） |
| **Prohibited Actions** ⚠️ | **Agent only** | 执行被禁止的操作 |
| **Sensitive Data Leakage** ⚠️ | **Agent only** | 泄露财务、PII、健康数据 |
| **Task Adherence** ⚠️ | **Agent only** | 未按规则完成任务 |

> ⚠️ **注意**：后 3 项（Prohibited Actions / Sensitive Data Leakage / Task Adherence）**仅支持 Agent 场景**，且必须在 **Cloud 环境**运行（需要沙箱隔离）。
>
> 实测建议：**不要全选**。根据你的 Agent 能力边界，精准选择 3-5 个最相关的分类，节省时间和成本。

---

### 2. 选攻击策略

Foundry 内置多种攻击策略，按**复杂度**分为三档：

| 复杂度 | 策略类型 | 说明 |
|--------|---------|------|
| **Easy** | 直接请求 | Base64、编码、ROT13、Flip |
| **Moderate** | 变换技巧 | Tense |
| **Difficult** | 高级绕过 | 多轮对话诱导、Crescendo等 |

> 关键洞察：**直接攻击被拒绝 ≠ 安全**。PyRIT 的对抗策略会添加"转换层"绕过防御，比如把恶意请求编码后注入。Moderate 档的变换技巧（如 Base64）就是典型的绕过手段。

---

## PyRIT：当 Foundry 的 24 种策略不够用

视频里我们用的是 **Foundry 内置的攻击策略**，开箱即用。但如果你有自己的安全团队，想走得更远，需要了解底层武器库 **PyRIT**。

### PyRIT 是什么

**PyRIT**（Python Risk Identification Tool）是微软 2022 年内部孵化、2024 年开源的 AI 红队测试框架：

| 指标 | 数据 |
|------|------|
| 开源协议 | MIT |
| GitHub Stars | 3,800+ |
| 贡献者 | 129 人 |
| 对抗数据集 | 53+ |
| 提示词转换器 | 70+ |
| 攻击策略 | 6+（可组合扩展） |

> "我们能在几小时内生成数千条恶意提示词并评估 Copilot 系统的输出——**而不是几周。**" — 微软安全博客

### Foundry 和 PyRIT 的关系

**简单理解**：
- **PyRIT** = 开源武器库（研究者、安全专家直接用）
- **Foundry Red Teaming** = 企业级封装（把 PyRIT 的策略打包成易用的界面，一键运行）

**Foundry 的攻击策略**，底层就是 PyRIT 的积累。微软把复杂的对抗工程封装成了企业友好的界面。

### 什么时候该用 PyRIT 直接开发

| 场景 | 用 Foundry 就够了 | 需要 PyRIT 自建 |
|------|------------------|----------------|
| 快速验证 Agent 安全 | ✅ Portal 点几下 | |
| 企业合规、定期巡检 | ✅ SDK + CI/CD | |
| **自定义攻击策略** | | ✅ PyRIT 可扩展 |
| **深度研究新型攻击** | | ✅ 组合策略、自定义转换器 |
| **构建内部红队平台** | | ✅ 基于 PyRIT 开发 |


**关键优势**：PyRIT 支持**插件式扩展**——企业可以基于内置的 53 个数据集和 70 个转换器，开发针对自身业务场景的**专属攻击策略**。

---

### 3. 看结果，修漏洞

运行完成后，Foundry 给出：

- **Attack Success Rate (ASR)**：攻击成功率
- **按风险分类的详细报告**：哪些攻击成功了，为什么
- **攻击-响应对**：逐条查看 Agent 是如何被"骗"的

**核心动作**：
1. 查看成功的红队攻击 → 理解攻击路径
2. 针对性修改 Agent 的防护策略（system prompt、工具权限、内容过滤器）
3. 重新运行，验证修复效果

---

## 不只是 Portal：SDK 让红队测试成为流水线的一环

Foundry Portal 适合快速验证，但企业级安全需要**自动化**：

**三种集成场景**：
1. **预发布门禁**：每次发版前自动跑红队测试，ASR 超标阻断
2. **模型升级验证**：换模型后自动对比安全水位
3. **定期巡检**：每周/每月自动扫描，持续监控

---

## 写在最后

AI Agent 的安全不是"测一次就过关"的考试，而是**持续对抗的过程**。

攻击者每天都在发明新的提示词注入、新的越狱技巧、新的社会工程套路。你的防御策略如果停留在几个月前的认知，就已经落后了。

**红队测试的价值不是"证明安全"，而是"持续发现不安全"。**

在攻击者找到漏洞之前，先对自己开一枪。

---
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1axRvB9Eeb&cid=38081265855&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen>
</iframe>

## 参考资源

- [AI Red Teaming Agent 官方文档](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/ai-red-teaming-agent)
- [PyRIT 开源工具包](https://github.com/Azure/PyRIT)
- [OWASP Top 10 for LLM Applications 2025](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [MITRE ATLAS Matrix](https://atlas.mitre.org/matrices/ATLAS)

