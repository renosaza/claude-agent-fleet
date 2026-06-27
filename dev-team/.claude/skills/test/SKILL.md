---
name: test
description: Author and assess tests for a change — happy path, edge cases, and failure modes — and read the resulting coverage. Use when writing tests or judging test adequacy. Defers the concrete framework and run command to the per-language standards skill.
---

# test

State *what* to cover; the per-language standards skill supplies the framework (pytest /
Vitest-Jest / `go test`) and the run command.

## What to cover
- **Happy path** — the primary behavior the change adds, asserted against the plan's acceptance
  criteria.
- **Boundaries** — empty / single / max inputs, off-by-one ranges, zero and negative values.
- **Failure modes** — invalid input, missing resources, timeouts; assert the error is raised and
  shaped as the interface promises.
- **Regression guard** — a test that fails if the specific behavior this change introduces
  regresses.

## How to write
- Assert **behavior**, not implementation detail — tests should survive a refactor that keeps
  behavior.
- One reason to fail per test; a clear name that states the case.
- Follow the repo's existing test layout and fixtures; do not introduce a new framework.
- Cover each acceptance criterion with at least one test; record gaps you cannot yet cover
  (e.g. an interface still open).

## Coverage
Read the coverage report from the run; report uncovered branches that matter (not a raw % chase).
Do not lower a threshold to make a suite pass.
