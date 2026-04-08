# 05 - Eval & Feedback

如何评估 Agent 的输出，并创建驱动改进的反馈循环。"无法衡量的东西，就无法改进。"

## Eval-Driven Development

在构建功能之前先构建 Eval。类似于针对 Agent 行为的 TDD。

### Eval 结构

每个 Eval 由三部分组成：
1. **Task**：Agent 必须完成的任务（输入 + 指令）
2. **Trial**：对任务的一次尝试（每个任务可能运行多次 Trial）
3. **Grader**：如何评判输出

### Grader 类型

| 类型 | 适用场景 | 示例 |
|------|----------|---------|
| **Code-based** | 输出可以通过编程方式验证 | 文件存在，测试通过，类型检查通过 |
| **Model-based** | 输出需要判断 | "这段代码结构是否良好？" |
| **Human** | 主观质量很重要 | 设计美感，UX 质量 |

### 指标

- **pass@k**：k 次尝试中至少有 1 次成功的概率。适用于 Agent 只需要成功一次的场景。
- **pass^k**：k 次尝试全部成功的概率。适用于对可靠性要求极高的场景。

### 起步
从源自 **真实失败案例** 的 20-50 个任务开始。不要凭空捏造抽象的测试用例 —— 捕捉 Agent 在你的项目中遇到的实际问题。

## "让 AI 检查 AI"模式

使用一个独立的 Agent 来验证第一个 Agent 的工作。比自我检查更可靠。

### Garbage Collection Agents

定期扫描代码库的 Agent，寻找：
- 代码与文档之间的一致性问题
- 死代码或未使用的 imports
- 违反约定的地方
- 陈旧的 TODO 或 fixme

**如何运行**：安排一个定期的 Agent 任务（每周或在重大更改后），使用如下 Prompt：

> "扫描此代码库，查找 docs/ 与实际代码之间的不一致之处。报告差异。"

通过 Agent 的 headless/非交互模式运行（例如 CLI one-shot、CI 流水线任务或计划自动化）。具体的调用方式取决于你的工具链 —— 重要的是这种模式：**自动化的、定期的、Agent 驱动的代码库审计**。

这是代码库层面的 Garbage Collector —— 在偏差演变成债务之前发现它。

## Agent 可读的可观测性

Agent 需要看到自己的遥测数据：

### 结构化日志
```typescript
// Agent 可以解析和推理的日志
logger.info('auth.login', {
  userId: user.id,
  duration_ms: 145,
  success: true,
  method: 'jwt'
});
```

### 错误报告
当发生错误时，生成 Agent 可读的报告：
```markdown
## Error Report
- **失败内容**：POST /api/users 返回 500
- **Stack trace**：src/services/user.ts:42 → src/db/queries.ts:18
- **近期修改**：修改了 user.ts (commit abc123)
- **可能原因**：user.email 缺少空值检查
```

## 反馈循环设计

### 保存时测试 (Test-on-Save)
```json
// package.json
{
  "scripts": {
    "dev": "concurrently 'vite' 'vitest --watch'",
    "check": "tsc --noEmit && eslint . && vitest --run"
  }
}
```

Agent 在每次修改后都能获得即时反馈。

### 浏览器自动化验证
对于前端工作，使用 Playwright/Puppeteer MCP 来：
1. 导航到运行中的应用
2. 截图
3. 与 UI 元素交互
4. 验证视觉和功能的正确性

Evaluator Agent 不仅仅看代码 —— 它像人类一样 **使用** 应用。

### 差异化评估 (Differential Evaluation)
将 Agent 输出与已知的良好参考进行比较：
- 针对 Agent 的更改运行上游测试套件
- 对比修改前后的截图
- 比较性能指标

## 评分标准 (Scoring Rubrics)（针对主观质量）

当质量是主观的时候，创建显式的评分标准：

```markdown
## Design Quality Rubric
- **5**：具有独特身份的凝聚整体。明显的自定义创意选择。
- **4**：带有某些独特元素的扎实设计。轻微的模板感。
- **3**：合格但通用。可以是任何模板。
- **2**：功能完备但视觉平淡。默认组件库的感觉。
- **1**：布局错乱，间距不一致，颜色冲突。
```

评分标准将"它好吗？"转换为"它是否满足准则 X, Y, Z？" —— 这是 Agent（以及 Evaluator Agent）可以真正回答的问题。

## 作为 Artifact 的执行计划

当 Agent 规划其工作时，将该计划捕获为一个文件：

```markdown
# execution-plan.md
## 目标：添加用户设置页面
## 步骤：
1. 在 src/pages/ 中创建 SettingsPage 组件
2. 在 src/router.ts 中添加路由
3. 创建设置 API 接口
4. 在用户模型中添加设置字段
5. 编写测试
## 依赖项：User 模型（已存在），Router（已存在）
## 预估复杂度：中等
```

计划是可审计、可评审的，并且可以在开始执行前进行评估。
