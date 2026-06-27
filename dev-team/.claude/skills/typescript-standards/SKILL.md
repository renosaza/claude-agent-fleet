---
name: typescript-standards
description: TypeScript / Node code standards and toolchain. Use when writing, reviewing, testing, or debugging TypeScript / JavaScript (.ts / .tsx / .js / package.json) — formatting, linting, type-checking, tests, and dependency audit for this stack.
---

# typescript-standards

Tool categories are the standard; concrete tools are current (refreshed by pattern-curator).
Do not hard-pin versions in agent prose.

| Concern | Tool | Command |
|---|---|---|
| package / env | `pnpm` (or `npm`) | `pnpm install` |
| format | Prettier (or Biome) | `pnpm exec prettier --write .` |
| lint (+ autofix) | ESLint flat config + `typescript-eslint` | `pnpm exec eslint . --fix` |
| type-check | `tsc` | `pnpm exec tsc --noEmit` |
| test | Vitest (or Jest) | `pnpm test` (`pnpm exec vitest run --coverage`) |
| vuln scan | `pnpm audit` (or `npm audit`) | `pnpm audit` |

## Definition of done (run on changed files before declaring done)
```
pnpm exec prettier --write . && pnpm exec eslint . --fix && pnpm exec tsc --noEmit && pnpm test
```
Use the repo's package manager and config (`eslint.config.*`, `tsconfig.json`, `.prettierrc`)
when present — the repo's settings win over these defaults. See `references/` for overrides.
