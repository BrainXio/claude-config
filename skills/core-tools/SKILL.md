# Core Tools Skill

All CLI capabilities are bundled in `claude-cli`. To discover available commands:

```bash
claude-cli --help
```

## Entry Points

| Command                  | Purpose                                    |
| ------------------------ | ------------------------------------------ |
| `claude-bootstrap`       | Hardware detection, profile selection      |
| `claude-session-start`   | Inject KB context into session             |
| `claude-session-end`     | Flush transcript, run CI gate              |
| `claude-pre-compact`     | Pre-compaction safety flush                |
| `claude-pre-commit`      | Run formatters, type check, secret scan    |
| `claude-standards-guard` | PreToolUse hook — block forbidden patterns |
| `claude-post-tool-use`   | PostToolUse hook — record tool usage       |
| `claude-dispatch`        | Workflow dispatch engine                   |

## Hooks (Automatic)

These run automatically via `settings.json`. Do not invoke manually unless debugging.

| Hook           | Handler                                     |
| -------------- | ------------------------------------------- |
| `SessionStart` | `claude-bootstrap` + `claude-session-start` |
| `PreToolUse`   | `claude-standards-guard`                    |
| `PostToolUse`  | `claude-post-tool-use`                      |
| `PreCompact`   | `claude-pre-compact`                        |
| `SessionEnd`   | `claude-session-end`                        |

## Knowledge Engine

The `claude_knowledge` package provides in-process KB operations:

| Module   | Import                                              |
| -------- | --------------------------------------------------- |
| Ingest   | `from claude_knowledge.ingest import ingest_dir`    |
| Compile  | `from claude_knowledge.compile import compile_logs` |
| Query    | `from claude_knowledge.query import query_kb`       |
| Validate | `from claude_knowledge.validate import validate_kb` |

## Quality Gate

The `claude_quality` package provides enforcement logic:

| Module      | Import                                                            |
| ----------- | ----------------------------------------------------------------- |
| Gate        | `from claude_quality.gate import run_quality_gate, run_ci_gate`   |
| Anti-gaming | `from claude_quality.anti_gaming import validate_proposal`        |
| Schemas     | `from claude_quality.schemas import WorkItem, WorkLifecycleEvent` |
