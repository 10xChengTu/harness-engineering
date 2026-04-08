# Harness Engineering

> Harness = AI Agent 的操作系统。模型是 CPU，Context Window 是 RAM，Harness 是 OS。

## 快速开始 / Quick Start

```bash
# English version
npx skills add 10xChengTu/harness-engineering

# 中文版
npx skills add 10xChengTu/harness-engineering-zh
```

## 架构概览 / Architecture Overview

本项目是一个 [Agent Skill](https://skills.sh)，用于为代码仓库搭建和优化 **Harness 工程** —— 让 AI Agent 在你的代码库上高效工作的基础设施。

## 目录结构 / Directory Structure

```
harness-engineering/
├── AGENTS.md                          # 本文件 —— Agent 入口 / Agent entry point
├── README.md                          # 项目主 README（英文 + 双语导航）
├── ref/index.md                       # 参考文献索引（原始英文）
├── skills/harness-engineering/        # Skill 定义（英文版）
│   ├── SKILL.md
│   └── references/
└── skills/harness-engineering-zh/     # Skill 定义（中文版）
    ├── SKILL.md
    └── references/
```

## 关键约定 / Key Conventions

1. **英文为 Source of Truth**：所有内容先写英文，再翻译为中文
2. **技术名词不翻译**：Agent、Token、Harness、Prompt、Context、MCP、JWT、eval、lint 等技术术语保留英文
3. **代码块不翻译**：所有代码示例保持英文原样
4. **双语 Skill 同步**：`skills/harness-engineering/` 和 `skills/harness-engineering-zh/` 内容必须保持同步（见下方"Skill 同步规则"）

## 文档地图 / Documentation Map

### Skill 定义（可安装 Skill）

| 版本 | SKILL.md | references/ |
|------|----------|-------------|
| **English** | `skills/harness-engineering/SKILL.md` | `skills/harness-engineering/references/01~07` |
| **中文** | `skills/harness-engineering-zh/SKILL.md` | `skills/harness-engineering-zh/references/01~07` |

### 参考模块（7 个）

| 模块 | 主题 | 英文 Skill | 中文 Skill |
|------|------|------------|------------|
| 01 | Project Setup | `skills/harness-engineering/references/01-project-setup.md` | `skills/harness-engineering-zh/references/01-project-setup.md` |
| 02 | Context Engineering | `skills/harness-engineering/references/02-context-engineering.md` | `skills/harness-engineering-zh/references/02-context-engineering.md` |
| 03 | Constraints & Guardrails | `skills/harness-engineering/references/03-constraints.md` | `skills/harness-engineering-zh/references/03-constraints.md` |
| 04 | Multi-Agent Architecture | `skills/harness-engineering/references/04-multi-agent.md` | `skills/harness-engineering-zh/references/04-multi-agent.md` |
| 05 | Eval & Feedback | `skills/harness-engineering/references/05-eval-feedback.md` | `skills/harness-engineering-zh/references/05-eval-feedback.md` |
| 06 | Long-Running Tasks | `skills/harness-engineering/references/06-long-running.md` | `skills/harness-engineering-zh/references/06-long-running.md` |
| 07 | Diagnosis | `skills/harness-engineering/references/07-diagnosis.md` | `skills/harness-engineering-zh/references/07-diagnosis.md` |

### 参考文献索引
- `ref/index.md`

## 常见任务 / Common Tasks

| 任务 | 命令/操作 |
|------|-----------|
| 修改英文 Skill | 编辑 `skills/harness-engineering/` 下对应文件，然后同步更新 `skills/harness-engineering-zh/` 对应文件 |
| 添加新参考模块 | 在两个 Skill 的 `references/` 目录下同时创建 |
| 安装英文 Skill | `npx skills add 10xChengTu/harness-engineering` |
| 安装中文 Skill | `npx skills add 10xChengTu/harness-engineering-zh` |

## Agent 工作规则

### 修改 Skill 时（Skill 同步规则）

`skills/harness-engineering/`（英文）和 `skills/harness-engineering-zh/`（中文）是两个独立可安装的 Skill，内容必须保持同步。

**文件对应关系：**

| 英文 Skill | 中文 Skill |
|---|---|
| `skills/harness-engineering/SKILL.md` | `skills/harness-engineering-zh/SKILL.md` |
| `skills/harness-engineering/references/01-project-setup.md` | `skills/harness-engineering-zh/references/01-project-setup.md` |
| `skills/harness-engineering/references/02-context-engineering.md` | `skills/harness-engineering-zh/references/02-context-engineering.md` |
| `skills/harness-engineering/references/03-constraints.md` | `skills/harness-engineering-zh/references/03-constraints.md` |
| `skills/harness-engineering/references/04-multi-agent.md` | `skills/harness-engineering-zh/references/04-multi-agent.md` |
| `skills/harness-engineering/references/05-eval-feedback.md` | `skills/harness-engineering-zh/references/05-eval-feedback.md` |
| `skills/harness-engineering/references/06-long-running.md` | `skills/harness-engineering-zh/references/06-long-running.md` |
| `skills/harness-engineering/references/07-diagnosis.md` | `skills/harness-engineering-zh/references/07-diagnosis.md` |

**同步规则：**
1. 修改英文 Skill 的任何文件后，必须同步修改中文 Skill 的对应文件（翻译新增/变更的内容）
2. 修改中文 Skill 的任何文件后，必须同步修改英文 Skill 的对应文件（将变更翻译为英文）
3. 新增参考模块时，必须在两个 Skill 的 `references/` 目录下同时创建
4. 删除参考模块时，必须同时删除两个 Skill 中的对应文件
5. SKILL.md 的 frontmatter 差异是正常的：英文版 `name: harness-engineering`，中文版 `name: harness-engineering-zh`
6. 这是手动同步，不是脚本化翻译 —— 每次修改时由 Agent 或人工确保两边一致

### 翻译规则
- 技术名词保留英文（Agent、Harness、Token、Prompt、Context、eval、lint、MCP 等）
- 代码块保持原样
- 保持 Markdown 格式一致
