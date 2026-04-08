# 01 - Project Setup

如何设置项目的的基础 Harness 文件，以便 Agent 能够高效工作。

## AGENTS.md (入口点)

AGENTS.md 是一个**目录**，而不是百科全书。它告诉 Agent 在哪里可以找到所需内容，而不是包含它需要知道的所有信息。

### 结构

```markdown
# 项目名称

## 快速开始
<1-3 条运行命令>

## 架构概览
<2-3 句话 + 指向 docs/architecture.md 的指针>

## 目录结构
<关键目录树及单行描述>

## 关键约定
<Agent 必须遵守的 5-10 条规则 —— 仅包含最重要的规则>

## 文档地图
<按主题指向 docs/ 文件的指针>

## 常见任务
<任务 → 相关文件/命令的映射>
```

### 反模式 (Anti-patterns)
- 将整个代码库的知识全部堆砌在 AGENTS.md 中（太长，Agent 会跳读）
- 完全没有 AGENTS.md（Agent 只能靠猜）
- AGENTS.md 内容陈旧且与现实冲突（比没有文件更糟糕）

## docs/ 目录 (记录系统)

`docs/` 目录是存放详细知识的地方。AGENTS.md 指向这里。

### 推荐结构

```
docs/
├── architecture.md      # 系统设计，组件关系
├── conventions.md       # 编码标准，命名，模式
├── api.md              # API 契约，端点
├── data-model.md       # 数据库 schema，数据流
├── testing.md          # 测试策略，如何运行，测试什么
├── deployment.md       # 构建，部署，环境
└── decisions/          # 架构决策记录 (ADRs)
    └── 001-chose-x.md
```

### 为 Agent 编写文档

- **明确说明"为什么"** —— 当 Agent 理解背后的原因时，能更好地遵守规则
- **包含示例** —— 展示模式，而不仅仅是描述它
- **保持文件聚焦** —— 每个文件一个主题，理想长度 <300 行
- **为超过 100 行的文件添加目录 (TOC)**
- **注明日期和版本** —— Agent 需要知道哪些内容是当前的

## 源码树中的设计笔记 (Design Notes)

在 Agent 会遇到的地方嵌入 Context —— 就在源码树本身。

```
src/
├── components/
│   ├── DESIGN_NOTES.md    # 为什么组件要这样构建
│   └── Button/
├── api/
│   ├── DESIGN_NOTES.md    # API 设计原则
│   └── routes/
```

这些文件在不同会话 (session) 间持久存在。它们是跨会话的记忆，防止 Agent 重新做决策或违背过去的决定。

## init.sh 模式 (用于自动化设置)

对于在 Agent 开始工作前需要环境设置的项目：

```bash
#!/bin/bash
# init.sh - 在 Agent 开始工作前运行
set -e

# 安装依赖
npm install  # 或 pip install, cargo build 等。

# 设置本地配置
cp .env.example .env.local

# 验证设置
npm run typecheck
npm test -- --run

echo "环境已就绪，Agent 可以开始工作"
```

## 初始 commit 约定

设置好 Harness 文件后，进行初始 commit：
```
git add -A && git commit -m "harness: initial project setup"
```

这为 Agent 提供了一个干净的基准线，以便在需要时进行 diff 或回滚。

## 质量评分 (可选)

对于较大的 docs/，添加质量元数据：

```markdown
---
quality: 0.8
last_verified: 2025-01-15
owner: @username
---
```

这有助于 Agent（以及人类）了解哪些文档值得信任，哪些需要更新。
