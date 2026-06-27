# Claude Agent Fleet

> 🇷🇺 **Русская версия с пошаговой инструкцией и объяснением «на пальцах»:** [`README.ru.md`](README.ru.md)

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

## Deployment

You only need to set up the project(s) you actually want to run. Each project is
self-contained — pick one and follow the steps for your tool.

### Prerequisites

| Tool | Needed for | Install |
|---|---|---|
| **Claude Code** | running the Claude version | https://claude.com/claude-code |
| **OpenAI Codex CLI** | running the Codex version | `npm i -g @openai/codex` (see https://developers.openai.com/codex) |
| **Node.js + `npx`** | `sequential-thinking`, `context7` MCP servers | https://nodejs.org (LTS) |
| **Python + [`uv`](https://docs.astral.sh/uv/) (`uvx`)** | `fetch`, `headroom` MCP servers | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **`git`** | cloning this repo | preinstalled on macOS/Linux |

Most MCP servers used here **install themselves on first launch** via `npx`/`uvx` — you do
not pre-install them. Only a few need manual setup (see the MCP table below).

### 1. Clone

```sh
git clone https://github.com/renosaza/claude-agent-fleet.git
cd claude-agent-fleet
```

### 2A. Run on Claude Code

Claude Code auto-discovers `CLAUDE.md`, `.claude/agents/`, and `.mcp.json` from the
directory it is launched in. So:

```sh
cd dev-team          # or proj_arch / rnd / mathematik / smm-med
claude               # launch Claude Code in this directory
```

On first run Claude Code asks to approve the project's MCP servers — accept the ones you
want. The `CLAUDE.md` is loaded as the orchestrator's instructions, and the workers in
`.claude/agents/` become dispatchable automatically. That's the whole setup for projects
whose MCP servers are all auto-installing (`dev-team`, `rnd`, `mathematik`).

For `proj_arch` (and any project using `cloakbrowser` / `memorygraph`), edit that project's
`.mcp.json` first — see **MCP servers** below. `dev-team` adds one manually-installed server,
`codebase-memory` (see the same table).

### 2B. Run on Codex CLI

Codex uses `AGENTS.md` (already at each project root) for instructions, but its custom
agents live **globally** in `~/.codex/agents/`, not in the repo. From a project directory:

```sh
cd dev-team

# install this project's workers as global Codex custom agents
mkdir -p ~/.codex/agents
cp codex/agents/*.toml ~/.codex/agents/

# install MCP servers + sandbox config as a trusted project-local config
mkdir -p .codex && cp codex/config.toml .codex/config.toml

codex                # launch; restart if it was already running, so agents reload
```

Full details, the model-tier mapping, and a note on agent-name collisions between projects
are in [`CODEX.md`](CODEX.md).

### MCP servers — what each needs

| Server | Used by | Setup |
|---|---|---|
| `sequential-thinking` | all | none — `npx` auto-installs |
| `fetch` | all | none — `uvx` auto-installs |
| `context7` | proj_arch | none — `npx` auto-installs |
| `headroom` | proj_arch | none — `uvx` auto-installs (`headroom-ai[mcp]`) |
| `cloakbrowser` | proj_arch | **manual** — point the path at your own install, or remove the block |
| `memorygraph` | proj_arch | **manual** — point the path at your own install, or remove the block |
| `codebase-memory` | dev-team | **manual** — install the binary once (one command), then it works; see below |

The first two manual servers use placeholder paths (`${HOME}/.claude/mcp-venvs/...`). Edit
them in the project's `.mcp.json` (Claude) and `codex/config.toml` (Codex), or delete the
server's block if you don't run it — the projects work without them, just without that
capability (e.g. no persistent memory without `memorygraph`).

`codebase-memory` ([DeusData/codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp))
is a code-intelligence server: it indexes a codebase into a per-project knowledge graph so the
dev-team workers can ask structural questions without re-reading files. Install the binary once:

```sh
curl -fsSL https://raw.githubusercontent.com/DeusData/codebase-memory-mcp/main/install.sh \
  | bash -s -- --skip-config
```

`--skip-config` installs **only** the binary (to `~/.local/bin/codebase-memory-mcp`) and does
**not** touch any agent's global config — the repo already wires the server into `dev-team`'s
`.mcp.json` / `codex/config.toml`. Once installed, open the project and tell the agent
*"index this project"* to build the graph for the current folder. Each indexed folder gets its
own isolated graph (stored under `~/.cache/codebase-memory-mcp/`), so different working folders
keep separate code maps.

### 3. Verify

- **Claude Code:** inside a project, run `/agents` to confirm the workers are listed, and
  `/mcp` to confirm the MCP servers connected.
- **Codex CLI:** run `/mcp` to list connected servers; reference a worker by name in a
  prompt (e.g. *"have `code-reviewer` look at this diff"*) to confirm custom agents loaded.

## What was removed

To make this safe to share, the export omits: all generated outputs and reports, research
sources, input photos, the project registry/changelogs, agent memory (memorygraph), and
local `settings.local.json` permission files. Machine-local paths were replaced with
placeholders — a couple of MCP servers reference `${HOME}/.claude/mcp-venvs/…` and need to be
pointed at your own installs (or removed).

## License

[MIT](LICENSE).
