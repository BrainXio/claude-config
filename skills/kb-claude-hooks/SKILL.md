# Claude Hooks Skill

The `claude-cli` package provides the Claude Code session lifecycle hooks.
These are configured in `settings.hooks.json` and run automatically by the
Claude Code harness during session events.

## Configuration

Hooks are configured in `settings.hooks.json`:

- **SessionStart:** `uvx --from git+https://github.com/BrainXio/claude-cli.git claude-bootstrap`, `claude-session-start`, `claude-check-model-vision`
- **PreCompact:** `uvx --from git+https://github.com/BrainXio/claude-cli.git claude-pre-compact`
- **SessionEnd:** `uvx --from git+https://github.com/BrainXio/claude-cli.git claude-session-end`
- **PreToolUse:** `uvx --from git+https://github.com/BrainXio/claude-cli.git claude-standards-guard`

When configured, hooks run from the published `claude-cli` package.

## Hook Catalog

| Command | Trigger | Purpose |
| --- | --- | --- |
| `claude-bootstrap` | SessionStart hook | Detect role, VRAM, profile; pull models; sync docs; write state |
| `claude-session-start` | SessionStart hook | Inject mode, KB index, doc stats, daily log into session context |
| `claude-session-end` | SessionEnd hook | Capture transcript, spawn flush process for memory extraction |
| `claude-pre-compact` | PreCompact hook | Capture transcript before auto-compaction, spawn flush process |
| `claude-flush` | Spawned by session-end/pre-compact | Write conversation context to daily log, trigger KB compilation |

## Hook Details

### claude-bootstrap

One-command environment setup. Runs at session start:

1. Detect auth provider (ollama vs anthropic from `ANTHROPIC_AUTH_TOKEN`)
1. Detect hardware role from `nvidia-smi` VRAM (worker < 8GB, helper >= 8GB, trainer >= 24GB)
1. Check ollama cloud availability via `ollama signin`
1. Select profile template from profiles (by role + source + VRAM)
1. Pull non-cloud models via `ollama pull`
1. Inspect models via `ollama show` (context length, quantization, capabilities)
1. Merge template env into `~/.claude/settings.local.json`
1. Install git hooks path: `git config core.hooksPath .githooks`
1. Sync `.agents/docs` to `~/.claude/docs/`
1. Persist state to `~/.claude/data/state.json`

### claude-session-start

Injects operational context into every session:

- Mode string (Native/Ollama-Cloud/Ollama-Hardware/Ollama-Hybrid)
- Today's date
- Knowledge Base Index (from `~/.claude/data/knowledge/index.md`)
- Documentation stats (file count, estimated tokens, context window %)
- Recent daily log (last 30 lines of today's or yesterday's log)

Output is injected as `additionalContext` in the SessionStart hook response.

### claude-session-end

Captures conversation transcript on session end:

1. Parse transcript from hook input
1. Extract conversation context and count turns
1. Skip if fewer than 3 turns (nothing substantial to extract)
1. Write context to temp file in `~/.claude/data/reports/tmp/`
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
1. Append to daily log at `~/.claude/data/daily/{date}.md`
1. Trigger KB compilation if `claude-cli` package is available

## Data Flow

```bash
SessionStart → bootstrap + session-start → inject context
     │
PreCompact → pre-compact → spawn flush → daily log
     │                                       │
SessionEnd → session-end → spawn flush → daily log → KB compile
     │
SessionEnd hook → state updated
```

## Related

- `settings.hooks.json` — Hook configuration entries
