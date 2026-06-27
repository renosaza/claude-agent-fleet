# Pattern: orchestrator-workers

## Rule
The **main agent** of a project (its `CLAUDE.md`) is an **orchestrator**. It plans, routes,
sequences, escalates to the user, and assembles results. It does **not** do the domain work
itself. The domain work lives in **worker subagents** under `<project>/.claude/agents/`,
each dispatched via the `Agent` tool.

## When it applies
Every project with more than one distinct kind of work, or where steps have different model
needs or benefit from a clean context window. This is the default shape for the fleet —
deviate only with a recorded reason.

## Why
- **Clarity.** One place owns "what happens next"; workers own "how this step is done".
- **Context hygiene.** Each worker gets a fresh window scoped to its task; the orchestrator's
  context is not polluted by a worker's intermediate reasoning.
- **Model economy.** Cheap orchestration (sonnet) dispatches expensive reasoning (opus) only
  where it pays off. See [model-tiering.md](model-tiering.md).

## How
- Give the orchestrator a **routing table**: trigger → worker → why that worker.
- Dispatch workers via the `Agent` tool. Independent workers go out **in parallel** (one
  message, multiple calls); dependent ones are sequenced by their data dependency.
- Workers return a short structured report to the orchestrator; the orchestrator decides the
  next step and talks to the user.
- The orchestrator owns all user interaction (questions, escalations, final summary).

## Anti-patterns
- **Worker spawns workers.** Nested spawning is technically allowed (Claude Code v2.1.172+),
  but as a fleet rule a worker does **not** dispatch other workers — it keeps cost
  predictable and the data flow legible. Orchestration stays in the main agent. A worker's
  `tools` list therefore omits `Agent` (enforced — see [agent-anatomy.md](agent-anatomy.md)).
- **Orchestrator does the work itself.** If the main agent starts solving the domain problem
  inline, the boundary has leaked; move that logic into a worker.
- **Hidden state between workers.** Workers communicate through files/JSON the orchestrator
  passes, not through assumed shared memory.

## Minimal example
```
mathematik (orchestrator)
  ├─ vision-extractor   (sonnet)  photos → tasks.json
  ├─ math-solver        (opus)    tasks.json → solutions.json
  └─ pdf-composer       (sonnet)  solutions.json → output.pdf
```
The orchestrator sequences the three by data dependency, escalates conflicts to the user, and
reports the result. None of the three workers dispatches another agent.
