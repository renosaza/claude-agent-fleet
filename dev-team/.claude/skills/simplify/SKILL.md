---
name: simplify
description: Cut a change to the minimum and remove speculative complexity. Use when implementing, planning, or reviewing a diff to keep it as small and direct as the requirement allows — no premature abstractions, no unasked-for flexibility, no dead code.
---

# simplify

Deliver only the requested functionality. Refactor toward the smallest change that meets the
requirement.

## Rules
- **Only what was asked.** No speculative features, no unasked-for configuration, no "might need
  it later" abstraction. Build for the requirement in front of you.
- **Fewer lines.** If 200 lines could be 50, rewrite it. Prefer the direct expression over a
  layer of indirection that earns nothing.
- **No premature abstraction.** Do not introduce an interface/base class/strategy until there
  are two real callers that need it.
- **Match existing style.** Even if suboptimal; do not reformat or rename unrelated code.
- **Surgical scope.** Change only what the request needs. Do not refactor neighboring sections.
- **Dead code.** Remove only imports/functions your change made obsolete. Flag other dead code;
  do not delete it as part of an unrelated change.
- **Skip impossible edge cases.** Do not add error handling for conditions that cannot occur.

## How to apply
- *Implementing*: write the minimal version first; add complexity only when a concrete case
  forces it.
- *Reviewing*: for each added abstraction or option, ask "what breaks if we delete it?" — if
  nothing, recommend cutting it. Report over-built sections as `simplicity` findings.
