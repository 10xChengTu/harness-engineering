# 03 - Constraints & Guardrails

Constraints increase agent autonomy by making incorrect paths fail fast. Paradox: more rules → more freedom.

## The "Relocating Rigor" Principle

Traditional dev: humans enforce quality through code review, conventions, experience.
Agent dev: encode that rigor into automated checks. Agents run freely within checked boundaries.

## Architecture Enforcement via Linters

Custom linters that enforce dependency direction and layer boundaries.

### Layer Architecture Example (OpenAI Pattern)

```
Types → Config → Repo → Service → Runtime → UI
```

Each layer can only import from layers to its left. A custom lint rule rejects violations automatically.

### Implementation

```javascript
// .eslintrc.js or custom lint script
// Rule: UI files cannot import from Repo directly
// Rule: Service files cannot import from UI
// Rule: Types files cannot import from anything except other Types

module.exports = {
  rules: {
    'no-restricted-imports': ['error', {
      patterns: [
        { group: ['../repo/*'], message: 'UI cannot import repo directly. Use service layer.' }
      ]
    }]
  }
}
```

### One Rule Per Mistake

When an agent makes a mistake, don't just fix it — add a lint rule or check that prevents it forever. This is how the harness learns.

```markdown
## .agents/rules.md (or in AGENTS.md)

### Rule: No direct DB queries in route handlers
Added: 2025-01-15
Reason: Agent put raw SQL in an Express handler, bypassing the service layer
Fix: All DB access goes through src/services/. Lint rule enforces this.
```

## Type Systems as Guardrails

Strong types prevent entire categories of agent errors:

- **TypeScript strict mode**: `strict: true` in tsconfig catches type mismatches at build time
- **Zod/Valibot schemas**: Runtime validation of API boundaries
- **Database schemas**: Typed ORM (Prisma, Drizzle) prevents schema drift

**Never suppress types**: No `as any`, `@ts-ignore`, `@ts-expect-error`. If the agent can't make types work, the design is wrong.

## Structured Tests as Contracts

Tests serve as executable specification that agents can verify against:

```typescript
// This test IS the spec. Agent reads it to understand expected behavior.
describe('UserService', () => {
  it('should hash password before storing', async () => {
    const user = await UserService.create({ email: 'test@test.com', password: 'plain' });
    expect(user.password).not.toBe('plain');
    expect(user.password).toMatch(/^\$2[aby]\$/); // bcrypt format
  });
});
```

### Upstream Test Suite Bridging

If building on top of an existing framework/library, bridge their test suite:

```bash
# Run upstream tests to verify compatibility
npm run test:upstream  # Ensures our changes don't break framework contracts
npm run test:ours      # Our own tests
```

This gives agents a feedback loop: "my change broke upstream compatibility."

## Safe Autonomy Boundaries

Define what agents CAN and CANNOT do:

```markdown
## Agent Permissions

### Allowed
- Create/modify files in src/
- Run tests
- Install dev dependencies
- Create git branches

### Forbidden
- Modify CI/CD config without approval
- Delete test files
- Push to main/master
- Modify .env with real credentials
- Install production dependencies without approval
```

## Git as Safety Net

```markdown
## Git Conventions for Agents

- Commit after each logical unit of work (not at the end)
- Commit message format: `type(scope): description`
- Never force push
- Branch per feature: `agent/feature-name`
```

Small, frequent commits let humans (and other agents) review incrementally and revert surgically.

## Pre-commit Hooks

```bash
# .husky/pre-commit or similar
npm run typecheck
npm run lint
npm run test -- --changed
```

Agent gets immediate feedback if a commit violates constraints. Fail fast, fix fast.
