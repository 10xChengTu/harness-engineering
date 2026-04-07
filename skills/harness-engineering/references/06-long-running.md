# 06 - Long-Running Tasks

Patterns for agents working on tasks that span hours, multiple sessions, or exceed context windows.

## The Two Core Problems

1. **Context degradation**: As context fills, agent loses coherence and may "rush to finish"
2. **State loss**: Between sessions, everything not on disk is forgotten

## Initializer + Worker Pattern

Split long tasks into setup and execution:

### Initializer Agent
Runs once at the start. Creates:
- `init.sh` — environment setup script
- `progress.md` — tracks what's done and what's next
- `features.json` — structured feature list with status
- Initial git commit — clean baseline

### Worker Agent
Runs iteratively. For each cycle:
1. Read progress file
2. Pick next incomplete feature
3. Implement it
4. Run tests
5. Git commit
6. Update progress file
7. Repeat or handoff

## Progress Tracking

### File-Based Progress (Critical)
```markdown
# progress.md
Updated: 2025-01-15T14:30:00Z

## Completed Features
- [x] User authentication (commit: abc123)
- [x] Database schema (commit: def456)

## In Progress
- [ ] Dashboard UI — layout done, charts pending

## Remaining
- [ ] Settings page
- [ ] Export functionality

## Known Issues
- Auth token refresh has edge case with expired sessions
- Dashboard chart library version conflict (pinned to 2.x for now)

## Decisions Made
- Using SQLite for MVP, will migrate to PostgreSQL later (see docs/decisions/001.md)
```

**Update after every commit.** This is the cross-session memory.

### Structured Feature Tracking
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

### Context Reset (Full Swap)
- Kill current agent, start fresh agent
- Pass handoff artifact with full state
- Pros: Clean context, no "anxiety", fresh reasoning
- Cons: Higher latency, must encode all state in artifact

### Compaction (In-Place Summary)
- Summarize early conversation, continue in same session
- Pros: Preserves continuity, lower latency
- Cons: May not fully reset "context anxiety", residual confusion

### Decision Guide
- Model shows degraded performance with long context → Context Reset
- Model handles long context well → Compaction is fine
- Task requires fresh perspective → Context Reset
- Task benefits from continuity → Compaction

## Handoff Artifacts

When one agent passes work to another, the handoff must contain enough state for a new agent to continue **without reading the full conversation history**.

### Structured Session Handoff Template

Use this template at the end of every session:

```markdown
# Session Handoff

## Verified Now
- What is currently working: <list with evidence — test output, screenshot, commit hash>
- What verification actually ran: <exact commands and their results>

## Changed This Session
- Code or behavior added: <list of functional changes with file paths>
- Infrastructure or harness changes: <config, deps, build changes>

## Broken Or Unverified
- Known defect: <specific bugs with reproduction steps>
- Unverified path: <code paths that exist but haven't been tested>
- Risk for the next session: <things likely to break or cause confusion>

## Next Best Step
- Highest-priority unfinished feature: <one specific item>
- Why it is next: <reasoning — dependency order, user priority, risk>
- What counts as passing: <concrete success criteria>
- What must not change during that step: <protected invariants>

## Commands
- Startup: <exact commands to get the dev environment running>
- Verification: <exact commands to confirm everything works>
- Focused debug command: <most useful command if something breaks>
```

**Key difference from a generic handoff**: This template separates _verified_ from _unverified_ state, and defines _what counts as passing_ for the next step. A new agent doesn't just know what to do — it knows how to prove it's done.

### Legacy Format (Simpler Alternative)

For simpler projects, a lighter format works:

```markdown
# handoff.md

## What Was Done
<list of completed items with commit references>

## Current State
<what the codebase looks like now, running services, env state>

## What To Do Next
<ordered list of remaining tasks>

## Critical Context
<decisions, constraints, gotchas the next agent MUST know>

## Files Modified
<list of changed files and what changed in each>
```

Use the structured template for long-running multi-session projects. Use the legacy format for simple one-off handoffs.

## Git as Checkpoint System

```bash
# Commit after every feature/logical unit
git add -A && git commit -m "feat(auth): implement login flow"

# Tag milestones
git tag -a sprint-1-complete -m "Sprint 1: Auth + DB schema"
```

If something goes wrong, agent (or human) can revert to last known good state.

## Incremental Verification

Don't wait until the end to test. After each feature:

```bash
# Verify as you go
npm run typecheck      # Types still correct?
npm run test           # Tests still pass?
npm run dev            # App still runs?
```

Catching errors early prevents compound failures that are hard to debug.

## Session Lifecycle

A complete long-running session follows four phases. Each phase has specific actions and outputs.

### Phase 1: START
1. `pwd` — confirm working directory
2. Read `AGENTS.md` — load project conventions and entry points
3. Read `progress.md` — understand current state (what's done, what's next)
4. Read `features.json` / feature list — understand full scope
5. Run `git log --oneline -10` — see recent changes and commits

### Phase 2: SELECT
6. Pick ONE feature from the remaining list
7. Verify preconditions (dependencies completed, blockers resolved)
8. Announce the feature and scope to the user (or log it)

### Phase 3: EXECUTE
9. Implement the feature
10. Run verification after each logical unit (typecheck, test, dev server)
11. Git commit with descriptive message after each verified unit
12. Update `progress.md` — mark completed items, note decisions

### Phase 4: WRAP UP
13. Run full verification suite (`typecheck + lint + test + dev`)
14. Fill out session handoff template (see above)
15. Git commit the updated progress and handoff artifacts
16. Assess: continue to next feature, or context reset needed?

### Decision: Continue vs Reset
- Context window < 60% used, agent reasoning still sharp → Continue to Phase 2
- Context window > 60%, or agent showing signs of degradation → Context reset with handoff

## Anti-Patterns

- **Premature completion**: Agent declares "done" when it's not. Fix: explicit feature checklist + verification step.
- **Scope drift**: Agent adds unrequested features. Fix: structured feature list, agent checks against it.
- **Undocumented state**: Agent makes changes without recording them. Fix: mandatory progress updates.
- **Big bang testing**: Testing only at the end. Fix: test after each feature.
