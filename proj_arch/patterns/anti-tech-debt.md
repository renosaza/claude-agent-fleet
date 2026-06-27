# Pattern: anti-tech-debt

## Rule
Every project whose agents **write or modify code** wires three anti-tech-debt practices in
**by default** — not as an opt-in the user requests each time. An AI coder takes the shortest
path: it re-implements helpers it already wrote, leaves scaffolding behind, and ships
shortcuts that read fine in isolation but rot into legacy. The three practices counter that:

1. **Check before create (no duplication).** Before a code-writing worker creates any new
   function / helper / module, it searches the target repo (`Grep`/`Glob`) for an existing one
   that already does — or nearly does — the job, and **reuses or extends** it instead of
   writing a near-duplicate. A genuinely unavoidable duplicate is written but **flagged with
   the reason**.
2. **Cleanup pass after each feature (behavior-preserving refactor).** When a unit of work is
   done, the worker runs the `simplify` skill over the **diff it just produced** — removing
   scaffolding, dead lines, and duplicate logic, unifying to one style. Behavior must not
   change. The pass is **scoped to that diff**, never a free-roaming refactor of unrelated code
   (the surgical-changes discipline still binds — see [orchestrator-workers.md](orchestrator-workers.md)).
3. **Interrogate unexplained choices.** The reviewer role never silently approves AI-written
   code. For each non-trivial design choice it asks "why this over a cleaner / simpler one, and
   does it duplicate something that already exists?" An unjustifiable shortcut or near-duplicate
   is flagged as tech debt, not signed off.

This is a **conditional** pattern, in the same status class as [pdf-reports.md](pdf-reports.md)
and [presentations.md](presentations.md): it governs a class of projects, not the whole fleet.
It is **not** a fourth fleet-wide invariant — the canon has exactly three (orchestrator-only,
English authoring, proactive memory) and this does not change that count.

## When it applies
Any project whose workers write or change source code: a dev team, a single `implementer`-style
worker, a migration / refactor tool, a code-generating agent. The `project-scaffolder` wires
the three practices into such a project by default.

## When it does NOT apply
Non-code projects are **out of scope** — the check-before-create gate, the diff cleanup pass,
and code interrogation are meaningless there:
- **Research** (e.g. `rnd`) — produces analysis / PRDs, not a code diff.
- **Content** (e.g. `smm-med`) — produces prose; prose quality is `stop-slop`'s job (see
  [skills.md](skills.md)), not this pattern's.
- **PDF / rendering** (e.g. `mathematik`) — renders a deliverable, does not maintain a codebase.

A project that incidentally writes a one-off script is not thereby a code project; this applies
where maintaining code IS the domain.

## Why
- **Duplicate code is the dominant AI failure mode.** The agent forgets it already wrote a
  similar function and clones it; the codebase fills with near-duplicates that drift apart.
  Check-before-create is the DRY discipline made a mandatory gate.
- **Shortest-path code accretes scaffolding.** A behavior-preserving cleanup pass over the
  fresh diff pays down the debt while the author still has full context — cheaper than a later
  dedicated refactor sprint.
- **AI-written code is not trustworthy by default.** A reviewer that interrogates each
  non-trivial choice catches the crutch the code cannot explain, before it ships as legacy.

These are durable engineering disciplines (DRY, behavior-preserving refactor, adversarial
review), not version-specific Claude Code mechanics — they do not expire with a release.

## How (wiring for the scaffolder)
- **Code-writing worker(s)** (e.g. an `implementer`) own **rules 1 and 2**. They already need
  `Grep`/`Glob` (to search the repo) and the `simplify` skill (the cleanup pass) — **these are
  the only capabilities required; no new tool grants.** Rule 1 is a mandatory step before
  writing any new symbol, with the reuse result recorded in the report; rule 2 is a mandatory
  step before declaring the work done, scoped to the diff.
- **Reviewer worker** (e.g. a `code-reviewer`) owns **rule 3**: a body step that interrogates
  each non-trivial choice and the duplication question, and a hard rule that an unexplained
  choice is never approved.
- **Orchestrator `CLAUDE.md`** states anti-tech-debt is built in. Drop one line into the
  project's Hard Rules:

  > Anti-tech-debt is on by default (see `proj_arch/patterns/anti-tech-debt.md`): the
  > implementer runs check-before-create and a diff-scoped cleanup pass; the reviewer
  > interrogates unexplained choices. This is built in, not a per-request opt-in.

The dev-team implementation is the worked example: `dev-team/.claude/agents/implementer.md`
(check-before-create gate + cleanup pass, both reported in its Output) and
`dev-team/.claude/agents/code-reviewer.md` (the interrogation step + the "never sign off an
unexplained choice" hard rule). The practices were sourced from a `@vibecoderz` anti-tech-debt
("vibecoding") set and proven in dev-team before being promoted here.

## Anti-patterns
- **Making it opt-in.** Asking the user each time whether to dedup / clean up. It is on by
  default; the user can waive it, not invoke it.
- **A free-roaming cleanup pass.** Treating rule 2 as license to refactor unrelated code. The
  pass is scoped to the diff just produced; surgical-changes discipline holds.
- **Cloning instead of reusing.** Writing a near-duplicate without running the
  check-before-create gate, or without flagging the reason when a duplicate is unavoidable.
- **Silent approval.** A reviewer signing off AI code without interrogating its non-trivial
  choices — the shortcut ships as legacy.
- **Granting new tools for it.** It needs only `Grep`/`Glob` + `simplify`, which a code worker
  already has; do not bolt on extra capabilities to "support" it.
- **Applying it to a non-code project.** Wiring a check-before-create gate into a research or
  content project, where there is no codebase to dedup.

## Minimal example
Implementer workflow (rules 1 and 2), abbreviated:
```
2. Check before create — Grep/Glob the repo for an existing function that does the job;
   reuse or extend it. Unavoidable duplicate → write it, flag WHY in the report.
...
5. Cleanup pass — run `simplify` over the diff just written: remove scaffolding / dead /
   duplicate lines, unify to one style. Behavior unchanged. Scoped to this diff only.
```
Code-reviewer workflow (rule 3), abbreviated:
```
3. Interrogate each non-trivial choice: "why this over a cleaner one, any duplication?"
   An unjustified shortcut or near-duplicate is flagged as tech debt, never approved.
```
Orchestrator `CLAUDE.md` Hard Rules gets the one-line template above. No new tools: the
implementer already lists `Grep, Glob` + `skills: simplify`; the reviewer already lists
`Grep, Glob` + `skills: simplify`.
