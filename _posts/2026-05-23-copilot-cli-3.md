---
layout:     post
title:      GitHub Copilot CLI：代码写完了，但工程还没结束
subtitle:   Copilot CLI（3）
date:       2026-05-23
author:     Bruce Wong
header-img: img/copilot_cli.webp
catalog: true
tags:
    - Agile
    - AI Coding
    - Copilot
    - AI
---

今天继续第三部分。功能已经开发完成、测试全部通过，但**工程工作还没结束**——代码需要被理解、被记录、被传承。

---

## 问题：AI 写完了代码，然后呢？

第二部分结束时，我们有了一个完整的功能：PDF 转 Markdown。18 个单元测试全部通过，代码质量合格。

但一个可交付的项目，除了功能代码还需要什么？

- **README/Architecture Docs**：后续维护者如何理解这个项目？
- **Commit Message**：代码历史的叙事线索
- **Git 提交**：所有改动作为一个完整的故事单元

传统做法是：开发者手动写文档、手动整理 Changelog、手动写 Commit。现实很多时候是人们不愿意做这些重复性工作，往往这些作为项目记忆存在的东西被简单化或者彻底省略了。既然 Copilot 参与了整个开发过程，它理应**比任何人都清楚**改了什么、为什么改。

所以第三部分的核心问题是：**如何让 AI 完成工程闭环？**

---

## 实战：让 Copilot 总结架构并提交代码

### 第一步：验证功能正确性

在让 Copilot 总结之前，先确保功能确实可用。拿一个真实 PDF 文件测试转换：

```bash
python -c "
from clio import convert_pdf
print(convert_pdf('path/to/file.pdf', 'output/'))
"
```

验证输出内容的完整性和格式正确性。这是后续所有文档和提交的基础——**只有经过验证的代码才值得被记录**。

### 第二步：让 Copilot 更新 README/Architecture Docs

关键指令：

> "总结当前项目的改动和架构，更新到 README 文件中，作为长期记忆，方便后续 AI 维护项目使用。"

注意这里的措辞——不是"帮我写文档"，而是**"建立长期记忆"**。这个 framing 很重要：
- 它暗示了文档的**持续性价值**
- 它明确了**目标读者**（后续的 AI 或人类维护者）
- 它设定了**更新频率**（每次重大改动后）

Copilot 生成的 README 包含：

- **功能描述**：将 PDF 文档转换为 Markdown 格式，保留文本层级和基础排版
- **运行方式**：安装、转换命令、测试脚本
- **架构决策**：使用 `pdf-parse` 提取文本、自定义规则引擎识别标题层级、输出符合你定制的标准

这份文档成为项目的"长期记忆"——下次无论是谁接手，都能快速进入上下文。当然他不止与README文件中，还可以是你要求的独立目录下面的一个系列文档。类似于Skill的方式让AI可以渐进式读取和加载。

### 第三步：让 Copilot 撰写 Commit Message

接下来是最容易忽视但最重要的环节。

为什么让 AI 写 Commit Message？
- Commit 是代码历史的"索引"，好的 Commit 让 `git log` 成为可读的叙事
- Copilot 参与了全部开发，它的**全局视角**比人类更适合总结改动
- 这是 AI 协作的**信任凭证**——人类审查者通过 Commit 理解 AI 做了什么

指令：

> "总结当次改动内容，撰写 Commit Message，遵循 Conventional Commits 规范。"

### 第四步：提交到仓库

最后一步：将所有改动（代码、测试、文档）一起提交。你只需要使用Command让Copilot CLI代替你执行就可以了。

---

## 关键洞察：文档即记忆，Commit 即叙事

### 1. 为什么让 AI 维护 README？

在 AI 协作的时代，README 不是"给别人看的说明书"，而是"给未来 AI 的上下文窗口"。

想象一下：三个月后，你或另一个 AI 需要在这个项目上继续开发。打开 README，看到的是：
- 功能边界在哪里
- 架构决策为什么这样选
- 测试如何运行

而不是需要翻遍代码才能理解。

### 2. 为什么让 AI 写 Commit Message？

人类写 Commit 往往只记录"改了什么文件"，但 AI 能总结：
- **改了什么**（功能层面）
- **为什么改**（需求驱动）
- **影响范围**（依赖、测试、文档）

这种全局视角是 AI 的优势，应该被利用。

### 3. 一气呵成的工作流

开发 → 测试 → 文档 → 提交，四个环节在同一个会话中完成。

不需要：
- ❌ 切换上下文去写文档
- ❌ 回忆"刚才改了什么"来写 Commit
- ❌ 等待"文档补写"的 TODO

---

## 写在最后

Copilot CLI 的价值不只是"在终端里写代码"，而是**重新定义了人机协作的边界**。

传统开发中，人类完成"创造性工作"（设计、架构），AI 辅助"机械性工作"（编码、格式化）。但这个系列展示的是另一种可能：

- **Plan 模式**：人类定方向，AI 拆解计划
- **Autopilot 模式**：AI 执行，人类监督
- **文档 & Commit**：AI 总结，人类验收

边界是动态的，不是固定的。关键是找到每个环节最合适的分工。

---

<iframe
  src="//player.bilibili.com/player.html?bvid=BV1dTGi64E6R&cid=38552863578&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen>
</iframe>
---

**理解AI，用好AI，让AI帮助自我进化，加油。**

