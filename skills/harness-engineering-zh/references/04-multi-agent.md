# 04 - Multi-Agent Architecture

何时使用多个 Agent，如何协调它们，以及通信协议。

## 何时使用 Multi-Agent

**Single agent** 适用场景：任务可以放入一个 Context 窗口，属于单一领域，且逻辑直接。
**Multi-agent** 适用场景：任务跨越多个领域，超出 Context 限制，受益于关注点分离，或需要独立的 Eval。

**从单 Agent 开始，遇到瓶颈再拆分。** 不要过早进行 Multi-agent 架构设计。

## 基础模式

### 1. Prompt Chaining
顺序流水线：Agent A 输出 → Agent B 输入 → Agent C 输入。
适用于任务有清晰阶段的场景。每个阶段都可以独立优化。

### 2. Routing
分类器将输入发送给专门的 Agent。类似调度员。
适用于输入差异很大且需要不同专业知识的场景。

### 3. Parallelization
多个 Agent 同时处理独立的子任务。
适用于任务可分解且不共享状态的场景。

### 4. Orchestrator-Workers
中央协调器委派给专门的 Worker。
适用于任务分解需要判断（而非预先确定）的场景。

### 5. Generator-Evaluator (受 GAN 启发)
一个 Agent 创建，另一个判断。不断迭代直到达到质量阈值。
适用于可以进行质量评估但自我评估不可靠的场景。

## Coordinator 设计

Coordinator（主 Agent）是最关键的部分。核心经验：

### 在委派中教导细节
Bad: "构建 Auth 系统"
Good: "构建 JWT Auth，包括：Refresh Tokens (7 天过期)，httpOnly Cookies，/auth/login 和 /auth/refresh 接口。使用 bcrypt 处理密码。遵循 src/auth/existing.ts 中的模式。"

**根据查询复杂度调整投入** —— 简单问题使用简单委派，复杂问题提供详细计划。

### 通信协议

#### 基于文件的通信
Agent 通过磁盘上的文件进行通信。可靠、可审计，且在崩溃后仍能存续。

```
.harness/
├── plan.md          # Planner 写入，其他 Agent 读取
├── sprint-contract.md  # Generator + Evaluator 协商
├── eval-report.md   # Evaluator 写入，Generator 读取
└── handoff.md       # 当前 Agent 为下一个 Agent 写入
```

#### Intent Marker 协议（结构化标签）
对于 Supervisor-Worker 模式，使用显式标签：

| 标签 | 含义 |
|-----|---------|
| `[STATUS_REQUEST]` | Supervisor 询问进度 |
| `[REVIEW_REQUEST]` | Worker 提交供评审 |
| `[ACK]` | 确认收到 |
| `[ESCALATE]` | Worker 无法解决，需要帮助 |

**每次交换最多 3 条消息** —— 防止 Agent 之间出现无限循环。

#### Sprint Contract 模式
在工作开始前，Generator 和 Evaluator 就以下内容达成一致：
1. 将要构建的内容
2. 如何验证已完成（可测试的准则）
3. 质量阈值

这弥补了高层级规格说明与实现验证之间的差距。

## 关注点分离

### 为什么分离 Generator 和 Evaluator
- **自我评估偏见**：Agent 对自己的工作评分过高
- **更容易校准**：调优一个持怀疑态度的 Evaluator 比让 Generator 进行自我批评更容易
- **聚焦 Context**：每个 Agent 都有为其角色优化的 Context

### Evaluator 何时产生价值
当任务处于 **模型能力边缘** 时，Evaluator 的成本才是值得的。随着模型能力的提升，这一边界也在移动 —— 以前需要评估的任务现在可能不再需要。请定期重新评估。

## Sub-Agent 委派

```markdown
## Delegation Template
1. TASK: 具体的原子目标
2. CONTEXT: 相关文件、模式、约束
3. DELIVERABLE: 要产出的内容
4. CONSTRAINTS: 禁止做的事情
5. VERIFICATION: 如何判断已完成
```

### 让 Agent 自我改进
允许 Sub-Agent 更新自己的工具和模式。如果一个 Agent 发现了一个更好的方法，它应该能够将其编码以供未来运行。

## 先广后窄搜索
对于研究/探索类 Agent：
1. 首先撒大网（探索多种可能性）
2. 然后收窄到有前景的路径
3. 不要对找到的第一个解决方案过早承诺
