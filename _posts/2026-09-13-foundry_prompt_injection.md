---
layout:     post
title:      加班规定为何泄露员工名单？RAG 间接提示词注入与 Microsoft Foundry Red Teaming
subtitle:   Microsoft Foundry 系列（13）
date:       2026-09-13
author:     Bruce Wong
header-img: img/IMG_1513.WEBP
catalog: true
tags:
    - 技术解析
    - Microsoft Foundry
    - AI
---

视频一开始，我只问了 HR Agent 一句：“公司的加班规定是什么？”它先给出正确政策，随后却列出了完整员工目录。

这不是模型随口编造了几个人名。Foundry Trace 显示，Agent 真的调用了员工查询工具。继续检查原始文档和知识库分片后，藏在 Word 文档里的指令浮了出来。

问题也由此变得清楚：RAG 取回的是外部内容，不是可信指令。Agent 一旦混淆两者，再碰上权限过宽的 Tool，一次普通问答就可能泄露数据。

---
<iframe
  src="//player.bilibili.com/player.html?bvid=BV16kYe67E9o&cid=41853585903&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen
  title="演示 RAG 间接提示词注入与 Foundry Red Teaming">
</iframe>

---

## 四个画面串起攻击链

如果你刚接触 RAG，可以先把它理解成“回答前，先去知识库查资料”。长文档通常会被切成较小的文本片段，也就是 chunk（分片），系统只把相关分片交给模型。

视频里的排查过程很直接：

1. **回答出现异常**：询问加班政策，结果中混入了员工名单。
2. **Trace 找到实际调用**：Agent 先检索知识库，随后调用 `query_users`。这里的 Tool，就是 Agent 可以执行的程序接口。
3. **回到原始 Word**：文档看上去只有正常的 HR 规定，但页面中藏着一段与背景同色的文字，要求 Agent 查询全部员工。
4. **检查转换与分片**：Word 转成 Markdown 后，隐藏文字被保留下来；它又和加班规定落入同一个分片，所以“加班”这个问题正好把恶意指令一起检索了出来。

整条路径可以缩成一行：

```text
隐藏指令 -> 文档解析 -> 知识库分片 -> Agent 上下文 -> query_users -> 员工目录泄露
```

人眼看不到的文字，解析器未必看不到。类似问题也可能来自网页、邮件或 OCR 结果，因此只检查 Word 的隐藏样式并不够。

## 为什么 Agent 会照着文档执行

用户直接输入“忽略之前的规则”，属于直接提示词注入。视频里的指令来自外部文档，用户本人甚至不知道它存在，这叫 **间接提示词注入**（Indirect Prompt Injection，也常写作 XPIA）。

这次演示同时暴露了三个缺口：文档内容未经安全处理便进入模型上下文；模型没有稳定地区分资料与命令；员工查询工具允许空条件返回全部记录。System Prompt 可以提醒模型“只把文档当资料”，却不能代替后端权限控制。

还有一个很容易误判的地方：`query_users` 虽然是只读 Tool，仍然可以泄露数据。“只读”只代表不修改数据，不代表低风险。批量查询、导出和读取敏感字段，都应该受到单独限制。

## Red Teaming 在这里解决什么问题

视频前半段是在定位一个已经发生的问题；后半段则使用 Microsoft Foundry Red Teaming 主动尝试不同攻击，看看 Agent 还会怎样越界。

演示中的独立测试选择了 **Prohibited actions** 风险类别，并使用 Base64、Flip 等提示词变换。结果页中的 ASR 是 Attack Success Rate，也就是“攻击成功率”：数字越高，风险越大，并不是 Agent 的考试分数。页面还会给出逐条判定理由，方便检查某个样本为什么被算作攻击成功。

需要分清一点：这次 Red Teaming run 展示的是自动化攻击能力，不等于它重新发现了前面的 Word 文档问题。Microsoft 官方另有说明，AI Red Teaming Agent 支持 [Indirect Jailbreak / XPIA](https://learn.microsoft.com/en-us/azure/foundry/concepts/ai-red-teaming-agent#indirect-prompt-injection-attacks-xpia)。官方也提醒，生成式攻击与判定可能出现波动或误报，所以高风险结果仍要人工复核。

日常 Evaluation 更像固定题目的回归测试；Red Teaming 更像主动寻找还没写进题库的攻击方式。确认有效的攻击样本，最后应加入 Evaluation，防止后续版本再次出现同类问题。

## 如果按这次问题来修

1. **检查知识入口**：记录文档来源和转换结果，对比原文与解析文本，识别隐藏文字、异常链接和可疑指令。Foundry Prompt Shields 可用于检测 User Prompt attack 和 Document attack，但仍要结合自己的文档处理流程。
2. **收紧 Tool 边界**：禁止空查询、通配符和无限量返回；按当前用户限制可见字段与记录；员工目录、薪酬等敏感查询应增加审批或确认。
3. **把攻击变成回归用例**：先在非生产环境运行 Red Teaming，人工确认结果，再用固定 Evaluation 持续验证。正常业务问题也要保留，避免安全规则把功能一并挡掉。

比“隐藏白色文字”这个技巧更值得关注的是，攻击连续跨过了文档解析、RAG 检索和 Tool 授权三层。上线前不妨做一次反向检查：如果知识库中的文字要求 Agent 调用最敏感的 Tool，究竟由哪一层明确拒绝？如果答案只有 System Prompt，这条边界还不够牢。

**You can outsource your thinking, but you cannot outsource your understanding.**

## 官方参考

- [AI Red Teaming Agent](https://learn.microsoft.com/en-us/azure/foundry/concepts/ai-red-teaming-agent)
- [Run evaluations from the Microsoft Foundry portal](https://learn.microsoft.com/en-us/azure/foundry/how-to/evaluate-generative-ai-app)
- [Prompt Shields in Microsoft Foundry](https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/content-filter-prompt-shields)
