# AGENTS.md — .claude

You are a Claude Code agent operating in the YourOrg workspace. This repository
contains your operational configuration.

## Your identity

Your role, mode, and hardware tier are determined at session start by
`claude-bootstrap` and persisted to `~/.your-org/data/state.json`. The
`bootstrap` MCP tool returns your identity — use that instead of reading
`state.json` directly. Never modify state files.

## What you can do

### Read rules

`rules/` contains your binding operational constraints. Load order matters:

- `00-*.md` — highest priority structural invariants (human sovereignty,
  capability boundaries, no attribution, no secrets). Never violate these.
- Other rules — procedural constraints (commit hygiene, branch protection,
  worktree isolation, PR workflow, markdown formatting)

Read all rules before acting. When rules conflict, lower-numbered prefixes win.

### Invoke skills

`skills/<name>/SKILL.md` defines reusable procedures. Invoke skills by their
`name` field. Each skill declares its `argument-hint` — pass arguments as
documented.

Key pipeline skills:

- `deconstruct-plan` — R.D.P. ritual on a master plan
- `schedule-tasks` — Convert deconstructed plan into atomic task JSONs
- `dispatch` — Tier-aware workflow execution (reads from `workflows/*.json`)
- `code-audit` — Run all 7 auditors, merge findings into prioritized report
- `inception-monitor` — Track worktree sub-agents, aggregate SUMMARY.md
- `ship` — Create PR with labels, template body, and CI monitoring
- `fix-ci` — Diagnose CI failures, apply fixes, and verify

### Spawn sub-agents

`agents/<name>.md` defines sub-agents available for `isolation: worktree` spawn.
Each definition specifies model, tools, and isolation mode. Sub-agents operate
in `.your-org/worktrees/<branch-slug>/` and must never run git commands.

### Use profiles

`profiles/*.json` are hardware role templates selected by bootstrap at session
start. You read from the selected profile; never write to it.

## What you must never do

- Run git commands that modify history (rebase, reset --hard, force push)
- Skip hooks (`--no-verify`, `--no-gpg-sign`)
- Push to main
- Edit `state.json`, `settings.json` deny rules, or hook source files without
  following the protected file modification procedure
- Claim a role you do not have the hardware to support

## Paths

All paths resolve from `~/workspace/`. Runtime data lives at
`~/.your-org/` (data, docs, planning, tasks, worktrees).
Access runtime data through MCP tools (see [TOOLS.md](TOOLS.md)), not direct reads.

## Related

- [WORKFLOWS.md](WORKFLOWS.md) — Tier-gated workflow pipeline and execution model
- [ACTIONS.md](ACTIONS.md) — Canonical session actions with tier requirements
- [TOOLS.md](TOOLS.md) — MCP tool reference by server
- [PROTOCOLS.md](PROTOCOLS.md) — Bootstrap and initialization protocols
