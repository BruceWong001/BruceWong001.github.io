---
layout:     post
title:      GitHub Copilot CLI：终端里的全能AI伙伴
subtitle:   Copilot CLI（1）
date:       2026-05-09
author:     Bruce Wong
header-img: img/copilot_cli.webp
catalog: true
tags:
    - Agile
    - AI Coding
    - Copilot
    - AI
---

## 一、Copilot CLI 是什么？

2026 年 2 月，GitHub 正式发布了 **Copilot CLI** 的 GA（General Availability）版本。它不是 VS Code 插件的命令行移植版，而是一个**独立的、终端原生的 AI 编程 Agent**。

官方的定位很清晰：

> *"Copilot CLI has grown from a terminal assistant into a full agentic development environment—one that plans, builds, reviews, and remembers across sessions, all without leaving the terminal."*

翻译成人话：**它是在终端里能独立思考、规划、编码、审查，还能记住你项目习惯的 AI 伙伴。**

所有 Copilot 订阅用户都能用（Free/Pro/Business/Enterprise），不需要额外付费。

---

## 二、它和 VS Code Copilot 有什么区别？

这是最容易混淆的地方。很多人以为 CLI 就是"把 VS Code 里的 Copilot 搬到终端"，其实完全不是。

### 2.1 定位差异

| 维度 | VS Code Copilot | Copilot CLI |
|------|----------------|-------------|
| **运行环境** | IDE 内部 | 终端/命令行 |
| **交互模式** | 编辑器内联补全、侧边栏 Chat | 终端全屏对话 |
| **核心能力** | 实时编码辅助 | 复杂任务规划与执行 |
| **上下文切换** | 在 IDE 内完成 | **不离开终端** |

### 2.2 功能对比

| 功能 | VS Code Copilot | Copilot CLI |
|------|----------------|-------------|
| 代码补全 | ✅ 实时内联 | ❌ 不是主要功能 |
| Agent 模式 | ✅ 有 | ✅ 有，且更强大 |
| 多步骤复杂任务 | ✅ 支持 | ✅ **更擅长** |
| Git 操作 | ⚠️ 有限 | ✅ **原生集成** |
| GitHub.com 操作（PR/Issue） | ⚠️ 需切换浏览器 | ✅ **直接操作** |
| Plan Mode（先规划再执行） | ✅ 有 | ✅ **`Shift+Tab` 更完善** |
| 跨会话记忆 | ✅ 有 | ✅ **Repository Memory 更强** |
| CI/CD 集成 | ❌ 难 | ✅ **`-p` 参数程序化调用** |
| Shell 命令执行 | ⚠️ 间接 | ✅ **原生支持** |

### 2.3 一句话总结区别

**VS Code Copilot 是"你写代码时的副驾驶"**——实时补全、快速解释、单文件重构。

**Copilot CLI 是"能独立完成任务的全栈工程师"**——你可以说"帮我重构整个模块"，然后去做别的事，它后台执行完后告诉你结果。

---

## 三、什么时候用 CLI，什么时候用 VS Code？

### 适合 VS Code Copilot 的场景

- 日常编码时的实时补全
- 单文件内的快速重构
- 解释选中代码的功能
- 在编辑器内直接查看和确认变更

### 适合 Copilot CLI 的场景

- **复杂多步骤任务**：创建完整功能、跨文件修改
- **DevOps/自动化**：生成 CI/CD 配置、Terraform 脚本
- **GitHub 操作**：批量管理 PR、审查代码、创建 Issue
- **后台任务**：长时间运行的代码分析、安全扫描
- **纯终端环境**：SSH 远程服务器、Docker 容器、WSL
- **脚本化集成**：把 AI 能力嵌入到自动化工作流

---

## 四、Copilot CLI 的三大杀手锏

### 4.1 后台委托（Background Delegation）

最独特的功能。在提示前加 `&`，任务就在云端异步执行，终端立即释放。

```bash
# 前缀加 &，任务在后台运行
& "分析这个代码库的所有安全漏洞并生成报告"

# 终端不阻塞，你可以继续做其他事
# 之后用 /resume 恢复查看结果
```

**价值**：对于长时间任务（代码分析、大规模重构），不需要傻等。

### 4.2 程序化调用（Programmatic Mode）

适合集成到脚本和自动化流程。

```bash
# 单次调用，适合脚本集成
copilot -p "Show me this week's commits and summarize them" --allow-tool='shell(git)'

# 管道输入
./script-outputting-options.sh | copilot
```

**价值**：可以集成到 CI/CD、定时任务、shell 脚本中。

### 4.3 Plan Mode — 先规划再执行

按 `Shift+Tab` 进入 Plan Mode，Copilot 会：

1. 分析你的请求
2. 提出澄清问题
3. 构建结构化实施计划
4. 等你确认后再开始编码

**价值**：避免"AI 一上来就乱改代码"，先沟通清楚需求。

---

## 五、安装与快速启动

### 5.1 安装

| 平台 | 命令 |
|------|------|
| macOS | `brew install copilot-cli` |
| Linux | `brew install copilot-cli` 或 `curl -fsSL https://gh.io/copilot-install \| bash` |
| Windows | `winget install GitHub.Copilot` |
| npm | `npm install -g @github/copilot` |

### 5.2 认证

```bash
copilot
/login
```

### 5.3 快速开始

```bash
# 1. 进入项目目录
cd my-project

# 2. 启动交互模式
copilot

# 3. 初始化项目自定义指令（让 Copilot 理解你的技术栈和约定）
/init
# 生成 .github/copilot-instructions.md，包含项目上下文

# 4. 开始构建
"Create a React component for user authentication"

```

---

**理解AI，用好AI，让AI帮助自我进化，加油。**

## 参考资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://docs.github.com/copilot/how-tos/copilot-cli/cli-getting-started |
| 最佳实践 | https://docs.github.com/copilot/how-tos/copilot-cli/cli-best-practices |
| 命令参考 | https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference |
| Slash 命令速查 | https://github.blog/ai-and-ml/github-copilot/a-cheat-sheet-to-slash-commands-in-github-copilot-cli/ |
| 产品页面 | https://github.com/features/copilot/cli |
