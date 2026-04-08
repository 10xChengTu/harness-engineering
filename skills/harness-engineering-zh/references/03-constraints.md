# 03 - Constraints & Guardrails

约束通过让错误的路径快速失败来增加 Agent 的自主性。悖论：规则越多 → 自由越多。

## "严谨性转移"原则 (Relocating Rigor)

传统开发：人类通过代码审查、约定和经验来保证质量。
Agent 开发：将这种严谨性编码到自动化检查中。Agent 在经过检查的边界内自由运行。

## 通过 Linter 强制执行架构

使用自定义 Linter 来强制执行依赖方向和层级边界。

### 层级架构示例 (OpenAI 模式)

```
Types → Config → Repo → Service → Runtime → UI
```

每一层只能从其左侧的层级进行 import。自定义 lint 规则会自动拒绝违规行为。

### 实施

```javascript
// .eslintrc.js 或自定义 lint 脚本
// 规则：UI 文件不能直接从 Repo import
// 规则：Service 文件不能从 UI import
// 规则：Types 文件除了其他 Types 之外，不能从任何地方 import

module.exports = {
  rules: {
    'no-restricted-imports': ['error', {
      patterns: [
        { group: ['../repo/*'], message: 'UI 无法直接 import repo。请使用 service 层。' }
      ]
    }]
  }
}
```

### 每个错误对应一条规则

当 Agent 犯错时，不要只是修复它 —— 添加一条 lint 规则或检查项，永久防止该错误再次发生。这就是 Harness 进化的方式。

```markdown
## .agents/rules.md (或在 AGENTS.md 中)

### 规则：禁止在 route handler 中直接进行 DB 查询
添加日期：2025-01-15
原因：Agent 在 Express handler 中放置了原始 SQL，绕过了 service 层
修复：所有 DB 访问必须通过 src/services/。通过 lint 规则强制执行。
```

## 将类型系统作为护栏 (Guardrails)

强类型可以防止整类 Agent 错误：

- **TypeScript 严格模式**：tsconfig 中的 `strict: true` 在构建时捕捉类型不匹配
- **Zod/Valibot schemas**：API 边界的运行时验证
- **数据库 schemas**：类型化 ORM (Prisma, Drizzle) 防止 schema 漂移

**永远不要压制类型错误**：不要使用 `as any`、`@ts-ignore`、`@ts-expect-error`。如果 Agent 无法让类型正常工作，说明设计有问题。

## 将结构化测试作为契约

测试作为可执行的规范，Agent 可以根据其进行验证：

```typescript
// 这个测试本身就是规范。Agent 通过阅读它来理解预期行为。
describe('UserService', () => {
  it('应该在存储前对密码进行 hash', async () => {
    const user = await UserService.create({ email: 'test@test.com', password: 'plain' });
    expect(user.password).not.toBe('plain');
    expect(user.password).toMatch(/^\$2[aby]\$/); // bcrypt 格式
  });
});
```

### 上游测试套件桥接 (Upstream Test Suite Bridging)

如果是在现有框架/库的基础上构建，请桥接它们的测试套件：

```bash
# 运行上游测试以验证兼容性
npm run test:upstream  # 确保我们的变更不会破坏框架契约
npm run test:ours      # 我们自己的测试
```

这为 Agent 提供了一个反馈循环："我的变更破坏了上游兼容性。"

## 安全自主权边界

定义 Agent 可以做和不可以做的事情：

```markdown
## Agent 权限

### 允许
- 创建/修改 src/ 中的文件
- 运行测试
- 安装开发依赖 (dev dependencies)
- 创建 git 分支

### 禁止
- 未经批准修改 CI/CD 配置
- 删除测试文件
- Push 到 main/master
- 使用真实凭据修改 .env
- 未经批准安装生产依赖 (production dependencies)
```

## 将 Git 作为安全网

```markdown
## Agent 的 Git 约定

- 在完成每个逻辑工作单元后进行 commit（不要等到最后）
- Commit 信息格式：`type(scope): description`
- 严禁 force push
- 每个功能一个分支：`agent/feature-name`
```

细粒度、高频次的 commit 让代码审查（由人类或其他 Agent 执行）可以增量进行，并能进行精确回滚。

## Pre-commit Hooks

```bash
# .husky/pre-commit 或类似工具
npm run typecheck
npm run lint
npm run test -- --changed
```

如果 commit 违反了约束，Agent 会立即得到反馈。快速失败，快速修复。
