# 07 - Diagnosis: When Agents Underperform

When the user is frustrated with agent output, the problem is almost always in the harness, not the model. This guide helps identify and fix the root cause.

## Symptom → Root Cause Map

| User Complaint | Likely Harness Gap | Fix |
|---|---|---|
| "It keeps making the same mistake" | No constraint preventing it | Add lint rule / type check / test |
| "It doesn't follow our conventions" | Conventions not documented or not discoverable | Write conventions in docs/, reference from AGENTS.md |
| "It broke something that was working" | No regression tests | Add tests for existing behavior before changing |
| "It goes off on tangents" | No clear task scope or feature list | Add structured feature list / execution plan |
| "It writes mediocre code" | No examples of good code in context | Add code examples / patterns in DESIGN_NOTES.md |
| "It forgets what we discussed" | Cross-session context not persisted | Write decisions to files, use progress.md |
| "It declares done too early" | No verification step | Add checklist, tests, evaluator agent |
| "It uses wrong patterns" | Competing patterns in codebase, no guidance | Document which pattern to use when |
| "Output quality is inconsistent" | No evaluation/feedback loop | Add eval system, GC agent |
| "It takes forever and costs too much" | Over-engineered harness or wrong architecture | Simplify — remove harness components that don't add value |

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
