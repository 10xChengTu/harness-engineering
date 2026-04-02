# 05 - Eval & Feedback

How to evaluate agent output and create feedback loops that drive improvement. "What you can't measure, you can't improve."

## Eval-Driven Development

Build evals BEFORE building features. Like TDD but for agent behavior.

### Eval Structure

Every eval has three parts:
1. **Task**: What the agent must accomplish (input + instructions)
2. **Trial**: One attempt at the task (may run multiple trials per task)
3. **Grader**: How to judge the output

### Grader Types

| Type | Use When | Example |
|------|----------|---------|
| **Code-based** | Output is verifiable programmatically | File exists, test passes, type checks |
| **Model-based** | Output needs judgment | "Is this code well-structured?" |
| **Human** | Subjective quality matters | Design aesthetics, UX quality |

### Metrics

- **pass@k**: Probability of at least 1 success in k attempts. Use when agent needs to succeed at least once.
- **pass^k**: Probability of success on ALL of k attempts. Use when reliability matters.

### Starting Point
Start with 20-50 tasks derived from **real failures**. Don't invent abstract test cases — capture actual problems agents hit in your project.

## "Let AI Check AI" Pattern

Use a separate agent to verify the first agent's work. More reliable than self-checking.

### Garbage Collection Agents

Periodic agents that scan the codebase for:
- Inconsistencies between code and docs
- Dead code or unused imports
- Convention violations
- Stale TODOs or fixmes

```bash
# Run weekly or after major changes
claude -p "Scan this codebase for inconsistencies between docs/ and actual code. Report discrepancies."
```

This is the codebase equivalent of a garbage collector — finds drift before it becomes debt.

## Agent-Readable Observability

Agents need to see their own telemetry:

### Structured Logging
```typescript
// Logs that agents can parse and reason about
logger.info('auth.login', {
  userId: user.id,
  duration_ms: 145,
  success: true,
  method: 'jwt'
});
```

### Error Reports
When an error occurs, generate agent-readable reports:
```markdown
## Error Report
- **What failed**: POST /api/users returned 500
- **Stack trace**: src/services/user.ts:42 → src/db/queries.ts:18
- **Recent changes**: Modified user.ts (commit abc123)
- **Likely cause**: Missing null check on user.email
```

## Feedback Loop Design

### Test-on-Save
```json
// package.json
{
  "scripts": {
    "dev": "concurrently 'vite' 'vitest --watch'",
    "check": "tsc --noEmit && eslint . && vitest --run"
  }
}
```

Agent gets instant feedback on every change.

### Browser Automation Verification
For frontend work, use Playwright/Puppeteer MCP to:
1. Navigate the running app
2. Take screenshots
3. Interact with UI elements
4. Verify visual and functional correctness

The evaluator agent doesn't just look at code — it **uses** the app like a human would.

### Differential Evaluation
Compare agent output against known-good reference:
- Run upstream test suite against agent's changes
- Diff screenshots before/after
- Compare performance metrics

## Scoring Rubrics (For Subjective Quality)

When quality is subjective, create explicit rubrics:

```markdown
## Design Quality Rubric
- **5**: Cohesive whole with distinct identity. Custom creative choices evident.
- **4**: Solid design with some unique elements. Minor template-ness.
- **3**: Competent but generic. Could be any template.
- **2**: Functional but visually flat. Default component library feel.
- **1**: Broken layout, inconsistent spacing, clashing colors.
```

Rubrics convert "is it good?" into "does it meet criteria X, Y, Z?" — which agents (and evaluator agents) can actually answer.

## Execution Plans as Artifacts

When an agent plans its work, capture that plan as a file:

```markdown
# execution-plan.md
## Goal: Add user settings page
## Steps:
1. Create SettingsPage component in src/pages/
2. Add route in src/router.ts
3. Create settings API endpoints
4. Add settings to user model
5. Write tests
## Dependencies: User model (exists), Router (exists)
## Estimated complexity: Medium
```

Plans are auditable, reviewable, and can be evaluated before execution begins.
