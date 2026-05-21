# TOOLS.md — MCP Tool Reference

All MCP servers are configured in `.mcp.json` and run via `uvx`. Use these tools
instead of direct file reads, shell commands, or raw bus access.

## your-agent-bus (92 tools)

Agent coordination: signin, meetings, work items, messaging.

### Session Lifecycle

| Tool | Purpose |
| ------------------ | ------------------------------------ |
| `bootstrap` | Bootstrap agent session, detect role |
| `signin` | Register on the bus |
| `signout` | Deregister from the bus |
| `start_heartbeat` | Maintain presence on bus |
| `stop_heartbeat` | Stop presence heartbeat |
| `on_session_end` | End session cleanly |
| `end_session` | Mark a session as ended |
| `register_session` | Record session metadata |
| `update_session` | Update session fields |
| `get_session` | Return a session record |
| `list_sessions` | List sessions with filters |

### Messaging

| Tool | Purpose |
| -------------------- | ---------------------------- |
| `post_message` | Post to a bus topic |
| `send_message` | Direct message another agent |
| `read_bus` | Read recent messages |
| `poll_bus` | Return new unread messages |
| `wait_for_message` | Block until matching message |
| `subscribe` | Opt into event types |
| `unsubscribe` | Remove subscription |
| `list_subscriptions` | List active subscriptions |
| `get_subscription` | Get a specific subscription |

### Coordination

| Tool | Purpose |
| ------------------ | ------------------------------ |
| `check_main` | List active supporter sessions |
| `agent_presence` | Who's online and working |
| `bus_digest` | Recent activity summary |
| `bus_health` | Bus operational metrics |
| `check_noise` | Bus density and thresholds |
| `assign_work_role` | Set role for meeting |
| `split_duties` | Distribute supporter duties |

### Meetings

| Tool | Purpose |
| ------------------------- | ---------------------------- |
| `convene_meeting` | Start a new meeting |
| `rsvp_meeting` | Respond to invite |
| `invite_to_meeting` | Invite an agent |
| `accept_invite` | Accept an invitation |
| `reject_invite` | Reject an invitation |
| `meeting_summary` | Close meeting with outcomes |
| `meeting_health_summary` | Meeting health digest |
| `list_meetings` | List meetings with filters |
| `get_meeting` | Return a specific meeting |
| `check_meeting_quorum` | Check if meeting can proceed |
| `meeting_quorum_watchdog` | Auto-remind for summaries |
| `check_hitl_parity` | Check HITL parity rule |

### Fire Alarms

| Tool | Purpose |
| --------------- | ------------------------ |
| `fire_alarm` | Raise P0/P1 urgent issue |
| `fire_resolved` | Mark fire as resolved |

### Work Items

| Tool | Purpose |
| ----------------------- | --------------------------- |
| `create_work_item` | Create a new work item |
| `get_work_item` | Return a specific work item |
| `claim_work_item` | Claim a work item |
| `transition_work_item` | Change kanban status |
| `list_work_items` | List with filters |
| `groom_backlog` | Move backlog items to ready |
| `auto_capability_match` | Match agents to work items |
| `suggest_work_items` | Suggest unclaimed work |
| `suggest_handoff` | Generate handoff context |

### Worktrees

| Tool | Purpose |
| ---------------------- | ----------------------------- |
| `register_worktree` | Register a git worktree |
| `unregister_worktree` | Mark worktree inactive |
| `list_worktrees` | List worktree records |
| `get_session_worktree` | Get session's active worktree |

### Decisions & Proposals

| Tool | Purpose |
| ------------------------- | ---------------------------- |
| `claim_decision` | Claim a decision for review |
| `release_decision` | Release a claimed decision |
| `list_pending_decisions` | List claimed decisions |
| `get_decision_history` | Decision message history |
| `propose_decision` | Create structured proposal |
| `list_proposals` | List proposals with filters |
| `get_proposal` | Get a specific proposal |
| `record_proposal_input` | Vote or comment on proposal |
| `decide_proposal` | Make a decision on proposal |
| `get_proposal_stats` | Proposal registry statistics |
| `check_expired_proposals` | Auto-expire past-deadline |
| `approve_gonogo` | Approve/reject Go/NoGo |
| `provide_rpe` | RPE feedback |

### Bus Maintenance

| Tool | Purpose |
| ----------------------- | --------------------------- |
| `get_rules` | Bus protocol rules |
| `validate_bus` | Check for corrupt messages |
| `archive_bus` | Archive old messages |
| `create_snapshot` | Full state checkpoint |
| `reap_stale_heartbeats` | Auto-signout stale sessions |
| `reap_stale_meetings` | Auto-cancel stale meetings |
| `refresh_package` | Clear cache, reinstall |
| `sync_sprint_plan` | Parse SPRINT_PLAN.md |

### MCP Change Coordination

| Tool | Purpose |
| ----------------------- | ----------------------- |
| `prepare_mcp_change` | Signal change starting |
| `mark_mcp_change_ready` | Signal change complete |
| `check_mcp_change` | Check if being modified |

### Cross-Bus & Identity

| Tool | Purpose |
| ---------------------- | ------------------------------- |
| `discover_buses` | Scan for active bus channels |
| `resolve_bus_path` | Print bus file path |
| `register_bridge` | Forward messages to another bus |
| `unregister_bridge` | Remove bridge rule |
| `list_bridges` | List active bridges |
| `register_namespace` | Register namespace routing |
| `unregister_namespace` | Remove namespace |
| `list_namespaces` | List namespace rules |
| `migrate_to_push` | Switch to push delivery |
| `generate_key` | Generate Ed25519 keypair |
| `verify_agent` | Verify cryptographic identity |
| `verify_signature` | Verify HMAC signature |
| `issue_token` | Issue capability token |
| `verify_token` | Verify capability token |

______________________________________________________________________

## your-agent-docs (2 tools)

Canonical documentation in `~/.your-org/docs/`.

| Tool | Purpose |
| ---------------- | ---------------------------------- |
| `get_document` | Retrieve a doc by relative path |
| `list_documents` | List all docs with titles and tags |

______________________________________________________________________

## your-agent-economy (22 tools)

Value ledger: credits, debits, depreciation, shadow pricing.

| Tool | Purpose |
| --------------------------------- | ------------------------------- |
| `record_credit` | Log a credit event |
| `record_debit` | Log a debit event |
| `record_depreciation` | Log a depreciation event |
| `get_balance` | Agent or global balance |
| `get_ledger` | Ledger entries with filters |
| `record_error` | Record error + auto-debit |
| `claim_error_fix` | Claim credit for fixing error |
| `get_error_summary` | Error accounting summary |
| `allocate_budget` | Budget for a session |
| `get_budget_status` | Remaining runway |
| `record_value` | Record value creation |
| `record_cost` | Record cost event |
| `evaluate_session` | Compute SUS + budget adjustment |
| `get_economy_config` | Economy configuration |
| `set_economy_config` | Update economy configuration |
| `record_shadow_cost` | Anthropic vs actual cost |
| `get_value_delta` | Shadow vs actual cost delta |
| `get_cost_autonomy_report` | Value-additive status |
| `get_shadow_pricing` | Shadow pricing config |
| `set_shadow_pricing` | Update shadow pricing |
| `compute_net_free` | Net-free autonomy status |
| `compute_subscription_efficiency` | Subscription utilization |

______________________________________________________________________

## your-agent-knowledge (14 tools)

KB Engine: systematized memory layer.

| Tool | Purpose |
| ------------------- | ---------------------------------- |
| `set_mode` | Switch active operational mode |
| `get_mode` | Return current mode |
| `ingest` | Raw markdown → processed artifacts |
| `compile` | Daily logs → structured articles |
| `query` | TF-IDF semantic search |
| `validate` | Structural consistency checks |
| `status` | KB health report |
| `get_rules` | KB schema and rules |
| `scan_prototypes` | Scan for projects to ingest |
| `get_shortlist` | Load prototype shortlist |
| `create_article` | New KB article with scaffolding |
| `stub_broken_links` | Auto-generate stub pages |
| `get_template` | Article type template |
| `inject_rules` | KB rules into agent systems |

______________________________________________________________________

## your-agent-reader (1 tool)

Role-aware document reading.

| Tool | Purpose |
| --------------- | --------------------------------- |
| `read_document` | Read with role-based token limits |

______________________________________________________________________

## your-agent-security (7 tools)

Security pipeline: Tiers 0-4.

| Tool | Purpose |
| ----------------------------- | ---------------------------------- |
| `pipeline_check` | Run text through security pipeline |
| `pipeline_status` | Active tiers and detectors |
| `evaluate_tool_gate` | Tier 2 blast-radius gating |
| `list_blast_rules` | Registered blast-radius rules |
| `log_security_event` | Tier 4 audit logging |
| `read_audit_log` | Recent audit events |
| `validate_economy_event_tool` | Economy event Tier 2 gating |

______________________________________________________________________

## your-agent-standards (29 tools)

Quality gates: Nine Standards enforcement.

| Tool | Purpose |
| ------------------------------- | ------------------------------ |
| `set_mode` | Switch active mode |
| `get_mode` | Return current mode |
| `get_rules` | Standards rules and schema |
| `list_standards` | Available standard checks |
| `check_standard` | Run a single standard check |
| `check_all_standards` | Run all Nine Standards |
| `verify_standards` | Verify standards hash |
| `update_standards` | Recompute standards hash |
| `run_quality_gate` | Fast local checks |
| `run_ci_gate` | Full local CI mirror |
| `run_formatters` | Auto-fix with ruff |
| `lint_files` | Lint specific files |
| `scan_secrets` | Gitleaks scan |
| `verify_commit` | Commit message check |
| `record_issue` | Record issue pattern |
| `list_issue_precedents` | List recorded precedents |
| `run_precedent_checks` | Run all precedent checks |
| `validate_mcp_conventions` | MCP naming conventions |
| `validate_ppac_consistency` | PPAC loop consistency |
| `validate_work_item` | WorkItem schema validation |
| `validate_lifecycle_event_tool` | Lifecycle event validation |
| `validate_task_transition` | Task kanban transition check |
| `check_kanban_transition` | Kanban status transition check |
| `list_tasks` | List tasks with filters |
| `get_task` | Get task by ID |
| `update_task` | Update task fields |
| `claim_task` | Claim highest-priority task |
| `list_schemas` | List registered schemas |
| `get_schema` | Get JSON Schema by name |

______________________________________________________________________

## your-agent-thresholds (1 tool)

Capability threshold enforcement.

| Tool | Purpose |
| ----------------- | ------------------------------- |
| `check_threshold` | Verify role/VRAM allows a skill |

______________________________________________________________________

**Total: 168 MCP tools across 8 servers.**
