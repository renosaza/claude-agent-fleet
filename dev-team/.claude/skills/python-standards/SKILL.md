---
name: python-standards
description: Python code standards and toolchain. Use when writing, reviewing, testing, or debugging Python (.py / pyproject.toml / requirements) — formatting, linting, type-checking, tests, and dependency vuln scanning for this stack.
---

# python-standards

Tool categories are the standard; concrete tools are current (refreshed by pattern-curator).
Do not hard-pin versions in agent prose.

| Concern | Tool | Command |
|---|---|---|
| package / env | `uv` | `uv sync`, `uv run <cmd>` |
| format | `ruff format` | `uv run ruff format .` |
| lint (+ autofix) | `ruff check` | `uv run ruff check --fix .` |
| type-check | `mypy` (canonical; `ty` is beta — do not mandate) | `uv run mypy .` |
| test | `pytest` | `uv run pytest --cov` |
| vuln scan | `pip-audit` / `uv` audit | `uv run pip-audit` |

## Definition of done (run on changed files before declaring done)
```
uv run ruff format . && uv run ruff check --fix . && uv run mypy . && uv run pytest
```
Follow the repo's existing config (`pyproject.toml`/`ruff.toml`/`mypy.ini`) when present —
the repo's settings win over these defaults. See `references/` for project overrides.
