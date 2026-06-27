# Pattern: agent-anatomy

## Rule
A subagent is a single Markdown file at `<project>/.claude/agents/<name>.md`: YAML frontmatter
+ an English prompt body with a fixed section order. One agent = one focused job.

## Frontmatter (required keys)
```yaml
---
name: kebab-case-name              # matches the filename
description: PROACTIVELY ...        # see below — drives auto-dispatch
tools: Read, Write, ...            # minimum set; workers MUST NOT list Agent
model: haiku | sonnet | opus       # per model-tiering.md
skills: stop-slop                  # optional, comma-separated
---
```

- **`description`** starts with `PROACTIVELY` (or `MUST BE USED`) and states *when* to invoke
  the agent, with concrete trigger phrases and the input/output contract. This is what makes
  the orchestrator pick it without being told. Write it for a dispatcher, not a human reader.
- **`tools`** is the **minimum** the agent needs. A worker never lists `Agent` (it is not an
  orchestrator — see [orchestrator-workers.md](orchestrator-workers.md)). Prefer dedicated
  tools (Read/Write/Edit) over `Bash` equivalents.
- **`model`** is chosen by the job, not by habit. See [model-tiering.md](model-tiering.md).

## Body (fixed section order)
1. **Role line** — one sentence: who this agent is and its single responsibility.
2. **Before starting** — confirm inputs; **query memorygraph** for prior lessons
   (see [proactive-memory.md](proactive-memory.md)); stop and report if inputs are missing.
3. **Workflow** — the ReAct loop: numbered, grounded in observed data, with explicit
   self-checks. No narrated fiction — act, observe, decide.
4. **Hard rules** — invariants and prohibitions, including: "You are a worker. Do NOT use the
   `Agent` tool. Orchestration lives in `<project>/CLAUDE.md`." and the agent's file-scope
   limits.
5. **Output** — exact schema / format the agent returns, plus the one-paragraph report it
   sends back to the orchestrator.
6. **Store to memory** — what to persist after the run (see [proactive-memory.md](proactive-memory.md)).

## Anti-patterns
- **Vague description** ("helps with X") → the orchestrator can't route to it. Be specific and
  start with `PROACTIVELY`.
- **Tool sprawl** → granting `Bash(*)` or the full toolset "just in case". Grant the minimum.
- **Multi-job agent** → if the role line needs an "and", split it into two agents.
- **Russian in the file** → agent files are English (deliverables can be Russian). See
  [mcp-and-skills.md](mcp-and-skills.md).
- **Worker with `Agent` in tools** → forbidden; that turns a worker into an orchestrator.

## Minimal example
```yaml
---
name: faq-builder
description: PROACTIVELY invoke when the user asks to build or extend a Q&A knowledge base
  from collected questions. Trigger phrases — "собери FAQ", "build the Q&A base". Input is a
  list of raw questions (+ project context); output is structured Q&A JSON + a short report.
tools: Read, Write, mcp__memorygraph__search_memories, mcp__memorygraph__store_memory
model: sonnet
skills: stop-slop
---
You are the Q&A knowledge-base worker. Your sole job: turn raw questions into a structured,
deduplicated Q&A set.
## Before starting
1. Confirm the input list and project context. If missing, stop and report.
2. Query memorygraph (tags=["<project>","faq"]) for existing entries to avoid duplicates.
## Workflow
...
## Hard rules
- You are a worker. Do NOT use the `Agent` tool.
## Output
...
## Store to memory
- New verified Q&A pairs → store_memory(type="faq", tags=["<project>","faq"]).
```
