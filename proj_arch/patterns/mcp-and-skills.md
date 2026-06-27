# Pattern: mcp-and-skills

## Rule
Wire only the MCP servers and skills a project actually uses. **Author all skills and agent
files in English.** User-facing communication and published deliverables stay in the
project's target language (Russian here).

## MCP servers
- **Project-local** servers go in `<project>/.mcp.json`. **Shared** servers
  (`memorygraph`, `cloakbrowser`, `context7`, `headroom`, `fetch`, `sequential-thinking`) are
  configured once and reused; with `enableAllProjectMcpServers: true` they are available to
  the project.
- Tool names in Claude Code are namespaced `mcp__<server>__<tool>` — agents reference that
  full form in their `tools:` list (e.g. `mcp__memorygraph__search_memories`).
- The standard shared set and its purpose:
  | Server | Purpose |
  |---|---|
  | `memorygraph` | Persistent knowledge graph (see [proactive-memory.md](proactive-memory.md)). |
  | `headroom` | Reversible local compression of large tool output before it eats context. |
  | `context7` | Verify library/API facts and obscure references before stating them. |
  | `cloakbrowser` | Headless Chromium: navigate, evaluate, print-to-PDF. |
  | `fetch` | URL → markdown. |
  | `sequential-thinking` | Explicit decomposition for hard multi-step reasoning. |
- Don't grant a server to an agent that doesn't use it. Match the agent's `tools:` to its job.

## Skills
- Skills live in `~/.claude/skills/<name>/SKILL.md` (personal), `.claude/skills/<name>/SKILL.md`
  (project), or a plugin; an agent references one via its `skills:` frontmatter key (which
  **preloads the full skill content**) or invokes it on demand via the Skill tool.
- A skill's own files (SKILL.md, instructions) are **written in English**, regardless of the
  deliverable language. `stop-slop` is the common one for prose-producing agents; it governs
  **style only** and never overrides a project's hard rules (legal disclaimers,
  anti-hallucination markers, required sections).
- **A prose-producing agent must both declare and invoke `stop-slop`**: list it in `skills:`
  **and** add an explicit body step that runs it over the generated prose before returning —
  preloading alone does not apply it. Non-prose agents do not need it. Full rule and the
  declare-without-invoke anti-pattern in [skills.md](skills.md).
- For dev projects — the general / per-language skill tiers, the bundled skills to reuse, and
  how a per-language standard is selected — see [skills.md](skills.md). This file owns only the
  English-authoring invariant and the MCP wiring.

## Language rule (fleet-wide invariant)
- **English**: every agent file, every skill, every pattern, every template, internal
  reasoning, code, comments, commit messages.
- **Russian** (target language): replies to the user, and any published content (posts,
  reports, PDF solutions). Mirrors `~/.claude/CLAUDE.md`.
- Practically: a `grep` for Cyrillic over `patterns/`, `templates/`, and `.claude/agents/`
  should return nothing — except quoted target-language **literals** (e.g. anti-hallucination
  marker tokens like `[НЕ ПОДТВЕРЖДЕНО]` shown as examples), which are data, not authored prose.

## Anti-patterns
- **Server sprawl** → adding MCP servers a project never calls. Wire what you use.
- **Russian agent prompts** → harms portability and consistency; author in English.
- **Skill overriding domain law** → `stop-slop` "no em-dash / no adverbs" heuristics never
  override required legal text or confidence/gap sections.

## Minimal example
```yaml
# agent frontmatter — only the servers this agent calls
tools: Read, Write, mcp__cloakbrowser__cloak_launch, mcp__cloakbrowser__cloak_navigate,
  mcp__cloakbrowser__cloak_pdf, mcp__memorygraph__store_memory
skills: stop-slop
```
