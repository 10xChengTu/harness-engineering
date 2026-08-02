# 02 - Context Engineering

Agent 看到的内容决定了它们的操作。Context Engineering = 为 Agent 的最佳表现设计信息环境。

## 核心公式

**Prompt Engineering** = 你对 Agent 说什么
**Context Engineering** = 你给 Agent 看什么
**Harness Engineering** = 整个系统（Context + 约束 + 反馈 + 架构）
**Loop 工程 (Loop Engineering)** = 系统如何自己转起来（见 `08-loops.md`）

## 渐进式披露 (Progressive Disclosure)

不要把所有内容都塞进系统 Prompt 中。分层呈现信息：

1. **始终可见**（约 100 字）：项目名称、技术栈、关键规则
2. **按需提供**（AGENTS.md → docs/）：架构、约定、模式
3. **即时提供**（Design Notes、内联注释）：特定文件的 Context

### 实施

```
AGENTS.md (始终加载)
  → "有关架构详情，请参阅 docs/architecture.md"
  → "有关 API 约定，请参阅 docs/api.md"

docs/architecture.md (当 Agent 处理架构时加载)
  → "具体到 auth 子系统，请参阅 src/auth/DESIGN_NOTES.md"
```

**将 AGENTS.md 作为路由**：它不直接包含知识 —— 它将 Agent 路由到知识所在的地方。就像一本书的目录，而不是整本书。

### 指令膨胀 (Instruction Bloat)

AGENTS.md 应保持在 **50–200 行**。超过这个范围，信噪比下降，Agent 会出现"中间遗忘"效应 —— 长文档中间的指令获得的关注度远低于开头和结尾。

指令膨胀的迹象：
- AGENTS.md 超过 300 行
- 同一概念在不同章节中被重复陈述
- 从未触发的规则（模型本身已经掌握了）

**对策**：将详细内容移至 `docs/` 文件中。保持 AGENTS.md 仅作为路由索引，只内联那些关键的、始终需要的规则。

## 工作状态管理 (Working State Management)

Agent 在不同会话 (session) 间会丢失状态。设计显式的状态持久化机制：

### Progress 文件模式

```markdown
# progress.md

## 当前状态
- 功能 X：已完成 80%，auth 流程已完成，UI 待处理
- 功能 Y：未开始

## 已完成
- [x] 数据库 schema 迁移
- [x] /users 的 API 端点

## 已阻塞
- 需要外部服务的 API key（已于 2025-01-15 询问用户）

## 下一步计划
1. 完成功能 X 的 UI 组件
2. 为 auth 流程编写集成测试
```

### 功能列表 JSON 模式

```json
{
  "features": [
    {
      "name": "User Authentication",
      "status": "complete",
      "files": ["src/auth/*", "src/middleware/auth.ts"],
      "tests": "passing"
    },
    {
      "name": "Dashboard",
      "status": "in_progress",
      "files": ["src/pages/dashboard/*"],
      "tests": "not_written"
    }
  ]
}
```

## 提供地图，而非手册

与其提供分步骤的指令，不如给 Agent 提供方位导向：

**不好**（手册）：
```
步骤 1：打开 src/auth/login.ts
步骤 2：找到 handleLogin 函数
步骤 3：通过...添加速率限制
```

**好**（地图）：
```
Auth 系统位于 src/auth/。登录流程：login.ts → validate.ts → session.ts。
速率限制中间件位于 src/middleware/rateLimit.ts —— 请遵循其模式。
测试位于 src/auth/__tests__/ —— 每一个 auth 变更都需要对应的测试。
```

地图让 Agent 能够自主导航。手册则会让 Agent 在面临任何偏差时都变得脆弱。

## 跨会话 Context (Cross-Session Context)

在 Agent 会话之间能够存留下来的内容：
- 磁盘上的文件（AGENTS.md、docs/、DESIGN_NOTES.md、progress 文件）
- Git 历史（commit 信息、diff）
- 代码注释和 docstrings

不能存留的内容：
- 对话历史
- Agent 的内部推理过程
- 口头约定（"我们决定使用 X 方案"）

**规则**：如果一个决策很重要，就必须写入文件。口头的 Context 就是丢失的 Context。

要把跨会话记忆当作一个系统来设计——生命周期、整理、遗忘——见 `09-memory.md`。

## 将 Context Window 视为 RAM

- Context Window 填满 → Agent 失去连贯性
- **Context 焦虑 (Context Anxiety)**：当 Agent 接近 Context 上限时，会表现出"过早收敛"行为 —— 急于完成、跳过验证步骤、输出质量下降。这是一个被观察到的行为模式，不是比喻。
- **压缩 (Compaction)**（总结旧的 Context）：维持连续性，但不能完全消除 Context 焦虑
- **Context 重置**（全新的 Agent + 交接产物）：干净的开始，消除 Context 焦虑，需要良好的交接文档
- 选择取决于模型：某些模型能很好地处理长 Context，而有些则会性能下降

### 交接产物 (Handoff Artifact) 结构

重置 Context 时，交接文档必须包含：
1. 已完成的工作
2. 所有变更文件的当前状态
3. 下一步要做什么
4. 已做的决策及其原因
