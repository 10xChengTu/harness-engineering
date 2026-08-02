# 08 - Loops: Autonomous Operation

Patterns for turning a harness into a system that runs itself — loops that prompt agents, verify their work, and know when to stop.

## The Fourth Handoff

Prompt engineering = what you say. Context engineering = what you show. Harness engineering = the environment. **Loop engineering = how the system runs itself.**

Each layer wraps the previous one — it doesn't replace it. There's still a prompt inside a loop — it just stops being something typed live and becomes a template the loop reassembles every iteration.

The human's position moves. **Sit on the loop, not in it.** Two jobs remain: design the loop's stages and its STOP condition, and review what comes out after it stops. Everything in between — discovering the work, writing the prompt, judging the output, deciding whether to go again — used to be typed by hand, one round at a time. Now it's assembled by the loop itself.

## Anatomy of a Loop

Simplest form: gather context → take action → verify work → repeat.

Full version has six stages:

```
DISCOVER  → find work (failing CI, new issues, a TODO list)
ASSEMBLE  → build this iteration's prompt + context
ACT       → execute
VERIFY    → run the verifier
PERSIST   → commit, update state files
DECIDE    → loop again, or STOP
```

DISCOVER and ASSEMBLE are the stages that used to be a human typing a prompt by hand. VERIFY and DECIDE are the stages that used to be a human reading output and deciding what to say next. Everything else — the agent doing the work — hasn't changed.

## Routine vs Workflow vs Loop

Not everything that runs unattended is a loop.

| | Routine | Workflow | Loop |
|---|---|---|---|
| Steps | Fixed | Branch on discovery | Iterate dynamically |
| Stop condition | Steps finished | Path finished | Verifier confirms goal reached |
| Self-check | None | Weak or none | Core mechanism |
| Example | Scheduled lint run + report | CI failure → triage → assign | Fix until `test/auth` is all green |

The only test that matters: **does it check its own work, and decide whether to continue based on that check?** A nightly cron job that runs lint and emails a report is a routine — it never asks "did that fix anything?" A loop that reruns lint, checks the diff against a target, and keeps going until the target is met is a different kind of system, even if the code driving it looks similar.

## The Verifier Is the Load-Bearing Wall

**A loop's value ceiling is set by its verifier, not its model.** Without a reliable answer to "is this done?", a loop either never stops, or stops in the wrong place and declares success with total confidence.

Build order: verifier before loop. Stand up something a machine can run — a test suite, a type checker, a diff scope check — before wrapping a loop around it. A stronger model pointed at a weak verifier just produces confident wrong answers faster.

| | Bad stop condition | Good stop condition |
|---|---|---|
| Example | "improve this code" | "all tests under `test/auth` pass and lint is clean" |
| Checkable? | Only by feel | A machine can check it |
| Loop behavior | Never knows when to stop | Has a clean exit |

**The checker is never the same agent as the maker.** This applies to the stop condition itself, too — an independent evaluator (usually a smaller, faster model) decides "done," not the model that wrote the code. See `04-multi-agent.md` (Generator-Evaluator).

The model that wrote the code is a poor judge of the code it wrote — self-evaluation bias runs in one direction, toward leniency. A second, independent pass is what catches the problems the first agent already talked itself into accepting.

## Four Failure Modes

A loop that runs unattended fails differently than an agent watched turn by turn — the failure has room to run before anyone notices.

| Failure mode | Example | Countermeasure |
|---|---|---|
| Reward hacking | Deleting failing tests to turn CI green | Verifier also checks that test count didn't drop; state the rule explicitly: removing or editing tests to pass is unacceptable |
| Hallucinated success | Agent reports "done" when it isn't | "Done" is a claim, not proof — trust the deterministic verifier, never a self-report |
| Compounding error | A small mistake in iteration 3 becomes a crater by iteration 15 | Small commits per iteration + independent verification per iteration surface errors early |
| Cost explosion | Loop runs unattended overnight, bill hits four figures | Three guardrails (below) |

None of these are exotic. All four show up in ordinary week-one loops, and all four are cheaper to design around up front than to discover at 3am.

## Three Guardrails (Install on Day One)

A loop without guardrails is a liability, not an asset.

1. **Iteration cap** — a hard ceiling on the number of rounds, so a stuck loop can't run indefinitely on the strength of a bug alone.
2. **Progress detection** — stop after N iterations with no measurable progress (empty diff, passing-test count unchanged). A loop that isn't moving isn't going to start moving on round 40 just because it didn't on round 10.
3. **Budget cap** — a hard ceiling on spend (tokens or dollars); stop the moment it's hit, no exceptions carved out for "it's almost done."

All three are cheap to add and expensive to skip. None of them require the model to be smarter — they just bound how much damage a dumb failure can do before a human sees it.

## Fresh-Context Iteration (Ralph Pattern)

Named after the technique's originator's fondness for a certain Simpsons character. Simplest possible implementation:

```bash
while :; do cat PROMPT.md | agent ; done
```

That's the whole thing — a dumb loop feeding the same prompt file to a coding agent, over and over. The core design choice: every iteration gets a fresh context. Progress lives in the repo, not in the model's memory — state moves to git and plan files, so the model never has to "remember" anything. Each iteration wakes up, reads the spec, checks the repo's current state, does one thing, commits, and dies; the next iteration picks up everything the last one left behind.

Operating discipline:
- **One task per iteration** — keeps a single iteration from blowing out its context window.
- Reassemble the same context stack deterministically every round (specs + `fix_plan.md`).
- Treat the main context as a scheduler — delegate high-context work (searching, running tests) to subagents.
- Maintain `fix_plan.md` as the loop's external task list; write what the agent learns about the project back to `AGENTS.md`.
- Require "search the codebase before changing it" — never assume a feature isn't implemented; that assumption is what causes duplicate work.
- Leave a "why this matters" comment with every change — the next iteration won't see this iteration's reasoning, only the note it left behind.
- Fallback: commit and tag often; if a run goes off the rails, `git reset --hard` and rerun — cheaper than debugging it by hand.

**Bounds: greenfield projects, roughly 0 to 90%. Do not use this on an existing production codebase.** See the context-reset spectrum in `06-long-running.md`.

## The Loop Stack

A single agent loop doesn't operate alone — it usually sits inside three more loops running at increasingly slower timescales, each one supervising the layer below it.

| Layer | Timescale | What | Example |
|---|---|---|---|
| L1 Agent loop | seconds–minutes | Model + tools, the base loop | Executing a single task |
| L2 Verification loop | minutes | Grader scores against a rubric, retries with feedback | Tests / typecheck / LLM-as-judge |
| L3 Event-driven loop | hours–days | Agent sits on a real event source, runs continuously | Cron, webhook, CI failure, chat message |
| L4 Self-improvement loop | weeks | An analysis agent reads production traces and rewrites the inner layers' config | Trace analysis → updated prompt/tools/rules |

Two points worth calling out:
- L2's grader should start deterministic (cheap: link resolves, CI passes, diff stays in scope) before reaching for LLM-as-judge.
- L4's feedback doesn't loop back to the start — it **rewrites L1's prompt/tool/config directly**. The compounding returns live in L3/L4; most teams over-invest in L1/L2.

Human oversight isn't a yes/no question — it's a question of which layer it's embedded in: confirmation before high-risk actions in L1; a human supplementing the grader in L2; approval before publishing output in L3; review before config changes ship in L4.

## When to Loop

Two axes decide it: can "done" be verified by a machine, and what's the cost of getting it wrong? Start with tasks that are repeatable, machine-verifiable, and low blast-radius: dependency bumps, lint debt cleanup, flaky-test triage, coverage gaps, PR babysitting.

Graduation criteria (must hit these before expanding scope): a week unattended with zero budget overruns, and 90%+ of the PRs it produces merge as-is after human review.

Don't start on the opposite quadrant — one-off, hard-to-verify, high-blast-radius work. That's exactly the profile of a loop that runs a long time, produces something confidently wrong, and gets merged before anyone catches it.

Parallelism constraint: the real bottleneck is human review bandwidth, not compute. Worktrees solve agents stepping on each other's files — they don't solve who reviews the output. Running ten loops in parallel doesn't help if one person still has to read every PR they produce; it just moves the queue from "agent time" to "review time."

## Comprehension Debt

The faster a loop runs, the wider the gap grows between "code that exists" and "code you understand."

Two risks follow from that: verification responsibility doesn't disappear just because the loop says it's done, and **cognitive surrender** — the smoother a loop runs, the easier it becomes to stop forming your own judgment and start rubber-stamping whatever it hands back.

Countermeasure: build forced comprehension checkpoints into the process — keep review inside the loop, sample deep-reads of what it produces instead of skimming diffs, require the agent to attach an explanation for non-trivial changes.

Two people can wire up the identical loop and walk away with opposite outcomes: one uses it to move faster on work they understand deeply, the other uses it to avoid ever having to understand the work at all. The loop can't tell the difference. Only the person running it can.

Build the loop like someone who intends to stay the engineer, not just the person who presses go.
