---
layout:     post
title:      Tool 已经复用了，流程怎么复用？用 Microsoft Foundry Toolbox 管理 Skills
subtitle:   Microsoft Foundry 系列（12）
date:       2026-09-04
author:     Bruce Wong
header-img: img/IMG_1513.WEBP
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
---

Toolbox 解决“Agent 可以使用哪些能力”，Skill 解决“Agent 应该按什么流程完成任务”。把两者放在一起，团队才能同时复用工具接入和工作方法。

## 视频：Foundry Toolbox Skill 演示

建议先看一遍完整演示，再结合后面的步骤和机制说明理解画面中的 Skill、Toolbox 与工具调用关系。

---
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1t3tB64E92&cid=41583447937&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen
  title="用 Foundry Toolbox 演示 Agent 工具管理">
</iframe>

---

## Tool、Skill、Toolbox 和 Agent 分别负责什么

| 组件 | 负责的问题 | 在演示中的例子 |
|---|---|---|
| Tool | 可以读取什么、执行什么动作 | CRM `get_customer`、Web Search、Work IQ |
| Skill | 完成一类任务时应遵守的步骤、边界和交付格式 | 先查客户与商机，再找产品更新，最后生成会议 briefing |
| Toolbox | 将工具和 Skills 组合、版本化，并通过统一 endpoint 提供给运行时 | `CRM-Toolbox` v5 |
| Agent / runtime | 理解用户意图，选择 Skill，调用工具并组织最终答案 | Foundry Client Meeting Assistant |

Microsoft Learn 对二者的区分很直接：Tools 定义 Agent **能做什么**，Skills 定义 Agent **如何完成任务**。Toolbox 则把经过筛选的工具与 Skills 集中交付给 Agent，并提供认证、版本和治理能力。[What is Toolbox in Microsoft Foundry?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/toolbox-overview)

Skill 也不会替代 Tool。会议流程可以要求“读取最新 CRM 商机”，但真正访问 CRM 的仍然是 MCP Tool；Skill 本身不会凭空获得 CRM 权限，也不能修复一个不可用的 endpoint。

## SKILL.md 里到底放什么

Foundry Skill 遵循 Agent Skills 格式。最小结构只有 YAML front matter 和 Markdown 指令：

```markdown
---
name: prepare-customer-meeting
description: Prepare a sourced, read-only customer meeting briefing and agenda.
---

# Prepare Customer Meeting

1. Gather the latest customer and opportunity data from CRM.
2. Find relevant product releases from authoritative sources.
3. Reconcile both evidence sets and generate the meeting agenda.
4. Cite important claims and report evidence gaps.
5. Do not send messages, create meetings, or modify CRM records.
```

`name` 是稳定标识，只能使用小写字母、数字和连字符，最长 64 个字符。`description` 不是宣传文案；运行时会用它判断这个 Skill 是否与当前任务相关。正文才承载完整 SOP，包括输入要求、来源优先级、工具使用规则、异常处理、输出格式和完成条件。官方当前允许通过 inline content、`SKILL.md` 或包含该文件的 ZIP 创建 Skill 版本。[Use skills in Foundry (preview)](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/skills)

视频中的会议 Skill 比上面的最小示例更完整。它要求：

- 校验 CRM 数据确实属于目标客户，并优先使用最新的有效商机；
- 先查内部产品资料，再查公开产品页面，并区分 Preview、计划项与已发布能力；
- 把最近沟通和客户新闻作为补充证据，而不是替代 CRM 与产品资料；
- 将内部信息和公开信息分开列出；
- 找不到来源时暴露缺口，不用推测填空；
- 只生成 briefing，不发送邮件、不建会议、不修改 CRM。

这也是“流程复用”和“Prompt 模板复用”的差别。Prompt 模板通常只是把一段长文本搬到别处；合格的 Skill 还要定义失败时怎么办、哪些动作禁止执行、怎样判断结果完成。

## 为什么一句短 Prompt 也能触发完整流程

从协议视角看，Tool 和 Skill 也不是同一条发现路径。

Toolbox 中的工具通过 MCP Tools 暴露。启用 Tool Search 后，初始 `tools/list` 不再把所有工具定义都塞给模型，而是先提供 `tool_search` 和 `call_tool` 两个 meta-tools；模型按意图搜索需要的工具，再发起调用。官方建议在 Toolbox 超过约 10–15 个工具，或者不同任务需要不同工具子集时使用这个能力。[Enable tool search in a toolbox](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/tool-search)

Skills 则作为 MCP Resources 暴露。支持该能力的客户端先通过 `resources/list` 发现 Skill，再用 `resources/read` 读取需要的内容。Microsoft Agent Framework 的参考模式会先向模型展示 Skill 的名称和描述，只有判断相关时才加载完整正文和补充资源。这种 progressive disclosure 避免每个会话都预先注入全部 SOP。[Use skills in Foundry (preview)](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/skills)

可以把视频中的运行过程理解为下面这条链路。它是基于官方协议行为和画面结果整理的简化模型，不代表视频展示了每一次底层 MCP 请求：

```text
用户短 Prompt
    ↓
Agent 根据 name / description 选择会议 Skill
    ↓
读取 Skill 正文，获得来源顺序、只读边界和输出格式
    ↓
按任务意图发现并调用 CRM、Web Search 等 Tool
    ↓
校验证据，生成 meeting briefing
```

所以，短 Prompt 并没有让需求变少。它只是把稳定的流程从每次对话中移到了一个可复用、可管理的 Skill 中。

## 版本管理比“上传成功”更重要

Foundry 中的 Skill 版本是不可变快照。修改 Skill 会创建新的 `SkillVersion`，父级 Skill 记录 `latest_version` 和当前生效的 `default_version`。Toolbox 版本同样是不可变快照。

把 Skill 引用到 Toolbox 时有两种选择：

- 不指定版本：跟随 Skill 的 `default_version`，适合快速迭代；
- 固定版本：Toolbox 始终使用某个已验证快照，更适合测试、审计和生产发布。

## 不要把所有 system prompt 都搬进 Skill

Skill 适合承载“只在某类任务中才需要”的流程，例如客户会议准备、工单分诊或发布检查。Agent 身份、所有会话都必须遵守的安全规则、不可覆盖的产品边界，仍应留在系统级配置和运行时策略中。

这一区分也影响冲突处理。如果 Agent 的系统规则要求所有写操作必须审批，而某个 Skill 要求直接发送邮件，不能期待模型仅靠文字自行裁决。生产设计应让系统策略和授权机制成为硬边界，再让 Skill 在边界内描述任务步骤。Skill 是可加载的任务说明，不应成为唯一的安全控制。

## 设计 Skill 时，最容易忽略的四个边界

### Skill 不能授予权限

“只读”写进 Skill 是行为约束，不是授权系统。真正的访问范围仍由 Toolbox connection、Entra ID、OAuth scope、外部系统权限和运行时审批共同决定。对发送邮件、删除记录等有副作用的 Tool，不能只依赖一句自然语言禁令。

### Tool Search 不负责寻找 Skill

Tool Search 搜索的是工具定义；Skill 通过 MCP Resources 被发现和加载。两者都能减少初始上下文负担，但处理的是不同对象。把 Skill 写得再好，也不能弥补 Tool 的 `name`、`description` 或参数说明含糊；反过来，工具搜索准确也不代表 Agent 会自动遵守完整业务 SOP。

### 客户端必须支持 MCP Resources

Toolbox endpoint 中已经附加 Skill，不等于所有 MCP client 都会使用它。客户端需要实现 `resources/list` 与 `resources/read`，或者使用能够加载 Toolbox Skills 的 provider。官方故障排查也把“client 是否支持 MCP Resources”列为 Skill 无法发现时的检查项。

## 如何验证一个会议 Skill 真的可用

不要只看最终回答像不像一份会议材料。至少做五类测试：

1. **正常路径**：CRM、产品资料和公开信息都有结果，检查议程能否把客户需求与产品变化对应起来；
2. **来源缺失**：像视频中的 Contoso 示例一样，公开搜索没有可靠结果时，确认 Agent 会暴露缺口而不是编造；
3. **数据冲突**：CRM next step 与最近邮件不一致时，结果应标记冲突并说明日期；
4. **副作用测试**：请求中加入“顺便发送邀请”，确认只读 Skill 和运行时审批都能阻止未授权动作；
5. **版本回归**：固定 Skill 与 Toolbox 版本，记录工具调用轨迹和输出结构，再比较新版本是否改变行为。

还要检查 Skill 是否选得对。如果用户只是问一个 CRM 字段，直接调用 Tool 往往更简单；一次性的开放式研究也未必值得维护专门 Skill。Skill 更适合重复发生、步骤相对稳定、输出有验收标准、错误成本又不低的任务。

## 最后的判断

这段视频真正完成的，不是把一份长 system prompt 换成一个 Markdown 文件。它把“准备客户会议”从某个 Agent 的临时提示词，变成了一个可以和 CRM、Web Search 等能力一起版本化交付的工作流。

如果团队已经在 Toolbox 中复用了工具，下一步可以检查这些工具背后的重复流程：会议准备、工单分诊、发布检查、报告生成、合规复核。先选一个边界清楚的任务，把来源、步骤、失败分支和验收条件写进 Skill，再用缺失数据和越权请求测试它。通过这些测试之后，Skill 才算真正可复用。

**You can outsource your thinking, but you cannot outsource your understanding.**

## 官方参考

- [What is Toolbox in Microsoft Foundry?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/toolbox-overview)
- [Use skills in Foundry (preview)](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/skills)
- [Create and manage a toolbox in Microsoft Foundry](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/toolbox)
- [Enable tool search in a toolbox](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/tool-search)
