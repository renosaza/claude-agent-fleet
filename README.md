# Claude Agent Fleet

A shareable reference architecture for **multi-agent projects** built on
[Claude Code](https://claude.com/claude-code) — with a parallel **OpenAI Codex CLI**
version of every project (see [`CODEX.md`](CODEX.md)).

Every project follows one shape: a **main agent that only orchestrates**, and **worker
subagents that do the actual work**. The orchestrator decomposes a task, routes each phase
to the right worker, runs the review loop, and talks to the user — it never does a worker's
job itself.

This repo contains the **architecture and patterns only**. All working data, generated
outputs, agent memory, and personal/client material have been stripped out — what's left is
the reusable skeleton you can copy, study, and adapt.

## What's inside

| Project | What it does | Workers |
|---|---|---|
| **`proj_arch`** | The **project factory** and architectural authority. Holds the canon every other project is built from, generates new projects + their agents, and keeps a fleet registry. | `project-scaffolder`, `agent-author`, `pattern-curator`, `registry-keeper` |
| **`dev-team`** | A full-cycle **SDLC agent team**: explore → architect → implement → test → debug → review → security-review → document. | 9 specialists (architect, implementer, code-reviewer, test-author, …) |
| **`rnd`** | A **deep-research unit**: forms falsifiable hypotheses, fans out parallel searchers, then runs a synthesis / red-team / audit analysis stack. | 11 workers (research-planner, web-researcher, synthesizer, red-team-council, …) |
| **`mathematik`** | A small **pipeline**: photos of math assignments → a single typeset PDF with full worked solutions. | `vision-extractor`, `math-solver`, `pdf-composer` |
| **`smm-med`** | An **SMM agent** for a medical clinic: content plans, posts, FAQ, review handling — within the rules of medical-services advertising. | `content-planner`, `content-writer`, `faq-builder`, `reviews-manager` |

`proj_arch` is the heart of it. Its [`patterns/`](proj_arch/patterns/) directory is the
single source of truth — the other four projects are concrete instances generated from
that canon.

## The canon (in `proj_arch/patterns/`)

Three invariants hold for every project:

1. **Main agent is an orchestrator only** — real work lives in worker subagents.
2. **Skills and agent files are written in English** — user-facing deliverables stay in the
   project's target language.
3. **Memory is proactive** — every agent queries its memory before acting and stores what it
   learned after.

Plus conditional canons for PDF reports, presentation decks, anti-tech-debt rules for code
projects, model tiering, and permission safety. See
[`proj_arch/patterns/README.md`](proj_arch/patterns/README.md).

## Layout

```
<project>/
├── CLAUDE.md            # orchestrator instructions (Claude Code)
├── AGENTS.md            # same content, for Codex CLI
├── .mcp.json            # MCP servers (Claude)
├── .claude/
│   ├── agents/*.md      # worker subagents (Claude)
│   └── skills/          # dev-team only
└── codex/               # Codex CLI version
    ├── config.toml      # MCP servers + sandbox/approval
    └── agents/*.toml    # worker subagents (Codex)

proj_arch/
├── patterns/            # the canon — one rule per file
└── templates/           # fill-in skeletons the scaffolder uses
```

## Using it

### Claude Code
Open any project directory with Claude Code. `CLAUDE.md` is loaded automatically and the
workers in `.claude/agents/` become available. Review `.mcp.json` and point any
machine-local MCP server paths at your own install (see notes below).

### Codex CLI
See [`CODEX.md`](CODEX.md) for the full install steps (copy `codex/agents/*.toml` into
`~/.codex/agents/`, set up `config.toml`).

## What was removed

To make this safe to share, the export omits: all generated outputs and reports, research
sources, input photos, the project registry/changelogs, agent memory (memorygraph), and
local `settings.local.json` permission files. Machine-local paths were replaced with
placeholders — a few MCP servers reference `${HOME}/.claude/mcp-venvs/…` or `/path/to/…`
and need to be pointed at your own installs (or removed).

## License

[MIT](LICENSE).
