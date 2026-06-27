# Pattern: skills

## Rule
Package reusable expertise as **skills**, in two tiers: **general / cross-language** skills
that apply to any codebase, and **per-language standards** skills that name a single language's
toolchain. An agent loads a skill by listing it in `skills:` frontmatter (full content
preloaded) or invokes one on demand via the Skill tool. **Author every SKILL.md in English**
(see [mcp-and-skills.md](mcp-and-skills.md)); deliverables stay in the target language.

## When it applies
Any dev project, and any agent that keeps repeating the same checklist or procedure. Prefer a
skill over pasting a procedure into an agent prompt: a skill's body loads only when used, so it
costs almost nothing until needed (progressive disclosure).

## How skills work (verified — Claude Code docs)
- A skill is a directory with a required `SKILL.md`. Frontmatter needs a `description` (it
  drives automatic invocation); `name` defaults to the directory name. Keep `SKILL.md` lean and
  move detail into a `references/` folder it links to.
- **Locations** (higher overrides lower on name clash): project `.claude/skills/<name>/`,
  personal `~/.claude/skills/<name>/`, plugins, enterprise. A same-named project/user/enterprise
  skill overrides a **bundled** skill.
- **Where an authored skill lives — default project-local.** An authored skill (general or
  per-language) lives in the project's own `.claude/skills/<name>/` by default, so it ships and
  is version-controlled **with** the project that depends on it. Promote a skill to personal
  `~/.claude/skills/<name>/` **only** when it is genuinely shared across many fleet projects and
  has stabilized; the project-local copy is the default and the unshared case. (Bundled skills
  are reused in place, not copied — see Tier 1.)
- **Progressive disclosure**: only every skill's `name` + `description` sit in context at
  startup; the body loads when the skill is selected. So the `description` must name *when* the
  skill applies — for a per-language skill, name the language and file types so it loads only on
  that language.
- **Subagents and skills**: a subagent's `skills:` frontmatter **preloads full skill content**
  into its context at startup; it can still invoke unlisted project/user/plugin skills via the
  Skill tool. Use `skills:` for the standard a role always needs (e.g. the per-language skill
  for a single-language project); leave optional skills to on-demand invocation.
- **Declaring a skill ≠ applying it.** `skills:` preloads the content but does not guarantee the
  agent runs it over its output. For an effect the deliverable must show (notably `stop-slop`),
  the agent's body must contain an explicit step that invokes it — see the prose rule below.

## stop-slop: prose agents must invoke it, not just declare it
A **prose-producing agent** — one whose deliverable is human-read prose (docs, posts, reports,
PRDs, changelogs, review replies, analytical write-ups) — MUST do **both**:
1. list `stop-slop` in `skills:` frontmatter (preloads the style rules), **and**
2. include an explicit step in its body Workflow / Output that **runs `stop-slop` over the
   generated prose before returning it.** Declaring without invoking ships un-checked prose.

`stop-slop` is **style-only**. It never overrides required sections, sourcing /
anti-hallucination markers (e.g. `[ДАННЫХ НЕДОСТАТОЧНО]`, `[НЕ ПОДТВЕРЖДЕНО]`), legal
disclaimers, or JSON / schema literals — those survive the style pass unchanged
(see [mcp-and-skills.md](mcp-and-skills.md)).

**Non-prose agents** (search / fetch, code-finder, planners, extractors, renderers of
non-prose, mechanical transforms) do **not** need `stop-slop`. Listing it in their frontmatter
is harmless but not required, and is never mandated for them.
- Use `disable-model-invocation: true` for side-effecting workflows you want triggered manually
  (`/skill-name`) rather than auto-loaded.

## Tier 1 — general / cross-language skills
Reuse Claude Code's **bundled** skills where they exist instead of reinventing them; author the
rest. The dev-team roles ([dev-team-roles.md](dev-team-roles.md)) map onto these:
| Skill | Use | Source |
|---|---|---|
| `code-review` | quality/readability/convention review of a diff | **bundled** `/code-review` |
| `debug` | structured root-cause workflow on a failure | **bundled** `/debug` |
| `run` / `verify` | launch the app / confirm a change against the running app | **bundled** (v2.1.145+) |
| `security-review` | OWASP-grade pass: injection, authz, secrets, deps | author |
| `simplify` (refactor) | cut a change to the minimum; remove speculative code | author |
| `test` | author + run tests, report coverage gaps | author (wraps the per-language runner) |

A general skill states the *concern* (what to check); it defers the concrete *commands* to the
per-language skill, so it stays language-agnostic.

## Tier 2 — per-language standards skills
One skill per language, named so its `description` routes by language. Each names the **stable
tool category** (formatter, linter, type-checker, test-runner, vuln-scanner, package manager)
and the **current concrete tool** for it. Categories are canon; the concrete tools are
version-current and the `pattern-curator` refreshes them.

| Concern | Python | TypeScript / Node | Go |
|---|---|---|---|
| package / env | `uv` | `pnpm` (or `npm`) | `go` modules |
| format | `ruff format` | Prettier (or Biome) | `gofmt -s` / `gofumpt` |
| lint (+ autofix) | `ruff check --fix` | ESLint (flat config + `typescript-eslint`) | `golangci-lint run` (v2; wraps `go vet` + `staticcheck`) |
| type-check | `mypy` (canonical); `ty` emerging — **beta**, do not mandate yet | `tsc --noEmit` | (compiler) + `go vet` |
| test | `pytest` (`--cov`) | Vitest (or Jest) | `go test ./... -race -cover` |
| vuln scan | `pip-audit` / `uv` audit | `npm audit` / `pnpm audit` | `govulncheck ./...` |

**Selecting a per-language standard:** the orchestrator (or the agent) detects the language
from the files in scope; the matching per-language skill loads via its description, or a
single-language project preloads it through a role's `skills:` field. A monorepo can place a
nested `.claude/skills/` per package so the right standard applies when editing that package.

## Anti-patterns
- **Reinventing a bundled skill** → writing a `code-review` / `debug` skill from scratch when
  the bundled one exists. Extend or override only with a stated reason.
- **Baking a fad into canon** → pinning a single beta tool as the mandated type-checker. Name
  the category; cite the current tool; mark anything beta (e.g. `ty`).
- **A per-language skill that always loads** → write the `description` to name the language so
  progressive disclosure keeps it out of context until relevant.
- **Russian in a SKILL.md** → skills are English; deliverables can be the target language.
- **Skill overriding domain law** → a style skill (e.g. `stop-slop`) never overrides required
  sections, security findings, or anti-hallucination markers (see [mcp-and-skills.md](mcp-and-skills.md)).
- **Declaring `stop-slop` but never invoking it** → a prose agent lists it in frontmatter yet has
  no body step that runs it, so the prose ships un-checked. Declare **and** invoke.
- **Authoring a project's skill into `~/.claude/skills/`** → a skill only one project uses
  should ship inside that project's `.claude/skills/`, not personal scope. Promote to personal
  only once it is genuinely reused across the fleet.

## Minimal example
```yaml
# per-language standards skill (project-local default): .claude/skills/python-standards/SKILL.md
---
name: python-standards
description: Python code standards and toolchain. Use when writing, reviewing, or testing
  Python (.py) files — formatting, linting, type-checking, and tests for this stack.
---
Toolchain: uv (env), ruff (format + lint --fix), mypy (type-check), pytest --cov (test),
pip-audit (deps). Run `uv run ruff format . && uv run ruff check --fix . && uv run mypy . &&
uv run pytest` before declaring a change done. See references/ for project overrides.
```
```yaml
# a role preloading the standard it always needs
skills: python-standards, simplify
```
