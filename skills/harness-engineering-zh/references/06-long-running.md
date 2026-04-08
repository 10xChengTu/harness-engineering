# 06 - Long-Running Tasks

Agent 处理跨越数小时、多个会话或超出 Context 窗口的任务时的模式。

## 两个核心问题

1. **Context 退化**：随着 Context 填满，Agent 失去连贯性，并可能"急于完成"
2. **状态丢失**：在会话之间，任何未保存在磁盘上的内容都会被遗忘

## Initializer + Worker 模式

将长任务分解为设置和执行：

### Initializer Agent
在开始时运行一次。创建：
- `init.sh` —— 环境设置脚本
- `progress.md` —— 跟踪已完成内容和下一步计划
- `features.json` —— 带有状态的结构化功能列表
- 初始 Git Commit —— 干净的基准线

### Worker Agent
迭代运行。在每个周期中：
1. 读取进度文件
2. 选择下一个未完成的功能
3. 实现它
4. 运行测试
5. 执行 Git Commit
6. 更新进度文件
7. 重复或执行 Handoff

## 进度跟踪

### 基于文件的进度（关键）
```markdown
# progress.md
更新时间：2025-01-15T14:30:00Z

## 已完成功能
- [x] 用户身份验证 (commit: abc123)
- [x] 数据库 Schema (commit: def456)

## 进行中
- [ ] Dashboard UI —— 布局已完成，图表待处理

## 剩余工作
- [ ] 设置页面
- [ ] 导出功能

## 已知问题
- Auth Token 刷新在会话过期时存在边界情况
- Dashboard 图表库版本冲突（目前固定在 2.x）

## 已做决策
- MVP 使用 SQLite，稍后将迁移到 PostgreSQL（见 docs/decisions/001.md）
```

**在每次 Commit 后更新。** 这是跨会话的记忆。

### 结构化功能跟踪
```json
{
  "features": [
    {
      "id": 1,
      "name": "User Auth",
      "status": "complete",
      "sprint": 1,
      "commits": ["abc123"],
      "tests": "passing",
      "notes": "JWT + refresh tokens"
    }
  ],
  "current_sprint": 3,
  "total_sprints": 8
}
```

## Context Reset vs Compaction

### Context Reset（完全替换）
- 终止当前 Agent，启动全新的 Agent
- 传递包含完整状态的 Handoff Artifact
- 优点：Context 干净，没有"焦虑感"，推理新鲜
- 缺点：延迟较高，必须在 Artifact 中对所有状态进行编码

### Compaction（就地总结）
- 总结早期的对话，在同一个会话中继续
- 优点：保持连续性，延迟较低
- 缺点：可能无法完全重置"Context 焦虑"，存在残留的混乱

### 决策指南
- 模型在长 Context 下表现下降 → Context Reset
- 模型能很好地处理长 Context → Compaction 即可
- 任务需要新鲜视角 → Context Reset
- 任务受益于连续性 → Compaction

## Handoff Artifacts

当一个 Agent 将工作交给另一个 Agent 时：

```markdown
# handoff.md

## 已完成工作
<带有 Commit 引用的已完成项目列表>

## 当前状态
<代码库现在的样子，运行中的服务，环境状态>

## 下一步工作
<剩余任务的有序列表>

## 关键 Context
<下一个 Agent 必须知道的决策、约束、注意事项>

## 已修改文件
<已更改的文件列表以及每个文件的具体更改内容>
```

**Handoff 必须包含足够的状态，以便新 Agent 能够在不阅读完整对话历史的情况下继续工作。**

## Git 作为检查点系统

```bash
# 在每个功能/逻辑单元完成后 Commit
git add -A && git commit -m "feat(auth): implement login flow"

# 标记里程碑
git tag -a sprint-1-complete -m "Sprint 1: Auth + DB schema"
```

如果出现问题，Agent（或人类）可以回滚到上一个已知的良好状态。

## 增量验证

不要等到最后才测试。在每个功能完成后：

```bash
# 边做边验证
npm run typecheck      # 类型是否仍然正确？
npm run test           # 测试是否仍然通过？
npm run dev            # 应用是否仍然可以运行？
```

及早发现错误可以防止难以调试的复合故障。

## 反模式 (Anti-Patterns)

- **过早宣告完成**：Agent 在未完成时宣告"Done"。对策：显式的功能清单 + 验证步骤。
- **范围漂移 (Scope drift)**：Agent 添加了未要求的特性。对策：结构化功能列表，Agent 对照检查。
- **未记录的状态**：Agent 进行了更改但未记录。对策：强制性的进度更新。
- **大爆炸式测试 (Big bang testing)**：仅在最后进行测试。对策：在每个功能后进行测试。
