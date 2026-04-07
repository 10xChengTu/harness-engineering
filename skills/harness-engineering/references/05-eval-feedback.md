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

## Evaluator Rubric (Post-Implementation Review)

Before accepting agent output, score it against a fixed rubric. This converts "does it look right?" into a structured, repeatable judgment.

### Standard 6-Category Rubric

| Category | Question | Score (0-2) | Notes |
|---|---|---|---|
| **Correctness** | Does the implemented behavior match the requested feature? | | |
| **Verification** | Did the required checks actually run, with evidence? | | |
| **Scope discipline** | Did the session stay inside the chosen feature scope? | | |
| **Reliability** | Does the result survive restart or rerun without repair? | | |
| **Maintainability** | Is the code and documentation clear enough for the next session? | | |
| **Handoff readiness** | Can a fresh session continue work from repo artifacts only? | | |

### Scoring

- **2**: Fully met, with evidence
- **1**: Partially met or met without evidence
- **0**: Not met or not attempted

### Verdict

Based on the total score:
- **Accept** (10-12): Output meets all criteria. Merge/ship.
- **Revise** (6-9): Fixable gaps. Agent continues in same session.
- **Block** (0-5): Fundamental problems. Restart with corrected harness.

### Required Follow-Up (on Revise/Block)

```markdown
- Missing evidence: <what verification is needed>
- Required fixes: <specific items to address>
- Next review trigger: <when to re-evaluate>
```

Use this rubric consistently across sessions. It surfaces harness gaps — if agents repeatedly score low on the same category, that category needs better harness support (constraints, context, or feedback loops).

## Quality Score Tracking

For projects with ongoing agent work, maintain a living quality score document. This tracks whether the repository is getting stronger or weaker over time.

### Grading Scale

- **A**: Verified, legible, stable, boundaries enforced
- **B**: Working with minor gaps
- **C**: Partially working, notable confusion or instability
- **D**: Broken, unsafe, or structurally unclear

### Product Domain Grades

Track quality per domain area of the product:

| Domain | Grade | Verification | Agent Legibility | Test Stability | Key Gaps | Last Updated |
|---|---|---|---|---|---|---|
| `auth` | B | Tests pass | Clear DESIGN_NOTES | 2 flaky | Token refresh edge case | 2025-01-15 |
| `dashboard` | C | Manual only | No docs | None | No automated tests | 2025-01-15 |

### Architectural Layer Grades

Track quality per technical layer:

| Layer | Grade | Boundary Enforcement | Agent Legibility | Key Gaps | Last Updated |
|---|---|---|---|---|---|
| Types | A | Strict mode, no `any` | Self-documenting | — | 2025-01-15 |
| Services | B | Lint rules | DESIGN_NOTES exists | Missing error handling docs | 2025-01-15 |
| UI | C | None | No component docs | Need component catalog | 2025-01-15 |

### Benchmark Snapshots

Record eval results over time to measure harness effectiveness:

| Date | Harness Variant | Completion Rate | Retries | Defects Before Review | Notes |
|---|---|---|---|---|---|
| 2025-01-10 | baseline | 60% | 3.2 | 5 | Before adding DESIGN_NOTES |
| 2025-01-15 | + DESIGN_NOTES | 80% | 1.5 | 2 | Significant improvement |

### Simplification Log

Track when harness components are removed and whether quality degraded:

| Date | Component Removed | Outcome | Decision |
|---|---|---|---|
| 2025-01-20 | Verbose auth comments | Unchanged | Keep removed |
| 2025-01-25 | Pre-commit type check | Degraded | Restore |

This log enforces the "remove what the model no longer needs" discipline. If removal doesn't degrade quality, the component was dead weight.

## Method Map (Failure Mode → Fix)

When agent work fails, use this map to quickly identify the right fix. Each failure mode has a primary fix and a supporting artifact.

| Failure Mode | What It Looks Like | Primary Fix | Supporting Artifact |
|---|---|---|---|
| **Cold-start confusion** | New session spends most time rediscovering setup and status | Make the repository the system of record | `progress.md` |
| **Scope sprawl** | Agent starts several features, finishes none cleanly | Restrict active scope to one feature | `features.json` / structured feature list |
| **Premature completion** | Agent claims done after code edits but before runnable proof | Bind completion to evidence (tests pass, app runs) | Clean-state checklist |
| **Fragile startup** | Every session re-learns how to boot the project | Standardize setup and verification | `init.sh` |
| **Weak handoff** | Next session can't tell what's verified, broken, or next | End with explicit structured handoff | Session handoff template |
| **Subjective review** | Review quality depends on taste or memory | Score output with fixed rubric categories | Evaluator rubric (see above) |

### Operating Principle

Add the **smallest artifact** that directly addresses the observed failure mode. Avoid solving every reliability problem by dumping more text into one global instruction file — each problem gets its own targeted fix.
