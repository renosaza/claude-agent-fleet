# Pattern: proactive-memory

## Rule
Every agent — orchestrator and worker alike — touches `memorygraph` **proactively**, without
being asked:
- **Before acting**: query the graph for prior lessons relevant to the task.
- **After acting**: store what was learned (decisions, bugs, confirmed/failed patterns).

Memory is not an optional afterthought. An agent that solves a problem already solved last
week, or repeats a recorded mistake, has failed this pattern.

## The MCP tools
Shared SQLite graph at `~/.claude/memory-graph/graph.db`; projects separated by **tag**.

- `mcp__memorygraph__search_memories(tags=[...], query=...)` — topical lookup.
- `mcp__memorygraph__recall_memories(project_path="…/<project>")` — everything for a project.
- `mcp__memorygraph__store_memory(type=..., title=..., content=..., tags=[...], importance=...)`.

Every agent that reads or writes memory lists the corresponding tools in its frontmatter
`tools:` (at minimum `search_memories` + `store_memory`).

## Query-before (required at session/task start)
- **Orchestrator, session start**: `recall_memories(project_path="…/<project>")` AND
  `search_memories(tags=["<project>"])` to recall prior runs, recurring failure modes, and
  user preferences. State briefly what was recalled before planning.
- **Worker, task start**: `search_memories(tags=["<project>","<agent>", <topic-terms>])` for
  lessons specific to this kind of task. Apply recorded anti-patterns.

## Store-after (required after these events — no exceptions)
**This instance's `type` enum is fixed and does NOT include `decision`.** The accepted values
are exactly: `task`, `code_pattern`, `problem`, `solution`, `project`, `technology`, `error`,
`fix`, `command`, `file_context`, `workflow`, `general`, `conversation`. A `store_memory` call
with any other `type` (`decision`, `bug`, `pattern`, `tech_choice`, `feedback`) **fails**.
Encode the *kind* of lesson as a **tag**, not as an invented type. Use this mapping verbatim:

| Event | `type` | Required tag | Key fields (in content) |
|---|---|---|---|
| Bug solved | `fix` | `bug` | root_cause, fix |
| Architecture / curation decision | `code_pattern` (or `project` / `general`) | `decision` | rationale, alternatives_considered |
| Approach failed | `code_pattern` | `antipattern` | why_it_failed |
| Pattern confirmed working | `code_pattern` | `success` | the reusable recipe |
| Technology chosen over alternatives | `technology` | `tech-choice` | chosen, rejected, reason |
| User-stated preference | `general` | `preference` | the preference + why |

Always include `tags=["<project>", …]` **plus the kind tag above**. Cross-tag when a lesson is
fleet-wide (`tags=["<project>","proj_arch"]`). Set `importance` 0.6–0.8 for non-trivial lessons.

> **Note — global CLAUDE.md override.** `~/.claude/CLAUDE.md` still phrases this as `type=Decision`,
> `type=Bug`, `type=Pattern`, `type=TechChoice`. Those are conceptual labels, not valid enum
> values for *this* memorygraph instance; the mapping above overrides them. Newly scaffolded
> agents must inherit the table above, not the global wording.

## Enforcement
- **Text**: this rule is restated in every project `CLAUDE.md` (`## Memory` section) and in
  every agent's "Before starting" and "Store to memory" sections.
- **Hook**: a soft `SessionStart` reminder in `~/.claude/settings.json` prints the
  query-before / store-after contract at the start of each session. It reminds; it does not
  block.

## Anti-patterns
- **Write-only memory** → storing results but never querying first. Both halves are required.
- **Solve-then-forget** → finishing a non-trivial task without storing the lesson.
- **Untagged nodes** → a memory with no project tag is unfindable. Always tag.

## Minimal example
```
# Before
recall_memories(project_path="~/Documents/claude/mathematik")
search_memories(tags=["mathematik","math-solver","ode-bernoulli"])

# After a tricky solve (note: type=code_pattern + "success" tag — NOT type=pattern)
store_memory(
  type="code_pattern",
  title="Bernoulli ODE: substitute v=y^(1-n) before integrating",
  tags=["mathematik","math-solver","ode-bernoulli","success"], importance=0.7)

# After a design call (note: type=code_pattern + "decision" tag — NOT type=decision)
store_memory(
  type="code_pattern",
  title="Chose split reviewer roles over one mega-reviewer",
  tags=["dev-team","architect","decision"], importance=0.7)
```
