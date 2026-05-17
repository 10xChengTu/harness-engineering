# Agent Brain Quickstart

This quickstart turns a repository from "one large prompt file" into a small agent brain that is easier for coding agents to load, audit, and improve.

## Pattern

Use a thin harness and fat skills:

1. Keep `AGENTS.md` short.
2. Put durable project knowledge under `docs/` or `brain/`.
3. Put task-specific procedures in skills or workflow files.
4. Add a resolver that tells the agent which file to read for each task.
5. Verify changes with a checklist before merging.

## Minimal layout

```text
AGENTS.md
brain/
  RESOLVER.md
  architecture.md
  decisions.md
  pitfalls.md
  workflows/
    debug.md
    ship.md
```

## Resolver example

```markdown
# Agent Brain Resolver

Before working, classify the task:

- Bug or failing test -> read `brain/workflows/debug.md`.
- Release or PR -> read `brain/workflows/ship.md`.
- Architecture change -> read `brain/architecture.md` and `brain/decisions.md`.
- Known fragile area -> read `brain/pitfalls.md`.

Only load files that match the task. Do not load the whole brain by default.
```

## Verification checklist

- [ ] Top-level agent instructions stay short.
- [ ] Repeated procedures live in workflow files.
- [ ] Long-term facts live in brain files, not chat memory.
- [ ] The resolver points to exact files.
- [ ] The agent can complete a task without reading unrelated context.
