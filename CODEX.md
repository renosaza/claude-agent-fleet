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
| `opus` | `gpt-5.2-codex` | `high` |
| `sonnet` | `gpt-5.2-codex` | `medium` |
| `haiku` | `gpt-5.2-codex` | `low` |

Set the model/effort that fits your account; the mapping is a starting point, not a
requirement.

## MCP paths to fix

Two MCP servers reference machine-local install paths (placeholders in this repo):

- `cloakbrowser` / `memorygraph` use `${HOME}/.claude/mcp-venvs/...` — point these at your
  actual MCP binaries, or drop the servers if you don't run them.
- `smm-med`'s `max-mcp` uses `/path/to/max-mcp` — a separate self-hosted server that is
  **not** part of this repo; remove the block if you don't have it.

## References

- AGENTS.md: https://developers.openai.com/codex/guides/agents-md
- Custom agents (TOML): https://developers.openai.com/codex/subagents
- MCP in Codex: https://developers.openai.com/codex/mcp
- Config reference: https://developers.openai.com/codex/config-reference
