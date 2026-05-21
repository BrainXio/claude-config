# PROTOCOLS.md — Bootstrap & Initialization

This document defines the protocols for ensuring all MCP tools are properly
bootstrapped and available at session start.

## 1. Session Bootstrap Sequence

Every session follows this exact sequence:

```
Step 1: SessionStart hook (automatic)
  └── claude-bootstrap: detect role, VRAM, profile → persist to state.json
  └── claude-session-start: flush daily log, check onboarding

Step 2: Agent bootstrap (manual — first action)
  └── mcp__your-agent-bus__bootstrap()
  └── Returns role, tier, profile from state.json

Step 3: Bus signin (manual — second action)
  └── mcp__your-agent-bus__signin()
  └── mcp__your-agent-bus__start_heartbeat()

Step 4: Tool availability check (manual — third action)
  └── Verify each MCP server responds
  └── See §2 for the check procedure

Step 5: Context load (manual — fourth action)
  └── Read .claude/rules/00-*.md  (structural invariants)
  └── Read .claude/rules/01-*.md through 04-*.md (procedural)
  └── Read docs/WORKFLOWS.md  (workflow definitions)
  └── Read docs/ACTIONS.md     (session actions)
  └── Read docs/TOOLS.md       (tool reference)
  └── Read SPRINT_PLAN.md (current work)

Step 6: Bus coordination check (manual — fifth action)
  └── mcp__your-agent-bus__check_main()
  └── mcp__your-agent-bus__poll_bus()
  └── Check for pending meetings or decisions
  └── Post status on agent-activity
```

## 2. Tool Availability Check

After bootstrap and signin, verify that all required MCP servers are responsive:

### Required Servers (must respond)

| Server                | Verification        | If Unavailable                              |
| --------------------- | ------------------- | ------------------------------------------- |
| your-agent-bus        | `read_bus(limit=1)` | Cannot coordinate — read-only mode          |
| your-agent-docs       | `list_documents()`  | Fallback: read `~/.your-org/docs/` directly |
| your-agent-standards  | `list_standards()`  | Proceed without quality gates               |
| your-agent-thresholds | `check_threshold()` | Assume minimum tier (worker)                |

### Optional Servers (degraded mode if unavailable)

| Server               | Verification         | If Unavailable         |
| -------------------- | -------------------- | ---------------------- |
| your-agent-economy   | `get_balance()`      | Skip economy tracking  |
| your-agent-knowledge | `status()`           | Skip KB operations     |
| your-agent-security  | `pipeline_status()`  | Skip security pipeline |
| your-agent-reader    | (no-op verification) | Read files directly    |

### Verification Procedure

```
for each server in [required, optional]:
    try:
        response = call_verification_tool()
        mark server AVAILABLE
    except Exception as e:
        mark server UNAVAILABLE
        log reason

if any required server is UNAVAILABLE:
    post status on bus: "Operating in degraded mode, missing: [servers]"
```

## 3. Sub-Agent Initialization

When spawning sub-agents via the Agent tool, the sub-agent inherits:

- **Tier**: Same as main session (never above — per capability-boundaries rule)
- **Profile**: Same profile as main session
- **Tools**: Restricted to the sub-agent's declared tool set
- **Bus**: Sub-agents do NOT sign in to the bus — they communicate via SUMMARY.md
- **Worktree**: Sub-agents with `isolation: worktree` get an isolated checkout

### Sub-agent handoff

The main session provides context via the Agent tool prompt. The sub-agent
writes `SUMMARY.md` on completion. The main session reads it via
`inception-monitor report`.

```
┌─────────────┐     prompt + context     ┌──────────────┐
│  Main       │ ────────────────────────→ │  Sub-agent   │
│  Session    │                           │  (worktree)  │
│             │ ←──────────────────────── │              │
└─────────────┘     SUMMARY.md + files    └──────────────┘
```

## 4. Multi-Session Initiation

When a second Claude session starts in the same workspace:

1. Both sessions run the same bootstrap sequence (§1)
1. Session B checks `check_main()` — sees Session A
1. Session B posts status on `agent-activity`
1. Session A receives notification via `poll_bus()`
1. Sessions coordinate via `send_message` if work overlaps

### Conflict resolution

| Scenario                               | Resolution                                            |
| -------------------------------------- | ----------------------------------------------------- |
| Same file, different tasks             | One session defers, picks different work              |
| Same file, same task                   | Higher-tier session claims, lower-tier assists        |
| Different files                        | Both proceed, post progress periodically              |
| One session idle (15 min no heartbeat) | Reap via `reap_stale_heartbeats`, claim orphaned work |

## 5. HITL Onboarding

HITL (human-in-the-loop) sessions require a completed onboarding protocol
before full participation. Defined in `rules/04-hitl-onboarding.md`.

### Phases

| Phase             | Duration | Gate                               |
| ----------------- | -------- | ---------------------------------- |
| 1. Telegram setup | ~10 min  | Receive test message               |
| 2. Claude config  | ~10 min  | Approve test decision via Telegram |
| 3. Bus protocol   | ~10 min  | Pass bus protocol quiz (100%)      |
| 4. Economy access | ~10 min  | Submit comprehension statement     |

Until `~/.your-org/config/hitl_onboarding_complete.json` exists, HITL operates
in restricted mode: read-only bus, no approvals, no meetings.

## 6. Shutdown Protocol

### Self-shutdown

```
Step 1: Post final status
  └── mcp__your-agent-bus__post_message()
  └── topic: agent-activity, type: status
  └── Include: deliverables, next steps, handoff context

Step 2: Stop heartbeat
  └── mcp__your-agent-bus__stop_heartbeat()

Step 3: End session
  └── mcp__your-agent-bus__on_session_end()

Step 4: Signout
  └── mcp__your-agent-bus__signout()

Step 5: SessionEnd hook (automatic)
  └── claude-session-end: flush daily log, cleanup
```

### Remote session termination

When a session is stale (heartbeat >15 min old) and must be ended by another
session:

```
Step 1: Confirm staleness
  └── mcp__your-agent-bus__reap_stale_heartbeats()
  └── Verify the session is in the reaped list

Step 2: Post notice on bus
  └── mcp__your-agent-bus__post_message()
  └── topic: agent-activity, type: status
  └── "Ending stale session <session_id>"

Step 3: End remotely
  └── mcp__your-agent-bus__end_session(session_id="<id>")
```
