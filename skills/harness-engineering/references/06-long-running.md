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

**Bootstrap Contract**: The initializer is done when a new agent can:
1. **Start** — environment runs with one command (`init.sh`)
2. **Test** — test suite passes on the baseline
3. **See progress** — progress file shows what's done and what's next
4. **Pick up** — next steps are unambiguous enough to begin immediately

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

When one agent passes work to another:

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

**The handoff must contain enough state for a new agent to continue without reading the full conversation history.**

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

## Anti-Patterns

- **Premature completion**: Agent declares "done" when it's not. Fix: explicit feature checklist + verification step.
- **Scope drift**: Agent adds unrequested features. Fix: structured feature list, agent checks against it.
- **WIP > 1**: Agent works on multiple features simultaneously, finishing none completely. Fix: enforce **WIP=1** — complete and verify one feature before starting the next. Overreach and under-finish are co-occurring problems.
- **Undocumented state**: Agent makes changes without recording them. Fix: mandatory progress updates.
- **Big bang testing**: Testing only at the end. Fix: test after each feature.

## Session Exit Checklist

Before ending a session, verify all five dimensions are clean:

1. **Build**: Project compiles / passes typecheck
2. **Tests**: All tests pass (no new failures)
3. **Progress**: Progress file updated with current state
4. **Artifacts**: No uncommitted changes, no temp files left behind
5. **Startup**: A new agent can resume using only on-disk artifacts (bootstrap contract holds)

**Session exit is a completion requirement, not a courtesy.** An unclean exit transfers debugging cost to the next session.
