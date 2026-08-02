<div align="right"><a href="README.md">English</a> | <strong>中文</strong></div>

# harness-engineering

一个 [Agent Skill](https://skills.sh)，用于搭建和优化 **Harness 工程** — 让 AI Agent 在你的代码库上高效工作的基础设施。

> **Harness = AI Agent 的操作系统。** 模型是 CPU，Context Window 是 RAM，Harness 是 OS。

## 安装

```bash
# English version
npx skills add 10xChengTu/harness-engineering/skills/harness-engineering

# 中文版
npx skills add 10xChengTu/harness-engineering/skills/harness-engineering-zh
```

## 功能介绍

这个 Skill 教你的 AI Agent 如何为任何项目构建和维护 Harness 层 — 包括 `AGENTS.md`、`docs/`、lint 规则、约束条件和评估系统，这些决定了 Agent 产出质量的好坏。

**核心原则：** 从简单开始，按需增加复杂度。每个 Harness 组件都编码了一个关于模型无法独立完成的假设。

### 触发场景

| 你说… | Skill 会做… |
|---|---|
| "为这个项目配置 AI Agent" | 完整的项目 Harness 搭建 |
| "创建一个 AGENTS.md" | 生成入口文件 + docs 目录结构 |
| "Agent 总是忽略约定" | 诊断 Harness 缺陷，而非模型问题 |
| "为什么它总是把 X 做错？" | 定位 Harness 层的根因 |
| "让 Agent 在这个代码库上工作得更好" | 评估并渐进式改进 Harness |

## 涵盖内容

该 Skill 包含 9 个参考模块，Agent 会按需查阅：

| 模块 | 涵盖内容 |
|---|---|
| **项目搭建** | `AGENTS.md` 结构、`docs/` 目录、设计笔记、初始化脚本 |
| **Context 工程** | Agent 看到什么、渐进式信息披露、工作状态管理 |
| **约束与护栏** | Linter、类型系统、架构强制、安全自治 |
| **多 Agent 架构** | Agent 分离、协调协议、委派模式 |
| **评估与反馈** | 测试 Agent 输出、评分、可观测性、反馈循环 |
| **长时间运行任务** | 进度追踪、Context 重置、交接产物 |
| **循环 (Loops)** | 验证器、停止条件、护栏、全新上下文迭代、循环栈 |
| **记忆 (Memory)** | 记忆生命周期、文件 vs 数据库、index-then-fetch、记忆整理、遗忘 |
| **诊断** | 当 Agent 表现不佳时 — 症状 → 根因映射 |

## 双语 Skill

本项目提供两个可安装的 Skill，内容相同但语言不同：

```bash
# English
npx skills add 10xChengTu/harness-engineering/skills/harness-engineering

# 中文
npx skills add 10xChengTu/harness-engineering/skills/harness-engineering-zh
```

## 为什么做这个

Agent 产出质量差几乎总是 Harness 的问题，而非模型的问题。当你的 Agent 忽略约定、做出错误假设或产出不一致时 — 解决方案是更好的 Context、约束和反馈循环，而不是更大的模型。

这个 Skill 编码了从真实 Agent 部署中学到的模式和反模式，让你不必重新踩坑。

## 兼容性

支持所有符合 [Agent Skills 规范](https://agentskills.io) 的 Agent，包括 Claude Code、OpenCode、Cursor、Codex、Cline、GitHub Copilot 等 [40+ 工具](https://github.com/vercel-labs/skills#supported-agents)。
