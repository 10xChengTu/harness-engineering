<div align="right"><strong>English</strong> | <a href="README.zh-CN.md">中文</a></div>

# harness-engineering

An [agent skill](https://skills.sh) for setting up and improving **harness engineering** — the infrastructure that makes AI agents work effectively on your codebase.

> **Harness = the operating system for AI agents.** Model is CPU, context window is RAM, harness is OS.

## Install

```bash
# English version
npx skills add 10xChengTu/harness-engineering/skills/harness-engineering

# 中文版
npx skills add 10xChengTu/harness-engineering/skills/harness-engineering-zh
```

## What It Does

This skill teaches your AI agent how to build and maintain the harness layer for any project — the `AGENTS.md`, `docs/`, lint rules, constraints, and evaluation systems that determine whether agents produce good or bad output.

**Core principle:** Start simple, add complexity only when needed. Every harness component encodes an assumption about what the model can't do alone.

### Trigger Scenarios

| You say... | The skill does... |
|---|---|
| "Set up this project for AI agents" | Full project harness setup |
| "Create an AGENTS.md" | Scaffolds entry point + docs structure |
| "The agent keeps ignoring conventions" | Diagnoses harness gaps, not model problems |
| "Why does it keep doing X wrong?" | Identifies root cause in harness layer |
| "Make agents work better on this codebase" | Assesses & incrementally improves harness |

## What's Covered

The skill includes 9 reference modules that the agent consults as needed:

| Module | What It Covers |
|---|---|
| **Project Setup** | `AGENTS.md` structure, `docs/` directory, design notes, init scripts |
| **Context Engineering** | What agents see, progressive disclosure, working state management |
| **Constraints & Guardrails** | Linters, type systems, architecture enforcement, safe autonomy |
| **Multi-Agent Architecture** | Agent separation, coordination protocols, delegation patterns |
| **Eval & Feedback** | Testing agent output, grading, observability, feedback loops |
| **Long-Running Tasks** | Progress tracking, context resets, handoff artifacts |
| **Loops** | Verifiers, stop conditions, guardrails, fresh-context iteration, loop stack |
| **Memory** | Memory lifecycle, files vs databases, index-then-fetch, consolidation, forgetting |
| **Diagnosis** | When agents underperform — symptom → root cause mapping |

## Bilingual Skills / 双语 Skill

This project provides two installable skills with identical content in different languages:

```bash
# English
npx skills add 10xChengTu/harness-engineering/skills/harness-engineering

# 中文
npx skills add 10xChengTu/harness-engineering/skills/harness-engineering-zh
```

## Why This Exists

Poor agent output is almost always a harness problem, not a model problem. When your agent ignores conventions, makes wrong assumptions, or produces inconsistent results — the fix is better context, constraints, and feedback loops, not a bigger model.

This skill encodes the patterns and anti-patterns learned from real-world agent deployments so you don't have to rediscover them.

## Compatibility

Works with any agent that supports the [Agent Skills specification](https://agentskills.io), including Claude Code, OpenCode, Cursor, Codex, Cline, GitHub Copilot, and [40+ more](https://github.com/vercel-labs/skills#supported-agents).
