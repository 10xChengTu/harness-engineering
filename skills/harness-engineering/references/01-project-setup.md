# 01 - Project Setup

How to set up a project's foundational harness files so agents can work effectively.

## AGENTS.md (The Entry Point)

AGENTS.md is a **table of contents**, not an encyclopedia. It tells the agent where to find things, not everything it needs to know.

### Structure

```markdown
# Project Name

## Quick Start
<1-3 commands to get running>

## Architecture Overview
<2-3 sentences + pointer to docs/architecture.md>

## Directory Structure
<tree of key directories with 1-line descriptions>

## Key Conventions
<5-10 rules the agent MUST follow — only the most important ones>

## Documentation Map
<pointers to docs/ files by topic>

## Common Tasks
<task → relevant files/commands mapping>
```

### Anti-patterns
- Dumping entire codebase knowledge into AGENTS.md (too long, agent skims)
- No AGENTS.md at all (agent guesses everything)
- Stale AGENTS.md that contradicts reality (worse than no file)

## docs/ Directory (System of Record)

The `docs/` directory is where detailed knowledge lives. AGENTS.md points here.

### Recommended Structure

```
docs/
├── architecture.md      # System design, component relationships
├── conventions.md       # Coding standards, naming, patterns
├── api.md              # API contracts, endpoints
├── data-model.md       # Database schema, data flow
├── testing.md          # Test strategy, how to run, what to test
├── deployment.md       # Build, deploy, environments
└── decisions/          # Architecture Decision Records (ADRs)
    └── 001-chose-x.md
```

### Writing Docs for Agents

- **Be explicit about "why"** — agents follow rules better when they understand the reasoning
- **Include examples** — show the pattern, not just describe it
- **Keep files focused** — one topic per file, <300 lines ideal
- **Add a TOC** for files >100 lines
- **Date and version** decisions — agents need to know what's current

## Design Notes in Source Tree

Embed context where agents will encounter it — in the source tree itself.

```
src/
├── components/
│   ├── DESIGN_NOTES.md    # Why components are structured this way
│   └── Button/
├── api/
│   ├── DESIGN_NOTES.md    # API design principles
│   └── routes/
```

These files survive across sessions. They're the cross-session memory that prevents agents from re-making decisions or contradicting past choices.

## init.sh Pattern (For Automated Setups)

For projects that need environment setup before agents can work:

```bash
#!/bin/bash
# init.sh - Run before agent starts working
set -e

# Install dependencies
npm install  # or pip install, cargo build, etc.

# Set up local config
cp .env.example .env.local

# Verify setup
npm run typecheck
npm test -- --run

echo "Environment ready for agent work"
```

## Initial Commit Convention

After setting up harness files, make an initial commit:
```
git add -A && git commit -m "harness: initial project setup"
```

This gives agents a clean baseline to diff against and revert to if needed.

## Quality Scoring (Optional)

For larger docs/, add quality metadata:

```markdown
---
quality: 0.8
last_verified: 2025-01-15
owner: @username
---
```

This helps agents (and humans) know which docs to trust and which need updating.
