# 02 - Context Engineering

What agents see determines what they do. Context engineering = designing the information environment for optimal agent performance.

## Core Formula

**Prompt Engineering** = what you say to the agent
**Context Engineering** = what you show the agent
**Harness Engineering** = the whole system (context + constraints + feedback + architecture)

## Progressive Disclosure

Don't dump everything into the system prompt. Layer information:

1. **Always visible** (~100 words): Project name, tech stack, critical rules
2. **On-demand** (AGENTS.md → docs/): Architecture, conventions, patterns
3. **Just-in-time** (Design Notes, inline comments): File-specific context

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

### Handoff Artifact Structure

When resetting context, the handoff must contain:
1. What was accomplished
2. Current state of all files changed
3. What to do next
4. Any decisions made and why
