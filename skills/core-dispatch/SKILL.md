# Dispatch Skill

Workflow executor. Reads workflow definitions from
`claude-workflows/<name>.json`, dispatches stages in dependency order with parallelism support.

## Usage

```text
/dispatch feature-implement
/dispatch fix-bug
/dispatch train-model
/dispatch document-only
/dispatch feature-implement --stage implement
/dispatch fix-bug --dry-run
```

Behind the scenes, the skill runs:

```bash
python3 .claude/skills/dispatch/dispatch.py <workflow> [--dry-run] [--stage <name>]
```

The script produces the execution plan; the skill then dispatches sub-agents
according to that plan.

## Procedure

### 0. Generate Plan (via dispatch.py)

Run the workflow engine to load the workflow and produce an execution plan:

```bash
python3 .claude/skills/dispatch/dispatch.py <workflow-name> [--dry-run] [--stage <name>] [--json]
```

| Flag        | Purpose                                             |
| ----------- | --------------------------------------------------- |
| `--dry-run` | Show which stages would execute without dispatching |
| `--stage`   | Execute a single stage only                         |
| `--json`    | Output raw JSON plan for programmatic use           |

The script reads `claude-workflows/<name>.json` and outputs the dispatch plan.
Use this as the source of truth for all subsequent steps.

### 1. Load Workflow

Read `claude-workflows/<name>.json`. Validate against the expected schema.

### 2. Evaluate Gates

For each stage, check `gate`:

| Gate          | Action                                      |
| ------------- | ------------------------------------------- |
| `"always"`    | Proceed unconditionally                     |
| `"tier >= N"` | Proceed if current tier >= N, else fallback |

When gate fails:

- `"fallback": "escalate"` — escalate to higher-capability agent
- `"fallback": "skip"` — log and move to next stage
- No fallback defined — raise warning and skip stage

### 4. Dispatch Stages

Respect dependency order and parallelism. Use the output from `dispatch.py` to
determine the exact sequence:

```bash
python3 .claude/skills/dispatch/dispatch.py <workflow-name> --json
```

**Sequential stages** — execute one at a time, wait for completion:

1. Read the template for the stage's agent from
   `claude-config/skills/orchestrate/templates/<agent>.md`
1. Substitute placeholders (`{FILE_PATHS}`, `{CHANGE_SPEC}`, etc.) with
   context from the current task
1. Dispatch via the `Agent` tool with the appropriate `subagent_type`
1. Wait for the sub-agent to return
1. Read `SUMMARY.md` or output files for results

**Parallel stages** — dispatch up to `max_concurrent` sub-agents simultaneously:

1. Group stages by dependency batch (all dependencies satisfied)
1. Launch multiple `Agent` calls in a single message for parallelism
1. Each sub-agent gets appropriate isolation to avoid file conflicts

**Agent type mapping** (from workflow JSON `agent` field):

| Workflow agent | `subagent_type`            | Isolation |
| -------------- | -------------------------- | --------- |
| `Explore`      | `Explore`                  | none      |
| `Plan`         | `Plan`                     | none      |
| `coder-worker` | `coder-worker`             | worktree  |
| `test-writer`  | `test-writer`              | none      |
| `bug-hunter`   | `bug-hunter`               | none      |
| `docs-writer`  | `docs-writer`              | worktree  |
| `code-audit`   | multi-audit (7 sub-agents) | none      |
| `trainer`      | `general-purpose`          | worktree  |
| `ship-skill`   | `general-purpose`          | none      |

**Post progress** — update activity channel after each stage completion with status and deliverables.

### 5. Track Completion

Each stage reports:

- Status (completed, failed, skipped, escalated)
- Output location (worktree path, SUMMARY.md, files modified)

### 6. Handle Failure

Per retry policy from workflow JSON:

1. First failure: retry with refined instructions
1. Second failure: main session does directly
1. Escalation: post to bus `coordination` topic with full context

## Dry-run Mode

`--dry-run` reports which stages would execute, which are gated, and which
would escalate — without dispatching any sub-agents. Output:

```text
Workflow: feature-implement (5 stages)
Configuration: from workflow

Stage          Gate        Status
──────         ────        ──────
research       always      will execute
plan           always      will execute
implement      always      will execute (parallel, 3 workers)
audit          always      will execute (7 sub-agents)
test           always      will execute
docs           always      will execute
ship           always      will execute
```

## Stage-Only Mode

`--stage <name>` executes a single stage from the workflow. Useful for
re-running a failed stage without restarting the entire pipeline.

## Integration

| System              | Integration Point                                                 |
| ------------------- | ----------------------------------------------------------------- |
| `orchestrate` skill | Calls dispatch for workflow loading and gate checking             |
| `bus`               | Post progress on activity channel                                 |
| `dispatch.py`       | Executable plan generator (reads workflow, outputs JSON or table) |
| `claude-workflows`  | Workflow definitions                                              |

## Related

- `claude-workflows/*.json` — Workflow definitions
- `orchestrate` skill — Higher-level orchestration that calls dispatch
