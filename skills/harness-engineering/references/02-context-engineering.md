# 02 - Context Engineering

What agents see determines what they do. Context engineering = designing the information environment for optimal agent performance.

## Core Formula

**Prompt Engineering** = what you say to the agent
**Context Engineering** = what you show the agent
**Harness Engineering** = the whole system (context + constraints + feedback + architecture)

## The Four Context Operations

Every token in the context window should earn its place through one of four operations:

1. **SELECT** — Load context just-in-time, not all-at-once. Pull in docs, files, and state only when the agent's current task demands it.
2. **WRITE** — Agent writes back to persistent storage (progress logs, decisions, discovered rules). If it's not written to a file, it's lost.
3. **COMPRESS** — Reactive compaction of older turns mid-session. When context usage exceeds ~80%, summarize older turns while preserving recent context.
4. **ISOLATE** — Delegated work must not pollute parent context. Sub-agents start with scoped context, not the full conversation.

These operations are how you manage context as a **budget**, not a dump.

## Progressive Disclosure

Don't dump everything into the system prompt. Layer information in three tiers:

1. **Tier 1 — Metadata (always present, cheap)**: ~100 words — project name, tech stack, feature list status, memory index, critical rules
2. **Tier 2 — Instructions (loaded on activation)**: AGENTS.md → docs/ — architecture, conventions, style guides
3. **Tier 3 — Resources (loaded on demand)**: DESIGN_NOTES.md, API references, examples — file-specific context pulled just-in-time

### Implementation

```
AGENTS.md (always loaded)
  → "For architecture details, see docs/architecture.md"
  → "For API conventions, see docs/api.md"

docs/architecture.md (loaded when agent works on architecture)
  → "For the auth subsystem specifically, see src/auth/DESIGN_NOTES.md"
```

**AGENTS.md as router**: It doesn't contain knowledge — it routes to knowledge. Like a table of contents, not the book.

## Working State Management

Agents lose state between sessions. Design explicit state persistence:

### Progress File Pattern

```markdown
# progress.md (or claude-progress.txt)

## Current State
- Feature X: 80% complete, auth flow done, UI pending
- Feature Y: Not started

## Completed
- [x] Database schema migration
- [x] API endpoints for /users

## Blocked
- Need API key for external service (asked user 2025-01-15)

## Next Steps
1. Complete Feature X UI components
2. Write integration tests for auth flow
```

### Feature List JSON Pattern

```json
{
  "features": [
    {
      "name": "User Authentication",
      "status": "complete",
      "files": ["src/auth/*", "src/middleware/auth.ts"],
      "tests": "passing"
    },
    {
      "name": "Dashboard",
      "status": "in_progress",
      "files": ["src/pages/dashboard/*"],
      "tests": "not_written"
    }
  ]
}
```

## Give Maps, Not Manuals

Instead of step-by-step instructions, give agents orientation:

**Bad** (manual):
```
Step 1: Open src/auth/login.ts
Step 2: Find the handleLogin function
Step 3: Add rate limiting by...
```

**Good** (map):
```
Auth system lives in src/auth/. Login flow: login.ts → validate.ts → session.ts.
Rate limiting middleware is in src/middleware/rateLimit.ts — follow its pattern.
Tests in src/auth/__tests__/ — every auth change needs a test.
```

Maps let agents navigate autonomously. Manuals make them fragile to any deviation.

## Cross-Session Context

What survives between agent sessions:
- Files on disk (AGENTS.md, docs/, DESIGN_NOTES.md, progress files)
- Git history (commit messages, diffs)
- Code comments and docstrings

What doesn't survive:
- Conversation history
- Agent's internal reasoning
- Verbal agreements ("we decided to use X approach")

**Rule**: If a decision matters, it must be written to a file. Verbal context is lost context.

## Context Window as RAM

- Context window fills up → agent loses coherence
- **Compaction** (summarize old context): Maintains continuity but doesn't reset "context anxiety"
- **Context reset** (fresh agent + handoff artifact): Clean start, requires good handoff docs
- Choice depends on model: Some models handle long contexts well, others degrade

### Compress Pattern (Mid-Session Compaction)

When a long session exhausts the context window:

1. **Trigger**: Context usage exceeds threshold (e.g., 80%)
2. **Summarize**: Older turns (first 50% by token count) into a structured snapshot
3. **Preserve**: Recent context (last 20% of turns) untouched
4. **Label**: Mark snapshot as "compacted at turn N"

```markdown
## Session Summary (Turns 1-15, compacted)

**Goal**: Implement Q&A feature with citations
**Decisions made**:
- Use streaming response for UX
- Citation format: [doc:chunk] inline references
**Key files created**:
- src/services/QaService.ts
- src/shared/types.ts (extended with QaResult)
```

### Context Budget Template

Track context usage explicitly to prevent silent degradation:

```markdown
## Context Budget (Session)

| Category | Budget | Current | Status |
|----------|--------|---------|--------|
| System prompt | 2,000 | 1,850 | ✓ |
| Instruction files | 3,000 | 2,400 | ✓ |
| Session history | 10,000 | 4,200 | ✓ |
| Working context | 15,000 | 3,100 | ✓ |
| **Total** | **30,000** | **11,550** | 38% used |

**Compaction trigger**: 80% (24,000 tokens)
```

### Handoff Artifact Structure

When resetting context, the handoff must contain:
1. What was accomplished
2. Current state of all files changed
3. What to do next
4. Any decisions made and why

## Encode Unseen Knowledge

A critical context engineering task: make agent-invisible knowledge discoverable in the codebase.

**Trigger signals** — you need this when:
- The agent keeps asking how the system works
- Humans say "we decided this in Slack" or "follow what X said last week"
- Reviews reference product or security rules that aren't written in-repo
- New sessions repeat discovery work that should already be settled

**Process:**
1. List invisible knowledge sources: docs, chats, tacit team rules, verbal decisions
2. For each source, classify it: architecture? product behavior? security policy? reliability expectation?
3. Encode it into the matching repo artifact:
   - Architecture → `ARCHITECTURE.md` or `docs/architecture.md`
   - Product behavior → `docs/product-specs/`
   - Design rationale → `docs/decisions/` (ADRs)
   - Execution state → progress files
   - Quality expectations → `docs/QUALITY_SCORE.md`
4. Replace vague statements with operationally useful wording
5. Remove or deprecate stale copies so the repo keeps one discoverable truth

**Encoding rules:**
- Write for discoverability, not for literary completeness
- Prefer short documents with clear filenames
- Link related artifacts together
- Store durable rules, not meeting transcripts
- Update the repo in the same session that the decision is made

**Success test:** A fresh agent can discover the relevant rule without asking a human.
