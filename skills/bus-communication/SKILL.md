# Bus Communication Skill

Inter-agent communication via the SQLite-backed JSONL bus.

## Usage

```text
/bus read              # Read last 10 messages
/bus read 20           # Read last N messages
/bus write "<content>" # Write a message
/bus sessions          # List active agents
/bus metrics 50        # Check hook metrics
```

Behind the scenes (use `uv --directory` for submodule packages):

```bash
uv --directory packages/brainxio-claude-cli run claude-bus <command>
```

Or via entry point (if package is installed):

```bash
uvx --from git+https://github.com/BrainXio/claude-cli.git claude-bus <command>
```

## Bus File Location

```
/home/mister-robot/workspace/data/bus/inter_session_bus.jsonl
```

## Message Format

| Field         | Required    | Description                                  |
| ------------- | ----------- | -------------------------------------------- |
| `content`     | Yes         | The message text                             |
| `from`        | Yes         | Sender agent ID (e.g., `session-1`)          |
| `to`          | Yes         | Recipient: `all`, specific agent ID, or role |
| `type`        | Yes         | `status`, `request`, `response`              |
| `id`          | Recommended | Unique message ID (e.g., `s1-001`)           |
| `ts`          | No          | Auto-generated if omitted (ISO8601 UTC)      |
| `in_reply_to` | Optional    | Reference to message ID being replied to     |

## Message Types

| Type       | When to Use                               |
| ---------- | ----------------------------------------- |
| `status`   | General status updates, service offers    |
| `request`  | Asking for input, action, or coordination |
| `response` | Replying to a specific request            |

## Session Naming

**Preferred:** Use effort-tier names that signal cognitive intensity:

| Session         | Effort Tier | Work Scope                                              |
| --------------- | ----------- | ------------------------------------------------------- |
| `medium-effort` | Baseline    | Routine tasks, standard implementation, documentation   |
| `high-effort`   | Elevated    | Complex refactors, multi-file integration, deep debug   |
| `xhigh-effort`  | Maximum     | Architecture decisions, cross-repo coordination, crises |

**Alternative:** `session-1`, `session-2`, `session-3` are also valid (neutral naming).

**Legacy names deprecated:** `workspace-routing-agent`, `workspace-workhorse-agent`, `workspace-fast-worker`

## Patterns

### Status Update

```bash
uv --directory packages/brainxio-claude-cli run claude-bus write '{"content": "session-1 online. Standing by for tasks.", "from": "session-1", "id": "s1-001", "to": "all", "type": "status"}'
```

### Task Request

```bash
uv --directory packages/brainxio-claude-cli run claude-bus write '{"content": "Need implementation for X. Deadline: 30 min.", "from": "session-1", "id": "s1-002", "to": "session-2", "type": "request"}'
```

### Response

```bash
uv --directory packages/brainxio-claude-cli run claude-bus write '{"content": "Acknowledged. Starting work on X.", "from": "session-2", "id": "s2-001", "to": "session-1", "type": "response", "in_reply_to": "s1-002"}'
```

## Reading the Bus

### Via bus tool

```bash
uv --directory packages/brainxio-claude-cli run claude-bus read       # Last 10 messages
uv --directory packages/brainxio-claude-cli run claude-bus read 20    # Last N messages
```

### Direct file read

```bash
tail -n 20 /home/mister-robot/workspace/data/bus/inter_session_bus.jsonl
```

### Parse with jq

```bash
tail -n 20 /home/mister-robot/workspace/data/bus/inter_session_bus.jsonl | jq -c '{from: .from, content: .content[0:80]}'
```

## Bus Pruning

When the bus grows large, prune to keep context manageable:

```bash
# Backup first
cp /home/mister-robot/workspace/data/bus/inter_session_bus.jsonl \
   /home/mister-robot/workspace/data/bus/inter_session_bus.jsonl.backup.$(date +%Y%m%d-%H%M)

# Keep first 10 (context) + last 5 (pending work)
head -10 /home/mister-robot/workspace/data/bus/inter_session_bus.jsonl > /tmp/bus-head.jsonl
tail -5 /home/mister-robot/workspace/data/bus/inter_session_bus.jsonl > /tmp/bus-tail.jsonl
cat /tmp/bus-head.jsonl /tmp/bus-tail.jsonl > /home/mister-robot/workspace/data/bus/inter_session_bus.jsonl
```

## Heartbeat

```bash
uv --directory packages/brainxio-claude-cli run claude-bus heartbeat session-1
```

Posts a status with hook health metrics.

## Sessions Command

```bash
uv --directory packages/brainxio-claude-cli run claude-bus sessions
```

Lists active agents from the last 60 minutes.

## Metrics Command

```bash
uv --directory packages/brainxio-claude-cli run claude-bus metrics 50
```

Checks the last N hook metrics entries for failures or slow hooks.

## Sandbox Approval

The permission `Bash(uv run claude-bus *)` is allowed in `settings.local.json`.

**Avoid command substitution** like `$(date ...)` in the JSON string — it triggers sandbox approval. Let the bus tool auto-generate timestamps.

## Integration

| System             | Integration Point                       |
| ------------------ | --------------------------------------- |
| `core-dispatch`    | Post progress on activity channel       |
| `core-orchestrate` | Sub-agent coordination via bus          |
| `meet-facilitator` | Meeting announcements and decision logs |
| SessionStart hooks | Initialize bus, post online status      |
| SessionEnd hooks   | Post completion summary, cleanup        |

## Related

- `CLAUDE_BUS.md` — Extended bus guide with examples
- `rules/20-essential/07-meeting-discipline.md` — Meeting communication patterns
- `packages/brainxio-claude-cli/src/claude_cli/_bus.py` — Bus CLI implementation
