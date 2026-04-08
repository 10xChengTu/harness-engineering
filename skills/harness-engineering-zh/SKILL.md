---
name: harness-engineering-zh
description: "为 AI Agent 友好的代码库搭建和改进 Harness 工程（包括 AGENTS.md、docs/、Lint 规则、Eval 系统、项目级 Prompt 工程）。触发场景：为 AI Agent 设置新项目/空项目，创建 AGENTS.md 或 CLAUDE.md，关于 Harness 工程的问题，让 Agent 在代码库上更高效地工作。当用户感到沮丧或抱怨 Agent 质量时也会触发（例如：'Agent 总是无视规范'、'它从不听从指令'、'为什么它总是做错 X'、'Agent 坏了'）— 因为 Agent 输出质量差几乎总是意味着 Harness 缺失，而不是模型问题。涵盖：Context 工程、架构约束、多 Agent 协作、评估、长运行任务 Harness 以及 Agent 质量问题诊断。"
---

# Harness Engineering

Harness = 为项目中工作的 AI Agent 提供的操作系统。Model 是 CPU，Context 窗口是 RAM，Harness 则是操作系统。

## 核心原则

**从简单开始，仅在必要时增加复杂度。** 每一个 Harness 组件都代表了对模型无法独立完成任务的一种假设。要对这些假设进行压力测试 — 随着模型能力的提升，这些假设会失效。为"删除"而构建。

## 何时激活此 Skill

| 信号 | 行动 |
|--------|--------|
| 空项目/新项目 | → 进行完整的项目设置 (Section 1) |
| 用户对 Agent 感到沮丧 | → 诊断并修复 Harness 缺失 (Section 7) |
| 现有项目需要改进 | → 评估并逐步改进 |
| 明确的 Harness 问题 | → 参考相关章节 |

## 工作流

### 对于新项目

1. **评估 (Assess)** — 项目是什么？技术栈？团队规模？Agent 将如何被使用？
2. **设置 (Setup)** — 创建基础 Harness 文件 → 阅读 `references/01-project-setup.md`
3. **Context** — 设计信息架构 → 阅读 `references/02-context-engineering.md`
4. **约束 (Constraints)** — 添加护栏和 Linters → 阅读 `references/03-constraints.md`
5. **评估 (Evaluate)** — 设置反馈循环 → 阅读 `references/05-eval-feedback.md`
6. 如果项目涉及多 Agent 或长任务 → 阅读 `references/04-multi-agent.md`, `references/06-long-running.md`

### 对于诊断 (Agent 表现不佳)

1. 立即阅读 `references/07-diagnosis.md`
2. 识别是哪一层 Harness 出现了问题
3. 从相关参考文档中应用针对性修复

### 对于逐步改进

评估当前 Harness 的成熟度，识别最薄弱的层级，一次改进一个层级。

## Harness 层级 (快速参考)

| 层级 | 内容 | 参考文档 |
|-------|------|-----------|
| **项目设置 (Project Setup)** | AGENTS.md, docs/, 目录规范 | `01-project-setup.md` |
| **Context 工程 (Context Engineering)** | Agent 看到的信息、渐进式展示、工作状态 | `02-context-engineering.md` |
| **约束与护栏 (Constraints & Guardrails)** | Linters、类型系统、架构强制执行、安全自主权 | `03-constraints.md` |
| **多 Agent 架构 (Multi-Agent Architecture)** | Agent 分离、协作协议、委派模式 | `04-multi-agent.md` |
| **Eval 与反馈 (Eval & Feedback)** | 测试、评分、GC Agent、可观测性 | `05-eval-feedback.md` |
| **长运行任务 (Long-Running Tasks)** | 进度跟踪、Context 重置、交付产物 | `06-long-running.md` |
| **诊断 (Diagnosis)** | 当 Agent 失败时 — 在 Harness 中识别根因，而非模型 | `07-diagnosis.md` |

## 自我更新协议

当你在项目中发现新的可复用 Harness 模式时：

1. 识别它属于哪个参考文件（或者是否需要一个新文件）
2. 添加该模式，包括：它解决了**什么**问题，**何时**使用，以及如何**实现**它
3. 保持简洁 — 拒绝废话，只保留模式
