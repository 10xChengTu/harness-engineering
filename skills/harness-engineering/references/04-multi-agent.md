# 04 - Multi-Agent Architecture

When to use multiple agents, how to coordinate them, and communication protocols.

## When to Use Multi-Agent

**Single agent** when: Task fits in one context window, one domain, straightforward.
**Multi-agent** when: Task spans domains, exceeds context limits, benefits from separation of concerns, or needs independent evaluation.

**Start single, split when you hit walls.** Don't prematurely architect multi-agent.

## Foundational Patterns

### 1. Prompt Chaining
Sequential pipeline: Agent A output → Agent B input → Agent C input.
Use when tasks have clear stages. Each stage can be optimized independently.

### 2. Routing
Classifier sends input to specialized agent. Like a dispatcher.
Use when inputs vary widely and different expertise is needed.

### 3. Parallelization
Multiple agents work on independent subtasks simultaneously.
Use when tasks are decomposable and don't share state.

### 4. Orchestrator-Workers
Central coordinator delegates to specialized workers.
Use when task decomposition requires judgment (not predetermined).

### 5. Generator-Evaluator (GAN-Inspired)
One agent creates, another judges. Iterate until quality threshold met.
Use when quality assessment is possible but self-assessment is unreliable.

## Coordinator Design

The coordinator (lead agent) is the most critical piece. Key lessons:

### Teach Detail in Delegation
Bad: "Build the auth system"
Good: "Build JWT auth with: refresh tokens (7d expiry), httpOnly cookies, /auth/login and /auth/refresh endpoints. Use bcrypt for passwords. Follow patterns in src/auth/existing.ts"

**Scale effort to query complexity** — simple questions get simple delegation, complex ones get detailed plans.

### Communication Protocols

#### File-Based Communication
Agents communicate through files on disk. Reliable, auditable, survives crashes.

```
.harness/
├── plan.md          # Planner writes, others read
├── sprint-contract.md  # Generator + Evaluator negotiate
├── eval-report.md   # Evaluator writes, Generator reads
└── handoff.md       # Current agent writes for next agent
```

#### Intent Marker Protocol (Structured Tags)
For supervisor-worker patterns, use explicit tags:

| Tag | Meaning |
|-----|---------|
| `[STATUS_REQUEST]` | Supervisor asks for progress |
| `[REVIEW_REQUEST]` | Worker submits for review |
| `[ACK]` | Acknowledge receipt |
| `[ESCALATE]` | Worker can't resolve, needs help |

**Max 3 messages per exchange** — prevents infinite loops between agents.

#### Sprint Contract Pattern
Before work begins, generator and evaluator agree on:
1. What will be built
2. How to verify it's done (testable criteria)
3. Quality thresholds

This bridges the gap between high-level specs and implementation verification.

## Separation of Concerns

### Why Separate Generator and Evaluator
- **Self-evaluation bias**: Agents rate their own work too highly
- **Easier calibration**: Tuning an evaluator for skepticism is easier than making a generator self-critical
- **Focused context**: Each agent has optimized context for its role

### When Evaluator Adds Value
The evaluator earns its cost when tasks are at the **edge of model capability**. As models improve, the boundary moves — tasks that needed evaluation before may not anymore. Re-assess periodically.

## Sub-Agent Delegation

```markdown
## Delegation Template
1. TASK: Specific, atomic goal
2. CONTEXT: Relevant files, patterns, constraints
3. DELIVERABLE: What to produce
4. CONSTRAINTS: What NOT to do
5. VERIFICATION: How to know it's done
```

### Let Agents Self-Improve
Allow sub-agents to update their own tools and patterns. An agent that discovers a better approach should be able to encode it for future runs.

## Broad-Then-Narrow Search
For research/exploration agents:
1. Cast a wide net first (explore many possibilities)
2. Then narrow to promising paths
3. Don't commit to the first solution found
