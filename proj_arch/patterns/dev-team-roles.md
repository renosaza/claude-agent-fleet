# Pattern: dev-team-roles

## Rule
A full software-development-lifecycle (SDLC) team is decomposed into **eight worker roles**,
each owning one phase of the lifecycle with an explicit consume → produce contract. The
orchestrator (the project `CLAUDE.md`) plans, routes between them, owns the review/fix loop,
and talks to the user — it never does a role's work itself (see
[orchestrator-workers.md](orchestrator-workers.md)). This is the canonical shape the
`project-scaffolder` builds a dev team from.

## When it applies
Any project whose domain is "write / change / ship software" across the full cycle. A
narrower project (e.g. only review, or only docs) uses the subset of roles it needs — do not
scaffold roles the project will never route to.

## Why
- The roles map to **durable SDLC phases** (plan, explore, build, test, debug, review, secure,
  document), not to invented job titles — so the decomposition survives tooling churn.
- It mirrors Anthropic's orchestrator-worker guidance (a lead agent decomposes and delegates
  to specialized subagents) and Claude Code's own shipped agents/skills: **Explore** (read-only
  Haiku), **Plan**, and the bundled **`/code-review`** and **`/debug`** skills. We follow that
  grain rather than fighting it.
- **One role = one responsibility** keeps each worker's context window clean and its prompt
  tight (see [agent-anatomy.md](agent-anatomy.md)).

## The roles (consume → produce)
| Role | Consumes | Produces |
|---|---|---|
| `architect` (planner) | task / requirements + codebase map | a written plan: approach, file-level change list, interfaces, risks |
| `codebase-explorer` | a question + repo path (read-only) | a focused map: relevant files, call sites, current behavior |
| `implementer` | the plan + relevant files | a code change (diff / changeset) against the agreed interfaces |
| `test-author` | the plan / interfaces (+ the change) | tests covering the new behavior and edge cases |
| `test-runner` | the changeset + test suite | structured pass/fail + failure output (no code changes) |
| `debugger` | failing test output + the diff | root-cause diagnosis + a minimal fix (or a precise report) |
| `code-reviewer` | the diff + the plan | review findings: correctness, readability, simplicity, conventions |
| `security-reviewer` | the diff + dependency/config changes | security findings: injection, authz, secrets, deps (OWASP-grade) |
| `documenter` | the final, merged change | updated docs / changelog / public API notes |

Notes:
- `architect` is a **worker**, not the orchestrator. The orchestrator routes to it for
  non-trivial design; trivial changes may skip straight to `implementer`.
- `code-reviewer` and `security-reviewer` are **separate** roles: general quality and
  adversarial security are different judgments with different model tiers (below).
- `test-runner` is read-and-execute only; it never edits code (that is `implementer` /
  `debugger`). Prefer Claude Code's bundled **`/run`** and **`/verify`** skills here (see
  [skills.md](skills.md)).

## Data dependencies (sequential vs. parallel)
Sequential spine, by data dependency:
```
explore → architect → implement → test/run → (debug if red) → review + security-review → document
```
Parallelize where work is independent (Anthropic's parallelization pattern):
- **Fan-out exploration**: several `codebase-explorer` calls over different areas at once.
- **Review in parallel**: `code-reviewer` + `security-reviewer` both read the same finished
  diff and run concurrently — neither depends on the other.
- **Test authoring alongside implementation**: when the interfaces are fixed by the plan,
  `test-author` can run in parallel with `implementer`; otherwise sequence it after.

The orchestrator owns the **loop**: review or test failures route back to `implementer` or
`debugger`, re-test, then re-review — it decides when the change is done and escalates genuine
ambiguity to the user. (Mirrors the rnd PRD review loop already in the fleet.)

## Per-role model tiering
Apply [model-tiering.md](model-tiering.md) (tier by cognitive load) to these roles. Grounded
in Anthropic's asymmetric-selection guidance (capable orchestrator, cost-effective workers)
and Claude Code's own defaults (Explore is Haiku; the shipped code-reviewer example is Sonnet).

| Role | Tier | Why |
|---|---|---|
| `architect` | **opus** | design under ambiguity; cost of a wrong decomposition is high. |
| `debugger` | **opus** | hard root-cause across a system; multi-step reasoning. |
| `security-reviewer` | **opus** | adversarial judgment; a missed vuln is expensive. |
| `implementer` | **sonnet** | bounded, structured coding; orchestrator raises a *specific* worker to opus for a genuinely hard change. |
| `test-author` | **sonnet** | real but bounded judgment about coverage and edge cases. |
| `code-reviewer` | **sonnet** | quality/readability judgment; matches Claude's shipped reviewer. |
| `documenter` | **sonnet** | clear writing from a known change; sonnet, or haiku for pure changelog appends. |
| `codebase-explorer` | **haiku** | read-only search/summarize; mirrors Claude Code's Explore. |
| `test-runner` | **haiku** | execute the suite, report pass/fail; no judgment. |
| orchestrator session | **sonnet → opus** | sonnet for routine routing; escalate the session to opus for hard decomposition. |

Record the reason for any opus worker in its agent file (per model-tiering.md). Do not put the
whole team on opus — that is the named anti-pattern.

## Anti-patterns
- **One mega "coder" agent** doing plan + implement + test + review → no context hygiene, no
  tier economy, and the reviewer can't be impartial about its own code. Split by role.
- **Self-review only** → the agent that wrote the code also reviews it. Use a separate
  `code-reviewer` (and `security-reviewer`) reading the diff fresh.
- **Skipping the explorer** → implementing blind in an unfamiliar codebase. Map first.
- **Scaffolding every role for a tiny project** → only wire the roles the project routes to.
- **Worker spawns workers** → roles never dispatch each other; the orchestrator sequences them
  (see [orchestrator-workers.md](orchestrator-workers.md)).

## Minimal example
```
dev-team (orchestrator, sonnet → opus)
  ├─ codebase-explorer   haiku   question        → relevant-files map
  ├─ architect           opus    task + map      → plan.md (changes, interfaces, risks)
  ├─ implementer         sonnet  plan + files    → diff
  ├─ test-author         sonnet  plan/interfaces → tests
  ├─ test-runner         haiku   diff + tests    → pass/fail report
  ├─ debugger            opus    failures + diff → root cause + fix
  ├─ code-reviewer       sonnet  diff            → quality findings   ┐ parallel
  ├─ security-reviewer   opus    diff + deps     → security findings  ┘
  └─ documenter          sonnet  merged change   → updated docs / changelog
```
The orchestrator sequences the spine, fans out review in parallel, runs the fix loop, and
reports to the user. No worker dispatches another worker.
