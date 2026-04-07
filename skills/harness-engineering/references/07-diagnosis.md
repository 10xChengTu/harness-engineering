# 07 - Diagnosis: When Agents Underperform

When the user is frustrated with agent output, the problem is almost always in the harness, not the model. This guide helps identify and fix the root cause.

## Symptom → Root Cause Map

| User Complaint | What It Looks Like | Likely Harness Gap | Fix | Supporting Artifact |
|---|---|---|---|---|
| "It keeps making the same mistake" | Same error pattern across sessions despite corrections | No constraint preventing it | Add lint rule / type check / test | Custom lint rule or test file |
| "It doesn't follow our conventions" | Wrong naming, wrong patterns, wrong file locations | Conventions not documented or not discoverable | Write conventions in `docs/`, reference from `AGENTS.md` | `DESIGN_NOTES.md` |
| "It broke something that was working" | Passing tests start failing after agent changes | No regression tests | Add tests for existing behavior before changing | Test suite |
| "It goes off on tangents" | Agent starts several features, finishes none cleanly | No clear task scope or feature list | Add structured feature list / execution plan | `features.json` |
| "It writes mediocre code" | Generic, template-like output lacking project personality | No examples of good code in context | Add code examples / patterns in `DESIGN_NOTES.md` | `DESIGN_NOTES.md` with examples |
| "It forgets what we discussed" | New session spends most time rediscovering setup and status | Cross-session context not persisted | Write decisions to files, use `progress.md` | `progress.md` |
| "It declares done too early" | Agent claims done after code edits but before runnable proof | No verification step | Add checklist, tests, evaluator agent | Clean-state checklist |
| "It uses wrong patterns" | Competing patterns used inconsistently across codebase | Competing patterns, no guidance on which to use | Document which pattern to use when | `DESIGN_NOTES.md` with pattern guide |
| "Output quality is inconsistent" | Review quality depends on reviewer's taste or memory | No evaluation / feedback loop | Add eval system, scoring rubric, GC agent | Evaluator rubric |
| "It takes forever and costs too much" | Agent spends more time on compliance than actual work | Over-engineered harness or wrong architecture | Simplify — remove harness components that don't add value | Simplification log |

## Diagnosis Process

### Step 1: Identify the Layer

Ask: Where in the harness stack is the failure?

1. **Context**: Agent doesn't have the right information
2. **Constraints**: Agent isn't prevented from making errors
3. **Feedback**: Agent doesn't know it's failing
4. **Architecture**: Single-agent can't handle the task's complexity
5. **Scope**: Task is too big or ambiguous

### Step 2: Minimal Fix

Apply the smallest change that addresses the root cause:

- Missing context → Add one doc file or DESIGN_NOTES.md
- Missing constraint → Add one lint rule or test
- Missing feedback → Add one verification step
- Architecture problem → Split into two agents (but only if single agent truly can't handle it)
- Scope problem → Break task into smaller pieces

**Don't over-engineer the fix.** One rule per mistake. Iterate.

### Step 3: Verify

After applying the fix:
1. Reproduce the original problem scenario
2. Confirm the fix prevents it
3. Confirm the fix doesn't break other things

## Common Harness Improvements (Ordered by Impact)

### High Impact, Low Effort
1. **Add AGENTS.md** if missing — immediate orientation improvement
2. **Add one lint rule** for recurring mistake — prevents whole category of errors
3. **Add test for broken behavior** — catches regression instantly
4. **Document the convention** agent keeps violating — eliminates guessing

### High Impact, Medium Effort
5. **Create docs/ with architecture overview** — reduces architectural mistakes
6. **Add pre-commit hooks** — catches issues before they compound
7. **Set up progress tracking** — prevents premature completion and scope drift
8. **Add DESIGN_NOTES.md** in key directories — provides just-in-time context

### High Impact, High Effort
9. **Implement evaluator agent** — for quality-critical or subjective tasks
10. **Build custom linters** — for project-specific architectural constraints
11. **Create eval suite** — systematic quality measurement and regression detection
12. **Set up GC agent** — periodic consistency checking

## The "One Rule Per Mistake" Discipline

Every time an agent makes a mistake:

1. Fix the immediate issue
2. Ask: "Could a rule prevent this forever?"
3. If yes → add the rule (lint, test, type, or documented convention)
4. If no → add context (docs, examples, DESIGN_NOTES.md)

Over time, the harness accumulates rules that prevent every mistake the agent has ever made. The error rate converges toward zero for known failure modes.

## When to Simplify

Signs the harness is over-engineered:
- Agent spends more time on harness compliance than actual work
- Multiple redundant checks for the same thing
- Harness rules that never trigger (the model learned past them)
- Cost/time significantly higher without proportional quality gain

**Remove harness components when the model no longer needs them.** Models improve — yesterday's scaffolding is today's dead weight.

## Harness as Dataset

Every agent interaction is a training signal. The harness captures traces:
- What the agent tried
- What worked
- What failed
- What the fix was

These traces are your competitive advantage. They're the data that makes your harness better over time.
