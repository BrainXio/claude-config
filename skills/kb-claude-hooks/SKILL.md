______________________________________________________________________

## name: claude-hooks description: Reference for the Claude Code hook system — session lifecycle, bootstrap, pre-commit, and memory flush argument-hint: list | hook <name> | source <name>

# Claude Hooks

The `src/claude_hooks` package provides the Claude Code session lifecycle hooks.
These are installable via `uvx` from the public GitHub repository and run
automatically by the Claude Code harness during session events.

## Source

- **Local:** `src/claude_hooks/` (in this repository)
- **Public:** `git+https://github.com/YourOrg/.claude.git`
- **Install:** `uvx --from git+https://github.com/your-org/.claude.git <command>`

When running from the local workspace, hooks use the source at `src/claude_hooks/`.
When installed remotely, they run from the published package.

## Hook Catalog

| Command                | Source File        | Trigger                            | Purpose                                                              |
| ---------------------- | ------------------ | ---------------------------------- | -------------------------------------------------------------------- |
| `claude-bootstrap`     | `bootstrap.py`     | SessionStart hook                  | Detect role, VRAM, profile; pull models; sync docs; write state.json |
| `claude-session-start` | `session_start.py` | SessionStart hook                  | Inject mode, KB index, doc stats, daily log into session context     |
| `claude-session-end`   | `session_end.py`   | SessionEnd hook                    | Capture transcript, spawn flush process for memory extraction        |
| `claude-pre-compact`   | `pre_compact.py`   | PreCompact hook                    | Capture transcript before auto-compaction, spawn flush process       |
| `claude-flush`         | `flush.py`         | Spawned by session-end/pre-compact | Write conversation context to daily log, trigger KB compilation      |
| `claude-pre-commit`    | `pre_commit.py`    | Git pre-commit                     | Run ruff format, ruff check --fix, mdformat, mypy --strict           |

## Actions

| Action   | Argument | Description                                           |
| -------- | -------- | ----------------------------------------------------- |
| `list`   | —        | List all hooks with trigger, source file, and purpose |
| `hook`   | `<name>` | Show detailed description of a specific hook          |
| `source` | `<name>` | Show the local source file path and public GitHub ref |

## Hook Details

### claude-bootstrap

One-command environment setup. Runs at session start:

1. Detect auth provider (ollama vs anthropic from `ANTHROPIC_AUTH_TOKEN`)
1. Detect hardware role from `nvidia-smi` VRAM (worker < 8GB, helper >= 8GB, trainer >= 24GB)
1. Check ollama cloud availability via `ollama signin`
1. Select profile template from `profiles/` (by role + source + VRAM)
1. Pull non-cloud models via `ollama pull`
1. Inspect models via `ollama show` (context length, quantization, capabilities)
1. Merge template env into `~/.claude/settings.local.json`
1. Install git hooks path: `git config core.hooksPath .githooks`
1. Sync `.agents/docs` to `~/.your-org/docs/`
1. Persist state to `~/.your-org/data/state.json`

### claude-session-start

Injects operational context into every session:

- Mode string (Native/Ollama-Cloud/Ollama-Hardware/Ollama-Hybrid)
- Today's date
- Knowledge Base Index (from `~/.your-org/data/knowledge/index.md`)
- Documentation stats (file count, estimated tokens, context window %)
- Recent daily log (last 30 lines of today's or yesterday's log)

Output is injected as `additionalContext` in the SessionStart hook response.

### claude-session-end

Captures conversation transcript on session end:

1. Parse transcript from hook input
1. Extract conversation context and count turns
1. Skip if fewer than 3 turns (nothing substantial to extract)
1. Write context to temp file in `~/.your-org/data/reports/tmp/`
1. Spawn detached `claude-flush` process to persist to daily log

### claude-pre-compact

Same as session-end but fires before context auto-compaction:

- Minimum 10 turns before flushing (higher threshold than session-end)
- Writes context to `flush-context-{session}-{timestamp}.md`
- Spawns detached `claude-flush` process

### claude-flush

Memory extraction agent. Writes conversation context to daily logs:

1. Check cooldown (60s between flushes to prevent rapid cycles)
1. Read context file and deduplicate by SHA-256 hash per session
1. Chunk context to fit 80% of model context window
1. Append to daily log at `~/.your-org/data/daily/{date}.md`
1. Trigger KB compilation if `your-org-tools` package is available

### claude-pre-commit

Pre-commit validation with auto-fix. Runs sequentially:

1. `uv run ruff format src/` — auto-fix formatting
1. `uv run ruff check --fix src/` — auto-fix lint issues
1. `uv run mdformat .` — auto-fix markdown formatting
1. `uv run mypy src/claude_hooks/ --strict` — block on type errors

Exits 0 only if all checks pass (ruff and mdformat auto-fix, mypy blocks).

## Configuration

Hooks are configured in `settings.json`:

```json
{
  "hooks": {
    "SessionStart": "uvx --from git+https://github.com/your-org/.claude.git claude-session-start",
    "PreCompact": "uvx --from git+https://github.com/your-org/.claude.git claude-pre-compact",
    "SessionEnd": "uvx --from git+https://github.com/your-org/.claude.git claude-session-end"
  }
}
```

## Data Flow

```
SessionStart → bootstrap.py + session_start.py → inject context
     │
PreCompact → pre_compact.py → spawn flush.py → daily log
     │                                          │
SessionEnd → session_end.py → spawn flush.py → daily log → KB compile
     │
SessionEnd hook → state.json updated
```

## Related

- `docs/PROTOCOLS.md` — Bootstrap sequence and session lifecycle
- `docs/STRATEGY.md` — Development roadmap for hook improvements
- `settings.json` — Hook configuration entries
