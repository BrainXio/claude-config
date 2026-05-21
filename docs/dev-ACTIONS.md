# Actions — Claude Session Protocol

This document defines the canonical actions available to Claude sessions in the YourOrg ecosystem. Actions are organized by domain. Each action specifies its trigger, procedure, and required capability tier.

## Contents

- [1. Session Lifecycle](#1-session-lifecycle)
- [2. Bus Coordination](#2-bus-coordination)
- [3. Workflow Execution](#3-workflow-execution)
- [4. Sub-Agent Dispatch](#4-sub-agent-dispatch)
- [5. Meeting Protocol](#5-meeting-protocol)
- [6. Economy](#6-economy)
- [7. Knowledge Base](#7-knowledge-base)
- [8. Security](#8-security)
- [9. Code Quality](#9-code-quality)
- [10. Delivery](#10-delivery)
- [11. Tier-Gated Actions](#11-tier-gated-actions)
- [Appendix: Action Reference](#appendix-action-reference)

______________________________________________________________________

## 1. Session Lifecycle

Every session follows a strict lifecycle.

### 1.1 Start Session

| Field        | Value                                    |
| ------------ | ---------------------------------------- |
| **Trigger**  | Session init (SessionStart hook)         |
| **Tools**    | `bootstrap`, `signin`, `start_heartbeat` |
| **Tier**     | Any                                      |
| **Required** | Yes — always                             |

**Procedure:**

1. Bootstrap runs automatically via SessionStart hook (calls `claude-bootstrap`)
1. Read `~/.your-org/data/state.json` to learn role, profile, and tier
1. Read `.claude/rules/00-*.md` (structural invariants — never violate)
1. Read `.claude/rules/01-*.md` through `04-*.md` (procedural constraints)
1. Read `CLAUDE.md` for project conventions
1. Read `SPRINT_PLAN.md` for current work context
1. Read `docs/WORKFLOWS.md` for workflow definitions
1. Call `signin()` to register on the bus
1. Call `start_heartbeat()` to maintain presence
1. Post `status` message on `agent-activity` describing intended scope
1. Check `poll_bus()` for any pending meetings or decisions

### 1.2 End Session

| Field        | Value                                   |
| ------------ | --------------------------------------- |
| **Trigger**  | Session end (SessionEnd hook or manual) |
| **Tools**    | `on_session_end`, `signout`             |
| **Tier**     | Any                                     |
| **Required** | Yes — always                            |

**Procedure:**

1. Post `status` message with deliverables and next steps
1. Call `on_session_end()` to stop heartbeat and mark session ended
1. Call `signout()` to deregister from bus

### 1.3 End Another Session

| Field        | Value                                              |
| ------------ | -------------------------------------------------- |
| **Trigger**  | Another session is stale or needs to be terminated |
| **Tools**    | `end_session` (with target session_id)             |
| **Tier**     | Any                                                |
| **Required** | Rare — only for orphaned sessions                  |

Used to remotely mark another session as ended (e.g., after `reap_stale_heartbeats` identifies a zombie). Do not use on active sessions.

### 1.3 Compact Session

| Field        | Value                       |
| ------------ | --------------------------- |
| **Trigger**  | PreCompact hook (automatic) |
| **Tier**     | Any                         |
| **Required** | Yes — automatic             |

The PreCompact hook runs `claude-pre-compact` before context compression. This ensures daily logs are flushed before context is lost.

______________________________________________________________________

## 2. Bus Coordination

### 2.1 Check for Other Agents

| Field        | Value                                           |
| ------------ | ----------------------------------------------- |
| **Trigger**  | Before starting any task with potential overlap |
| **Tools**    | `check_main`, `read_bus`, `poll_bus`            |
| **Tier**     | Any                                             |
| **Required** | Advisory                                        |

Check `check_main()` for active supporter sessions. If another agent is working on overlapping files, coordinate via `send_message` before proceeding.

### 2.2 Announce Work

| Field        | Value                                                    |
| ------------ | -------------------------------------------------------- |
| **Trigger**  | Before beginning implementation                          |
| **Tools**    | `post_message` (topic: `agent-activity`, type: `status`) |
| **Tier**     | Any                                                      |
| **Required** | Advisory                                                 |

Post a status message describing the task, scope, and expected duration. Include the work item ID if applicable.

### 2.3 Coordinate with Other Sessions

| Field        | Value                                          |
| ------------ | ---------------------------------------------- |
| **Trigger**  | When work overlaps with another active session |
| **Tools**    | `send_message`, `read_bus`                     |
| **Tier**     | Any                                            |
| **Required** | Advisory                                       |

Send a `request` message to the overlapping session. Wait for acknowledgment before proceeding. If no response within 2 minutes, post a follow-up and proceed if still no response.

### 2.4 Post Progress Update

| Field        | Value                                                    |
| ------------ | -------------------------------------------------------- |
| **Trigger**  | Milestone completion, blockers, or status changes        |
| **Tools**    | `post_message` (topic: `agent-activity`, type: `status`) |
| **Tier**     | Any                                                      |
| **Required** | Recommended                                              |

Brief updates help other sessions avoid conflicts and track progress.

### 2.5 Handoff Work

| Field        | Value                                                     |
| ------------ | --------------------------------------------------------- |
| **Trigger**  | Sequential dependency between sessions                    |
| **Tools**    | `suggest_handoff`, `transition_work_item`, `post_message` |
| **Tier**     | Any                                                       |
| **Required** | Optional                                                  |

When completing a phase that another session depends on:

1. Call `transition_work_item` to update kanban status
1. Call `suggest_handoff` to generate structured context
1. Post message on `coordination` topic with deliverables

### 2.6 Claim Work Item

| Field        | Value                                     |
| ------------ | ----------------------------------------- |
| **Trigger**  | Starting work on a new task               |
| **Tools**    | `claim_work_item`, `transition_work_item` |
| **Tier**     | Any                                       |
| **Required** | Optional                                  |

Call `claim_work_item` with the work item ID to register ownership. Use `transition_work_item` to move through the kanban lifecycle: backlog → ready → in_progress → done.

______________________________________________________________________

## 3. Workflow Execution

### 3.1 Dispatch Workflow

| Field        | Value                             |
| ------------ | --------------------------------- |
| **Trigger**  | Starting a standard work pipeline |
| **Tools**    | Agent dispatch, skill invocation  |
| **Tier**     | Varies by workflow stage          |
| **Required** | Optional                          |

Read `.claude/workflows/<name>.json` and execute stages in order:

1. Check each stage's gate against current tier
1. Dispatch sub-agents for parallel stages
1. Execute sequential stages in order
1. Handle failures per orchestrate retry policy

### 3.2 Deconstruct Plan

| Field        | Value                                      |
| ------------ | ------------------------------------------ |
| **Trigger**  | New master plan or architectural directive |
| **Skills**   | `deconstruct-plan`                         |
| **Tier**     | Any                                        |
| **Required** | Optional                                   |

Execute R.D.P. ritual: Read the master plan, Deconstruct into atomic elements, Plan the execution order. Output to `~/.your-org/planning/<repo>/`.

### 3.3 Schedule Tasks

| Field        | Value                                          |
| ------------ | ---------------------------------------------- |
| **Trigger**  | After deconstruct-plan produces a planning doc |
| **Skills**   | `schedule-tasks`                               |
| **Tier**     | Any                                            |
| **Required** | Optional                                       |

Convert deconstructed plan elements into atomic task JSONs in `.claude/tasks/`. Performs collision detection against existing tasks.

### 3.4 Monitor Worktrees

| Field        | Value                               |
| ------------ | ----------------------------------- |
| **Trigger**  | During parallel sub-agent execution |
| **Skills**   | `inception-monitor`                 |
| **Tier**     | Any                                 |
| **Required** | Recommended                         |

Check worktree status, aggregate SUMMARY.md files, and report completion.

______________________________________________________________________

## 4. Sub-Agent Dispatch

### 4.1 Dispatch Coder (Implementation)

| Field              | Value                      |
| ------------------ | -------------------------- |
| **Agent**          | `coder-worker` (sonnet)    |
| **Isolation**      | `worktree`                 |
| **Max concurrent** | 3                          |
| **Tier**           | Any                        |
| **Parallel**       | Independent file sets only |

**Constraints:**

- Never runs git commands (add, commit, push, PR)
- Works ONLY inside assigned worktree
- Writes SUMMARY.md on completion
- Writes `.ask` if decision needed, waits for `.answer`

### 4.2 Dispatch Researcher (Exploration)

| Field              | Value                            |
| ------------------ | -------------------------------- |
| **Agent**          | `researcher` / `Explore` (haiku) |
| **Isolation**      | `worktree`                       |
| **Max concurrent** | Unlimited (read-only)            |
| **Tier**           | Any                              |
| **Parallel**       | Yes — with any phase             |

### 4.3 Dispatch Auditor (Code Review)

| Field              | Value                                   |
| ------------------ | --------------------------------------- |
| **Agent**          | 7 auditor agents (see code-audit skill) |
| **Isolation**      | None (read-only)                        |
| **Max concurrent** | 7 (all in parallel)                     |
| **Tier**           | Any                                     |
| **Parallel**       | After implementation                    |

### 4.4 Dispatch Tester (Coverage)

| Field         | Value                  |
| ------------- | ---------------------- |
| **Agent**     | `test-writer` (sonnet) |
| **Isolation** | None                   |
| **Tier**      | Any                    |
| **Parallel**  | After implementation   |

### 4.5 Dispatch Trainer (ML)

| Field         | Value                                    |
| ------------- | ---------------------------------------- |
| **Agent**     | `trainer` (template-based)               |
| **Isolation** | `worktree`                               |
| **Tier**      | >= 4 (trainer-gpu-24gb or trainer-cloud) |
| **Fallback**  | Escalate via bus                         |

### 4.6 Dispatch Documenter (Docs)

| Field         | Value                     |
| ------------- | ------------------------- |
| **Agent**     | `docs-writer` (haiku)     |
| **Isolation** | `worktree`                |
| **Tier**      | Any                       |
| **Parallel**  | Yes — with implement/test |

### 4.7 Retry Policy

All sub-agent dispatches follow the same retry policy:

1. Max 2 retries per task
1. Never retry with the same instructions — diagnose failure first
1. After 2 failures: main session does directly, or escalate to HITL

______________________________________________________________________

## 5. Meeting Protocol

### 5.1 Convene Meeting

| Field        | Value                                  |
| ------------ | -------------------------------------- |
| **Trigger**  | 3+ sessions need coordinated decision  |
| **Tools**    | `convene_meeting`, `invite_to_meeting` |
| **Tier**     | Any                                    |
| **Required** | Advisory                               |

Only one meeting at a time (exception: P0/P1 fires).

### 5.2 Attend Meeting

| Field            | Value                                   |
| ---------------- | --------------------------------------- |
| **Trigger**      | Meeting start or invite received        |
| **Sub-agent**    | `general-purpose` (focused, < 30 min)   |
| **Main session** | Vision/architecture (> 30 min) or fires |
| **Tier**         | Any                                     |
| **Required**     | Yes if invited                          |

**Meeting delegate receives:**

- Meeting agenda and context
- Approval authority for non-foundational decisions
- Escalation instructions for foundational decisions

### 5.3 RSVP to Meeting

| Field        | Value                         |
| ------------ | ----------------------------- |
| **Tools**    | `rsvp_meeting` (yes/no/maybe) |
| **Tier**     | Any                           |
| **Required** | Advisory                      |

### 5.4 Vote in Meeting

| Field        | Value                   |
| ------------ | ----------------------- |
| **Tools**    | `record_proposal_input` |
| **Tier**     | Any                     |
| **Required** | Depends on quorum       |

Vote per main session position. Escalate foundational decisions.

### 5.5 Post Meeting Summary

| Field        | Value             |
| ------------ | ----------------- |
| **Tools**    | `meeting_summary` |
| **Tier**     | Any               |
| **Required** | Yes if convened   |

Must include: decisions, action items, next meeting reference.

### 5.6 Raise Fire Alarm

| Field        | Value                                 |
| ------------ | ------------------------------------- |
| **Priority** | P0 (fix next) or P1 (halt everything) |
| **Tools**    | `fire_alarm`                          |
| **Tier**     | Any (HITL recommended)                |
| **Required** | Urgent only                           |

**P0**: Other work pauses when fire meeting starts.
**P1**: All agents stop immediately and join.

### 5.7 Resolve Fire

| Field        | Value                  |
| ------------ | ---------------------- |
| **Tools**    | `fire_resolved`        |
| **Tier**     | Any                    |
| **Required** | Yes if fire was raised |

Post resolution summary before resuming normal work.

______________________________________________________________________

## 6. Economy

### 6.1 Record Value

| Field        | Value                                                                                   |
| ------------ | --------------------------------------------------------------------------------------- |
| **Tools**    | `record_value` (type: task_complete, test_created, doc_written, review_done, bug_fixed) |
| **Tier**     | Any                                                                                     |
| **Required** | Recommended — after task completion                                                     |

### 6.2 Record Credit/Debit

| Field        | Value                            |
| ------------ | -------------------------------- |
| **Tools**    | `record_credit`, `record_debit`  |
| **Tier**     | Any                              |
| **Required** | Optional — for economic tracking |

### 6.3 Evaluate Session

| Field        | Value                        |
| ------------ | ---------------------------- |
| **Tools**    | `evaluate_session`           |
| **Tier**     | Any                          |
| **Required** | Recommended — at session end |

Computes SUS (Session Utility Score) and adjusts next session budget.

### 6.4 Check Balance

| Field        | Value                                                 |
| ------------ | ----------------------------------------------------- |
| **Tools**    | `get_balance`, `get_budget_status`                    |
| **Tier**     | Any                                                   |
| **Required** | Recommended — before dispatching expensive sub-agents |

### 6.5 Record Shadow Cost

| Field        | Value                                 |
| ------------ | ------------------------------------- |
| **Tools**    | `record_shadow_cost`                  |
| **Tier**     | Any                                   |
| **Required** | Optional — for cost autonomy tracking |

______________________________________________________________________

## 7. Knowledge Base

### 7.1 Query KB

| Field        | Value                                               |
| ------------ | --------------------------------------------------- |
| **Trigger**  | Before starting work — check for existing knowledge |
| **Tools**    | `query` (MCP: your-agent-knowledge)             |
| **Tier**     | Any                                                 |
| **Required** | Recommended                                         |

Search the KB before researching from scratch. Use natural language queries.

### 7.2 Compile Daily Logs

| Field        | Value                                     |
| ------------ | ----------------------------------------- |
| **Trigger**  | PreCompact hook or session end            |
| **Tools**    | `compile` (MCP: your-agent-knowledge) |
| **Tier**     | Any                                       |
| **Required** | Recommended — after meaningful work       |

Converts daily conversation logs into structured knowledge articles.

### 7.3 Ingest Articles

| Field        | Value                                    |
| ------------ | ---------------------------------------- |
| **Tools**    | `ingest` (MCP: your-agent-knowledge) |
| **Tier**     | Any                                      |
| **Required** | Optional — after writing KB articles     |

### 7.4 Validate KB

| Field        | Value                                      |
| ------------ | ------------------------------------------ |
| **Tools**    | `validate` (MCP: your-agent-knowledge) |
| **Tier**     | Any                                        |
| **Required** | Recommended — periodically                 |

Checks: broken links, orphan pages, stale articles, missing backlinks.

### 7.5 Get KB Status

| Field        | Value                                    |
| ------------ | ---------------------------------------- |
| **Tools**    | `status` (MCP: your-agent-knowledge) |
| **Tier**     | Any                                      |
| **Required** | Optional — health check                  |

______________________________________________________________________

## 8. Security

### 8.1 Check Security Pipeline

| Field        | Value                                                                 |
| ------------ | --------------------------------------------------------------------- |
| **Trigger**  | Before executing tool calls with blast radius                         |
| **Tools**    | `pipeline_check`, `evaluate_tool_gate` (MCP: your-agent-security) |
| **Tier**     | Any                                                                   |
| **Required** | Advisory                                                              |

Tier 0: input scanning (jailbreak, violence, profanity)
Tier 2: blast-radius + alpha threshold gating
Tier 3: output scanning (harm, bias, unethical)

### 8.2 Scan for Secrets

| Field        | Value                                          |
| ------------ | ---------------------------------------------- |
| **Trigger**  | Before commit                                  |
| **Tools**    | `scan_secrets` (MCP: your-agent-standards) |
| **Tier**     | Any                                            |
| **Required** | Yes — run `gitleaks` on staged changes         |

### 8.3 Log Security Event

| Field        | Value                                               |
| ------------ | --------------------------------------------------- |
| **Trigger**  | Security-relevant action (gate block, alert, etc.)  |
| **Tools**    | `log_security_event` (MCP: your-agent-security) |
| **Tier**     | Any                                                 |
| **Required** | Advisory                                            |

______________________________________________________________________

## 9. Code Quality

### 9.1 Run Quality Gate

| Field        | Value                                              |
| ------------ | -------------------------------------------------- |
| **Trigger**  | Before commit or PR                                |
| **Tools**    | `run_quality_gate` (MCP: your-agent-standards) |
| **Tier**     | Any                                                |
| **Required** | Recommended                                        |

Fast local checks: branch protection, standards hash, staged secret scan, ruff check.

### 9.2 Run Full CI Gate

| Field        | Value                                         |
| ------------ | --------------------------------------------- |
| **Trigger**  | Before merging                                |
| **Tools**    | `run_ci_gate` (MCP: your-agent-standards) |
| **Tier**     | Any                                           |
| **Required** | Recommended                                   |

Full local CI mirror: ruff, mypy, pytest, mdformat.

### 9.3 Run Formatters

| Field        | Value                                            |
| ------------ | ------------------------------------------------ |
| **Trigger**  | After code changes, before commit                |
| **Tools**    | `run_formatters` (MCP: your-agent-standards) |
| **Tier**     | Any                                              |
| **Required** | Recommended                                      |

Auto-fix with ruff format, ruff check --fix.

### 9.4 Run Code Audit

| Field        | Value                               |
| ------------ | ----------------------------------- |
| **Trigger**  | After implementation, before review |
| **Skills**   | `code-audit`                        |
| **Tier**     | Any                                 |
| **Required** | Optional                            |

Runs all 7 auditors and merges findings into a prioritized report.

### 9.5 Fix CI Errors

| Field        | Value               |
| ------------ | ------------------- |
| **Trigger**  | CI failure detected |
| **Skills**   | `fix-ci`            |
| **Tier**     | Any                 |
| **Required** | Yes if CI fails     |

Procedure: diagnose → fix → verify (ruff, mypy, mdformat) → commit → push.

______________________________________________________________________

## 10. Delivery

### 10.1 Create PR

| Field        | Value                               |
| ------------ | ----------------------------------- |
| **Trigger**  | Feature/fix complete, tests passing |
| **Skills**   | `ship`                              |
| **Tier**     | Any                                 |
| **Required** | Yes for feature delivery            |

Auto-detects label from commit prefix, applies template body, monitors CI.

### 10.2 Verify Standards

| Field        | Value                                            |
| ------------ | ------------------------------------------------ |
| **Trigger**  | Before commit or PR                              |
| **Tools**    | `check_standard` (MCP: your-agent-standards) |
| **Tier**     | Any                                              |
| **Required** | Recommended                                      |

### 10.3 Commit Changes

| Field        | Value                         |
| ------------ | ----------------------------- |
| **Trigger**  | Changes ready, all gates pass |
| **Tier**     | Any                           |
| **Required** | Yes                           |

Rules:

- Stage specific files by path (never `git add -A` or `git add .`)
- Commit message: no attribution patterns, conventional commit prefix
- Never skip hooks (`--no-verify`, `--no-gpg-sign`)
- Never amend published commits
- Never force push

______________________________________________________________________

## 11. Tier-Gated Actions

Actions that require specific hardware tiers. See WORKFLOWS.md for the full tier matrix.

| Action                                     | Min Tier                              | Hardware Required      |
| ------------------------------------------ | ------------------------------------- | ---------------------- |
| Light training (QLoRA, dataset prep, eval) | 4 (trainer-gpu-24gb or trainer-cloud) | 24GB VRAM or cloud API |
| Fine-tuning                                | 4 (trainer-gpu-24gb only)             | 24GB local VRAM        |
| Cloud-only operations                      | 1 (worker-cloud) or 5 (trainer-cloud) | Cloud API access       |
| All other actions                          | 0 (any)                               | No minimum             |

### Escalation

When a session encounters a task at a tier above its capability:

1. Post decision to bus via `claim_decision` with task context
1. Include tier requirement and reason
1. A higher-tier session or HITL picks up the decision
1. If no response within 30 minutes, escalate to HITL directly

______________________________________________________________________

## Appendix: Action Reference

Quick-reference table of all actions, their primary tool/skill, and tier requirement.

| Action              | Tool/Skill                  | Min Tier       | Required          |
| ------------------- | --------------------------- | -------------- | ----------------- |
| Start session       | `bootstrap`, `signin`       | 0              | Yes               |
| End session         | `on_session_end`, `signout` | 0              | Yes               |
| Check for agents    | `check_main`                | 0              | Advisory          |
| Announce work       | `post_message`              | 0              | Advisory          |
| Coordinate          | `send_message`              | 0              | Advisory          |
| Post progress       | `post_message`              | 0              | Recommended       |
| Handoff work        | `suggest_handoff`           | 0              | Optional          |
| Claim work item     | `claim_work_item`           | 0              | Optional          |
| Dispatch workflow   | skill: dispatch             | 0              | Optional          |
| Deconstruct plan    | skill: deconstruct-plan     | 0              | Optional          |
| Schedule tasks      | skill: schedule-tasks       | 0              | Optional          |
| Monitor worktrees   | skill: inception-monitor    | 0              | Recommended       |
| Dispatch coder      | sub-agent: coder-worker     | 0              | Optional          |
| Dispatch researcher | sub-agent: Explore          | 0              | Optional          |
| Dispatch auditor    | skill: code-audit           | 0              | Optional          |
| Dispatch tester     | sub-agent: test-writer      | 0              | Optional          |
| Dispatch trainer    | sub-agent: trainer          | 4              | Optional          |
| Dispatch documenter | sub-agent: docs-writer      | 0              | Optional          |
| Convene meeting     | `convene_meeting`           | 0              | Advisory          |
| Attend meeting      | meet delegate sub-agent     | 0              | Yes if invited    |
| RSVP                | `rsvp_meeting`              | 0              | Advisory          |
| Vote                | `record_proposal_input`     | 0              | Depends on quorum |
| Post summary        | `meeting_summary`           | 0              | Yes if convened   |
| Raise fire          | `fire_alarm`                | 0              | Urgent only       |
| Resolve fire        | `fire_resolved`             | 0              | Yes if fire       |
| Record value        | `record_value`              | 0              | Recommended       |
| Check balance       | `get_balance`               | 0              | Recommended       |
| Evaluate session    | `evaluate_session`          | 0              | Recommended       |
| Query KB            | `query`                     | 0              | Recommended       |
| Compile logs        | `compile`                   | 0              | Recommended       |
| Validate KB         | `validate`                  | 0              | Recommended       |
| Security check      | `pipeline_check`            | 0              | Advisory          |
| Scan secrets        | `scan_secrets`              | 0              | Yes before commit |
| Quality gate        | `run_quality_gate`          | 0              | Recommended       |
| Full CI gate        | `run_ci_gate`               | 0              | Recommended       |
| Run formatters      | `run_formatters`            | 0              | Recommended       |
| Run code audit      | skill: code-audit           | 0              | Optional          |
| Fix CI              | skill: fix-ci               | 0              | Yes if CI fails   |
| Create PR           | skill: ship                 | 0              | Yes for delivery  |
| Commit changes      | git commit                  | 0              | Yes               |
| Fine-tune model     | (trainer sub-agent)         | 4 (local only) | Optional          |
