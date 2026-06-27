# Pattern: model-tiering

## Rule
Pick the model by the **cognitive load of the job**, not by default. Three tiers:

| Tier | Use for | Examples |
|---|---|---|
| `haiku` | Mechanical, well-specified, low-ambiguity work. | Indexing, changelog appends, simple reformatting, registry bookkeeping. |
| `sonnet` | Structured work with moderate judgment; the fleet default. | Orchestration, web research, content writing, OCR/extraction, rendering. |
| `opus` | Multi-step reasoning, proofs, architecture, hard ambiguity, high cost of error. | Math solving, synthesis under contradiction, scaffolding a new project, authoring agents. |

Exact IDs (update via pattern-curator): `claude-haiku-4-5-20251001`, `claude-sonnet-4-6`,
`claude-opus-4-8`. The newest family also includes `claude-fable-5`.

Verified against Claude Code docs: `model:` accepts `haiku | sonnet | opus | fable`, a full
ID, or `inherit` (the default — the subagent uses the main conversation's model if `model:` is
omitted). State a tier explicitly rather than relying on `inherit`.

## When it applies
Every agent's `model:` field and the main agent's session model. For a full software-development
team, the proven per-role tier mapping lives in [dev-team-roles.md](dev-team-roles.md).

## How
- **Orchestrator**: start on `sonnet`. Escalate the *session* to `opus` only for
  high-complexity synthesis or architecture decisions; routine dispatch stays sonnet.
- **Worker**: set `model:` to the lightest tier that does the job reliably. A worker doing
  symbolic reasoning or proof-quality output earns `opus`; a worker doing a deterministic
  transform takes `haiku` or `sonnet`.
- Record the **reason** for any `opus` worker in the agent file (a one-line "Model tier opus
  is justified: …"). Opus without a stated reason is a smell.

## Anti-patterns
- **Everything on opus** → slow and expensive; most steps don't need it.
- **Everything on haiku** → silent quality collapse on judgment-heavy steps.
- **Tier by vibe** → choosing a tier without naming the cognitive load. State why.

## Minimal example
```
research unit:
  research-planner   sonnet   planning + classification
  web-researcher     sonnet   search + fact gathering
  synthesizer        sonnet   cross-source synthesis
  changelog-writer   haiku    append a dated entry, update an index
main session         sonnet   → opus only for hard synthesis
```
