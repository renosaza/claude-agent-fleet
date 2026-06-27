# Running this fleet on OpenAI Codex CLI

This repository was authored for **Claude Code**, but every project ships a parallel
**Codex CLI** version so you can run the same agent architecture on Codex. Nothing is
Claude-specific in the ideas — only the file layout differs.

## How the two map

| Concept | Claude Code | Codex CLI |
|---|---|---|
| Project instructions | `CLAUDE.md` (project root) | `AGENTS.md` (project root) — same content |
| Subagent / worker | `.claude/agents/<name>.md` (YAML front-matter + markdown body) | `~/.codex/agents/<name>.toml` (custom agent) |
| MCP servers | `.mcp.json` (JSON) | `codex/config.toml` → `[mcp_servers.*]` (TOML) |
| Permissions / sandbox | `.claude/settings.json` | `config.toml` → `approval_policy`, `sandbox_mode` |
| Skills | `.claude/skills/<name>/SKILL.md` | `~/.codex/skills/<name>/` |

Two important differences in Codex's model:

1. **Custom agents are global, not per-repo.** Codex loads custom agents from
   `~/.codex/agents/` (one `.toml` per agent), so they cannot live inside the repo the
   way Claude's `.claude/agents/` do. This repo keeps each project's Codex agents under
   `<project>/codex/agents/` purely for sharing — you install them by copying into
   `~/.codex/agents/`.
2. **Subagents are spawned only on explicit request**, and the orchestrator references
   them by name in the prompt (e.g. *"have `code-reviewer` judge the diff"*). The
   orchestration logic in each `AGENTS.md` already names its workers, so this works as-is.

## Per-project Codex layout

Each project directory contains:

```
<project>/
├── AGENTS.md            # = CLAUDE.md, read by Codex as project instructions
└── codex/
    ├── config.toml      # MCP servers + sandbox/approval defaults
    └── agents/
        └── *.toml       # one custom agent per worker
```

## Install (per project)

From inside a project directory (e.g. `dev-team/`):

```sh
# 1. Project instructions are already in place (AGENTS.md at the project root).

# 2. Install the custom agents globally:
mkdir -p ~/.codex/agents
cp codex/agents/*.toml ~/.codex/agents/

# 3. Configure MCP servers + sandbox. Either:
#    (a) trusted-project scope — copy config into a project-local .codex/:
mkdir -p .codex && cp codex/config.toml .codex/config.toml
#    (b) or merge the [mcp_servers.*] blocks into ~/.codex/config.toml by hand.

# 4. Restart Codex (or open a new chat) so agents and config reload.
```

> **Heads-up on agent name collisions.** Because `~/.codex/agents/` is global, installing
> two projects at once can collide on shared names (e.g. several projects use a
> `code-finder`/`changelog-writer`). Install one project's agents at a time, or prefix the
> filenames before copying.

## Models

Claude model tiers were mapped to Codex as follows (tune in each `.toml`):

| Claude tier | Codex `model` | `model_reasoning_effort` |
|---|---|---|
| `opus` | `gpt-5.5` | `high` |
| `sonnet` | `gpt-5.5` | `medium` |
| `haiku` | `gpt-5.4-mini` | `low` |

`gpt-5.5` is the current default and strongest model; `gpt-5.4-mini` is the light model,
reserved for the most atomic/mechanical workers (e.g. `registry-keeper`, `changelog-writer`,
`test-runner`). `model_reasoning_effort` accepts `none | minimal | low | medium | high | xhigh`.
Set the model/effort that fits your account; the mapping is a starting point, not a
requirement.

## MCP paths to fix

Some MCP servers reference machine-local install paths or need a one-time install:

- `proj_arch`'s `cloakbrowser` / `memorygraph` use `${HOME}/.claude/mcp-venvs/...` — point
  these at your actual MCP binaries, or drop the servers if you don't run them.
- `dev-team`'s `codebase-memory` expects the binary at `${HOME}/.local/bin/codebase-memory-mcp`.
  Install it once with
  `curl -fsSL https://raw.githubusercontent.com/DeusData/codebase-memory-mcp/main/install.sh | bash -s -- --skip-config`
  (binary only, no global config changes), then it works as wired.

## How MCP servers get downloaded

Most servers here **download themselves on first launch** — you do not pre-install them.
Codex starts the server with the `command` from `config.toml`, and the launcher fetches the
package:

- `command = "npx"` (e.g. `sequential-thinking`, `context7`) — `npx -y <pkg>` downloads the
  npm package on first run and caches it. Requires **Node.js + `npx`** on `PATH`.
- `command = "uvx"` (e.g. `fetch`, `headroom`) — `uvx <pkg>` downloads the Python package on
  first run and caches it. Requires **`uv` / `uvx`** on `PATH`
  (`curl -LsSf https://astral.sh/uv/install.sh | sh`).

So the only prerequisite for the self-installing servers is having `npx` and `uvx` available;
the packages themselves arrive automatically the first time an agent uses the server. The two
exceptions are the manual servers above: `cloakbrowser` / `memorygraph` point at your own
install paths, and `codebase-memory` is a single binary you install once with the command
shown above.

## Loading all MCP servers (globally vs. per project)

Codex has **no single "enable all" switch.** Servers load from one of two places:

- **Per project (trusted):** the `[mcp_servers.*]` blocks in that project's
  `.codex/config.toml` (what `cp codex/config.toml .codex/config.toml` sets up). Active only
  while you work in that project.
- **Globally for every project:** merge the `[mcp_servers.*]` blocks into
  `~/.codex/config.toml`. Then every Codex session sees those servers regardless of directory.

To make a project's servers available everywhere, append its blocks to the global file:

```sh
cat dev-team/codex/config.toml >> ~/.codex/config.toml   # then de-dup [mcp_servers.*] by hand
# or register one at a time with the CLI:
codex mcp add codebase-memory -- ${HOME}/.local/bin/codebase-memory-mcp
```

Disable an individual server without deleting its block by adding `enabled = false` to it.
There is no bulk toggle — list the servers you want; anything not in a loaded config is simply
not started.

## References

- AGENTS.md: https://developers.openai.com/codex/guides/agents-md
- Custom agents (TOML): https://developers.openai.com/codex/subagents
- MCP in Codex: https://developers.openai.com/codex/mcp
- Config reference: https://developers.openai.com/codex/config-reference
