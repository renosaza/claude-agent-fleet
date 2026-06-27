# Pattern: permissions-safety

## Rule
Every project carries a `.claude/settings.local.json` with three lists: **`allow`** (run
without prompting), **`ask`** (prompt first), **`deny`** (never). Grant the minimum a project
needs; gate outward-facing or hard-to-reverse actions behind `ask`; hard-block destructive
commands in `deny`.

## When it applies
At project creation, and whenever a project's capability surface changes.

## How
- **`allow`** — the project's routine, reversible operations: reading its own tree, writing to
  its own work/output dirs, the specific MCP tools its agents use, safe shell (`ls`, `cat`,
  `mkdir -p`, `grep`).
- **`ask`** — anything outward-facing or destructive-but-sometimes-needed: `git push`, `gh pr
  create`/`merge`, `rm`, `mv /*`, writes **outside** the project's own root.
- **`deny`** — irreversible damage with no legitimate use here: `rm -rf *`, `sudo rm`,
  `git push --force`, `git reset --hard`, `git clean -fd`, history rewrites, `--no-verify`,
  `dd`, `mkfs`, recursive `chmod`/`chown`.

### Filesystem scope
Scope `Read`/`Write`/`Edit` to absolute path globs. A normal project writes only within its
own root. A privileged project (e.g. `proj_arch`, which generates and edits other projects)
may read broadly but should **`ask`** before writing into another project's tree or into
`~/.claude/`.

### Tool-projects with an external target repo
A **tool-project** is an agent-project whose job is to write/change/ship code that lives in a
repository **outside its own tree** (e.g. `dev-team` operating on a user's codebase). The
project still owns only its own root; the target is foreign. The durable rule:
- **Default-deny / `ask` on every write outside the project's own root.** A tool-project never
  silently writes into the target repo — writes/edits there go through the `ask` gate.
- **The target repo path is supplied per task** by the user or the orchestrator. Never
  hard-code a target path in canon, settings, or an agent file; the project must work against
  any repo the user points it at.
- **Read access to the target is fine** (`allow` `Read`/`grep`/`ls` over the supplied path) —
  exploration and review need it. Only mutation is gated.
- **Never auto-commit or auto-push to a target repo.** `git commit`/`push`/`gh pr` inside a
  target repo require explicit per-task user confirmation (`ask`), even when the project's own
  pushes are allowed. Shipping someone else's repo is the highest-impact action here.
- **The orchestrator owns the hand-off.** Only the orchestrator receives and passes the target
  repo path into a worker's brief; workers never discover or choose the target themselves (see
  [orchestrator-workers.md](orchestrator-workers.md)). A worker writes only the scoped target it
  was handed and flags anything outside it back to the orchestrator.

## Anti-patterns
- **`Bash(*)` in allow with an empty deny** → one bad command from total loss. Always pair
  broad allow with the destructive-command deny list.
- **Writing outside the project root silently** → cross-project writes belong in `ask`.
- **Editing `~/.claude/` without a prompt** → settings changes are high-impact; gate them.

## Minimal example (privileged generator project)
```jsonc
{
  "permissions": {
    "allow": [
      "Bash(mkdir -p *)", "Bash(ls *)", "Bash(cp *)", "Bash(grep *)",
      "Read(/Users/me/.claude/**)", "Read(/Users/me/Documents/claude/**)",
      "Write(/Users/me/Documents/claude/proj_arch/**)",
      "mcp__memorygraph__search_memories", "mcp__memorygraph__store_memory"
    ],
    "ask": [
      "Write(/Users/me/.claude/**)",            // editing Claude settings
      "Edit(/Users/me/Documents/claude/rnd/**)", // editing another project
      "Bash(rm *)", "Bash(git push*)"
    ],
    "deny": [
      "Bash(rm -rf *)", "Bash(sudo rm *)", "Bash(git push --force*)",
      "Bash(git reset --hard *)", "Bash(* --no-verify*)"
    ]
  }
}
```
