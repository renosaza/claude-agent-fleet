# Patterns — canonical rules for the Claude project fleet

This directory is the single source of truth for how every project and agent in
`~/Documents/claude/` is built. `proj_arch` generates new projects from these rules; the
`pattern-curator` agent keeps them current against the live Claude Code release.

Each pattern file is short and prescriptive: **what** the rule is, **when** it applies, the
**anti-pattern** it prevents, and a minimal example. Read the relevant pattern before
scaffolding a project or authoring an agent — do not reinvent conventions.

| Pattern | Governs |
|---|---|
| [orchestrator-workers.md](orchestrator-workers.md) | Main agent orchestrates; subagents do the work. |
| [agent-anatomy.md](agent-anatomy.md) | Structure of a subagent file: frontmatter + body. |
| [model-tiering.md](model-tiering.md) | Which model tier (haiku / sonnet / opus) per role. |
| [proactive-memory.md](proactive-memory.md) | Query memorygraph before, store after — always. |
| [permissions-safety.md](permissions-safety.md) | `allow` / `ask` / `deny` and filesystem scope. |
| [mcp-and-skills.md](mcp-and-skills.md) | Wiring MCP servers and skills; English-only authoring. |
| [skills.md](skills.md) | General + per-language skills; bundled skills to reuse; selection. |
| [dev-team-roles.md](dev-team-roles.md) | Full-cycle SDLC team: roles, contracts, dependencies, per-role tier. |
| [anti-tech-debt.md](anti-tech-debt.md) | Code projects only: check-before-create, diff-scoped cleanup pass, reviewer interrogation — on by default. |
| [pdf-reports.md](pdf-reports.md) | PDF report style; PDF is opt-in, built only on explicit user request. |
| [presentations.md](presentations.md) | Grain Deck System — the default design for every slide deck the fleet produces. |

## Fleet-wide invariants (non-negotiable)

These three rules hold for every project `proj_arch` creates or governs:

1. **The main agent is an orchestrator only.** Real work happens in subagents. See
   [orchestrator-workers.md](orchestrator-workers.md).
2. **Skills and agent files are written in English.** User-facing communication and
   published deliverables stay in the project's target language (Russian here). See
   [mcp-and-skills.md](mcp-and-skills.md).
3. **Memory access is proactive.** Every agent queries memorygraph before acting and stores
   what it learned after. See [proactive-memory.md](proactive-memory.md).

## Verified environment (update via pattern-curator)

- Claude Code **2.1.178**.
- Nested subagent spawning is technically allowed since **v2.1.172** — but keeping
  orchestration in the main agent is still the preferred shape (clarity, predictable cost).
- Experimental agent teams enabled (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`).
- Subagents live in `<project>/.claude/agents/*.md`; the global `~/.claude/agents/` is empty
  by design.
