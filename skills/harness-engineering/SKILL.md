---
name: harness-engineering
description: "Set up and improve harness engineering (AGENTS.md, docs/, lint rules, eval systems, project-level prompt engineering) for AI-agent-friendly codebases. Triggers on: new/empty project setup for AI agents, AGENTS.md or CLAUDE.md creation, harness engineering questions, making agents work better on a codebase. ALSO triggers when users are frustrated or complaining about agent quality — e.g. 'the agent keeps ignoring conventions', 'it never follows instructions', 'why does it keep doing X', 'the agent is broken' — because poor agent output almost always signals harness gaps, not model problems. Covers: context engineering, architectural constraints, multi-agent coordination, evaluation, long-running agent harness, and diagnosis of agent quality issues."
---

# Harness Engineering

Harness = the operating system for AI agents working on your project. Model is CPU, context window is RAM, harness is OS.

## Core Principle

**Start simple, add complexity only when needed.** Every harness component encodes an assumption about what the model can't do alone. Pressure-test these assumptions — they expire as models improve. Build for deletion.

## When This Skill Activates

| Signal | Action |
|--------|--------|
| Empty/new project | → Full project setup (Section 1) |
| User frustrated with agent | → Diagnose & fix harness gaps (Section 7) |
| Existing project needs improvement | → Assess & incrementally improve |
| Explicit harness question | → Reference relevant sections |

## Workflow

### For New Projects

1. **Assess** — What's the project? Tech stack? Team size? How will agents be used?
2. **Setup** — Create foundational harness files → read `references/01-project-setup.md`
3. **Context** — Design information architecture → read `references/02-context-engineering.md`
4. **Constraints** — Add guardrails and linters → read `references/03-constraints.md`
5. **Evaluate** — Set up feedback loops → read `references/05-eval-feedback.md`
6. If project involves multi-agent or long tasks → read `references/04-multi-agent.md`, `references/06-long-running.md`

### For Diagnosis (Agent Not Performing Well)

1. Read `references/07-diagnosis.md` immediately
2. Identify which harness layer is failing
3. Apply targeted fix from the relevant reference

### For Incremental Improvement

Assess current harness maturity, identify weakest layer, improve one layer at a time.

## Harness Layers (Quick Reference)

| Layer | What | Reference |
|-------|------|-----------|
| **Project Setup** | AGENTS.md, docs/, directory conventions | `01-project-setup.md` |
| **Context Engineering** | What info agents see, progressive disclosure, working state | `02-context-engineering.md` |
| **Constraints & Guardrails** | Linters, type systems, architecture enforcement, safe autonomy | `03-constraints.md` |
| **Multi-Agent Architecture** | Agent separation, coordination protocols, delegation patterns | `04-multi-agent.md` |
| **Eval & Feedback** | Testing, grading, GC agents, observability | `05-eval-feedback.md` |
| **Long-Running Tasks** | Progress tracking, context resets, handoff artifacts | `06-long-running.md` |
| **Diagnosis** | When agents fail — identify root cause in harness, not model | `07-diagnosis.md` |

## Self-Update Protocol

When you discover a new reusable harness pattern during a project:

1. Identify which reference file it belongs to (or if it needs a new one)
2. Add the pattern with: **what** it solves, **when** to use it, **how** to implement it
3. Keep it concise — no fluff, just the pattern
