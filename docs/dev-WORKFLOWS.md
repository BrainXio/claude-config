# Tier-Aware Workflow & Session Collaboration Framework

## Context

The current system has rich sub-agent definitions (18 agents), skills (13), profiles (6 hardware/subscription tiers), and an orchestrate skill that dispatches work — but there is no **unified, tier-gated workflow engine**.

Key gaps addressed by this framework:

- The `orchestrate` skill dispatches any role regardless of hardware capability
- No conditional branching based on subscription model (cloud vs local GPU)
- No economy-aware scheduling (cost of sub-agent vs value of work)
- Session collaboration patterns are ad-hoc (bus-based, but no structured handoff)
- Training/ML tasks can be dispatched to any session, even those without GPU resources
- 18 sub-agents exist with overlapping concerns but no priority-based dispatch order

______________________________________________________________________

## 1. Tier Matrix — What Each Profile Can Dispatch

Derived from existing profiles in `.claude/profiles/` and the capability-boundaries rule (00-capability-boundaries.md):

```
                    Cloud    Wkr8Gb  Hlp8Gb  Hlp16Gb Trn24Gb TrnCloud
                    ─────    ──────  ──────  ─────── ─────── ─────────
Implement           ✅       ✅      ✅      ✅      ✅      ✅
Test (analysis)     ✅       ✅      ✅      ✅      ✅      ✅
Audit (all 7)       ✅       ✅      ✅      ✅      ✅      ✅
Research            ✅       ✅      ✅      ✅      ✅      ✅
Docs                ✅       ✅      ✅      ✅      ✅      ✅
Light training      ❌       ❌      ❌      ❌      ✅      ✅
Fine-tuning         ❌       ❌      ❌      ❌      ✅      ❌
Cloud-only ops      ✅       ❌      ❌      ❌      ❌      ✅
```

**Light training** = QLoRA, dataset prep, evaluation (>= 24GB VRAM or cloud API).
**Fine-tuning** = Full fine-tuning (requires local 24GB GPU).
**Cloud-only ops** = API-dependent tasks (deployments, cloud infra).

### Tier Level Mapping

```
0: worker-gpu-8gb    (Worker, no cloud)
1: worker-cloud       (Worker, cloud API)
2: helper-gpu-8gb    (Helper, 8GB local + cloud)
3: helper-gpu-16gb   (Helper, 16GB local)
4: trainer-gpu-24gb  (Trainer, 24GB local)
5: trainer-cloud     (Trainer, cloud API)
```

______________________________________________________________________

## 2. Workflow Pipeline — Tier-Gated Stages

Each stage has a **gate condition** and references existing skills/agents. Stages only execute when their gate passes; otherwise the workflow escalates or skips.

```
Phase 1:  Inception     (gate: always)
Phase 2:  Research      (gate: always)
Phase 3:  Plan          (gate: always)
Phase 4:  Implement     (gate: always)
Phase 5:  Audit         (gate: always)
Phase 6:  Test          (gate: always)
Phase 7:  Train/ML      (gate: tier >= 4)
Phase 8:  Document      (gate: always)
Phase 9:  Review & Ship (gate: always)
Phase 10: Retrospective (gate: always)
```

### Phase Details

#### Phase 1: Inception

- **Gate**: always passes (any tier)
- **Skills**: `deconstruct-plan` (R.D.P. ritual), `schedule-tasks`
- **Output**: `~/.your-org/planning/<repo>/` + `.claude/tasks/<n>.json`

#### Phase 2: Research

- **Gate**: always passes (any tier)
- **Sub-agents**: `Explore` / `researcher` (haiku, read-only)
- **MCP tools**: `your-agent-knowledge query`
- **Parallel**: can run concurrently with Phase 1

#### Phase 3: Plan

- **Gate**: always passes (any tier)
- **Sub-agents**: `Plan` agent (if architectural decisions needed)
- **Output**: Implementation plan with file paths, dependency order, risk assessment

#### Phase 4: Implement

- **Gate**: always passes (any tier, but model quality varies by profile)
- **Sub-agents**: `coder-worker` (sonnet, worktree-isolated)
- **Constraints**: max 3 concurrent, independent file sets only
- **Rules**: implementers never commit/push — main session handles git

#### Phase 5: Audit

- **Gate**: always passes (any tier)
- **Skills**: `code-audit` → runs 7 auditors:
  - `bug-hunter` (sonnet) — logic bugs, race conditions, type errors
  - `complexity-reducer` (sonnet) — high cyclomatic complexity
  - `dead-code-hunter` (haiku) — unused functions, variables, configs
  - `dry-enforcer` (sonnet) — duplicated logic blocks
  - `kiss-auditor` (sonnet) — unnecessarily complex implementations
  - `single-source-auditor` (haiku) — duplicated constants and config values
  - `yagni-auditor` (haiku) — over-engineered code, speculative features
- **Output**: Merged, deduplicated, severity-prioritized report

#### Phase 6: Test

- **Gate**: always passes (any tier — analysis only, no GPU needed)
- **Sub-agents**: `test-writer` (sonnet) — coverage analysis, gap identification
- **Note**: test-writer is analysis-only; use `coder-worker` to write test code

#### Phase 7: Train/ML

- **Gate**: requires trainer-tier profile (tier >= 4: 24GB local or trainer-cloud)
- **Escalation**: if Worker/Helper encounters ML task, escalate via bus `claim_decision`
- **Sub-agents**: (new) `trainer` template — model evaluation, dataset prep, QLoRA
- **Profiles**: `trainer-gpu-24gb` for fine-tuning, `trainer-cloud` for API-based training

#### Phase 8: Document

- **Gate**: always passes (any tier)
- **Sub-agents**: `docs-writer` (haiku, worktree-isolated)
- **MCP tools**: `your-agent-knowledge compile` / `ingest` / `validate`
- **Formatters**: `markdown-formatter` (mdformat)

#### Phase 9: Review & Ship

- **Gate**: always passes (any tier)
- **Review**: `bug-hunter` on changed files (fresh eyes)
- **Skills**: `ship` → PR creation, label mapping, CI monitoring
- **Fallback**: `fix-ci` skill if CI fails

#### Phase 10: Retrospective

- **Gate**: always passes (any tier)
- **MCP tools**: `your-agent-economy` — `record_value`, `evaluate_session`
- **KB**: `your-agent-knowledge compile` — daily logs → structured articles

______________________________________________________________________

## 3. Economy-Aware Dispatch Logic

When deciding whether to dispatch a sub-agent vs. do work directly:

```
Expected value >= cost → dispatch sub-agent
Expected value < cost  → do directly
Unknown value          → use defaults per task type
```

**Default thresholds**:

- Implementation: always dispatch (worktree isolation prevents conflicts)
- Audit: always dispatch (fresh eyes find more issues)
- Research: dispatch if domain is unfamiliar
- Docs: dispatch if multi-file or multi-module

**Sub-agent cost tiers** (relative token cost):

- sonnet-model agent: ~3x haiku
- haiku-model agent: ~1x
- worktree isolation overhead: +20%

**Retry budget** (from orchestrate skill):

- Max 2 retries per sub-agent
- After 2 failures: main session does directly, or escalate to HITL

______________________________________________________________________

## 4. Session Collaboration Patterns

Four patterns for multi-session work, using existing bus infrastructure:

| Pattern             | When                         | Bus Mechanism                              | Example                                    |
| ------------------- | ---------------------------- | ------------------------------------------ | ------------------------------------------ |
| **Fire-and-forget** | Independent parallel work    | worktree + `post_message` status           | Two coder-workers on non-overlapping files |
| **Handoff**         | Sequential dependency        | `transition_work_item` + `suggest_handoff` | Implement → Test                           |
| **Meeting**         | 3+ sessions, decision needed | `convene_meeting` + `rsvp` + vote          | Architecture decision                      |
| **Partnership**     | 2 sessions, non-foundational | `send_message` + coordinate                | Pair debugging                             |

**Bus protocol rules** (from 03-bus-coordination.md):

- **HITL parity**: HITL is N+1 vote at even quorums; majority rules at odd quorums
- **Partnerships**: need HITL for foundational decisions (schema, architecture, cross-cutting concerns)
- **Concurrent meetings**: prohibited (exception: P0/P1 fires)
- **Attendance**: never be late; spawn sub-agent delegate if busy

**Sub-agent delegation for meetings** (from orchestrate skill):

- Meeting delegate receives agenda, positions, approval authority
- Non-foundational decisions: vote per main session position
- Foundational decisions: escalate, do NOT vote

______________________________________________________________________

## 5. Workflow Definitions

Workflows are declarative JSON files in `.claude/workflows/`. Each defines a sequence of stages with agent assignments and tier gates.

### feature-implement.json

The default workflow for new features:

```json
{
  "workflow": "feature-implement",
  "description": "Full feature implementation pipeline: research → implement → audit → test → docs → ship",
  "stages": [
    { "name": "research",  "agent": "Explore",          "gate": "always",    "parallel": false },
    { "name": "plan",      "agent": "Plan",             "gate": "always",    "parallel": false },
    { "name": "implement", "agent": "coder-worker",     "gate": "always",    "parallel": true,  "max_concurrent": 3, "isolation": "worktree" },
    { "name": "audit",     "agent": "code-audit",       "gate": "always",    "parallel": true,  "sub_agents": ["bug-hunter", "complexity-reducer", "dead-code-hunter", "dry-enforcer", "kiss-auditor", "single-source-auditor", "yagni-auditor"] },
    { "name": "test",      "agent": "test-writer",      "gate": "always",    "parallel": false },
    { "name": "docs",      "agent": "docs-writer",      "gate": "always",    "parallel": false },
    { "name": "ship",      "agent": "ship-skill",       "gate": "always",    "parallel": false }
  ]
}
```

### fix-bug.json

Streamlined workflow for bug fixes:

```json
{
  "workflow": "fix-bug",
  "description": "Bug fix pipeline: audit → implement → test → ship",
  "stages": [
    { "name": "audit",     "agent": "bug-hunter",       "gate": "always",    "parallel": false },
    { "name": "implement", "agent": "coder-worker",     "gate": "always",    "parallel": false, "isolation": "worktree" },
    { "name": "test",      "agent": "test-writer",      "gate": "always",    "parallel": false },
    { "name": "ship",      "agent": "ship-skill",       "gate": "always",    "parallel": false }
  ]
}
```

### train-model.json

ML training workflow (tier-gated):

```json
{
  "workflow": "train-model",
  "description": "ML training pipeline: research → prep → train → eval → docs",
  "stages": [
    { "name": "research",  "agent": "Explore",          "gate": "always",    "parallel": false },
    { "name": "prep",      "agent": "coder-worker",     "gate": "always",    "parallel": false, "isolation": "worktree" },
    { "name": "train",     "agent": "trainer",          "gate": "tier >= 4", "parallel": false, "fallback": "escalate" },
    { "name": "eval",      "agent": "trainer",          "gate": "tier >= 4", "parallel": false, "fallback": "escalate" },
    { "name": "docs",      "agent": "docs-writer",      "gate": "always",    "parallel": false }
  ]
}
```

______________________________________________________________________

## 6. Profile-Aware Dispatcher

A new `/dispatch` skill that wraps the orchestrate skill with tier-aware gate checking:

```
/dispatch <workflow-name> [--stage <stage>] [--dry-run]
```

### Procedure

1. **Read workflow** — load `.claude/workflows/<name>.json`
1. **Check profile** — read current tier from `state.json`
1. **Evaluate gates** — for each stage:
   - `"gate": "always"` → proceed
   - `"gate": "tier >= N"` → compare current tier; if insufficient, use fallback
1. **Dispatch stages** — respecting dependency order and parallelism:
   - Sequential stages execute one at a time
   - Parallel stages dispatch up to `max_concurrent` sub-agents simultaneously
1. **Track completion** — each stage reports status on the bus via `post_message`
1. **Handle failure** — follow orchestrate's failure handling:
   - 2 retries with refined instructions → main session direct → escalate to HITL

### Dry-run Mode

`--dry-run` reports which stages would execute, which are gated, and which would escalate — without dispatching any sub-agents.

______________________________________________________________________

## 7. Tier Detection

The bootstrap already detects VRAM via `nvidia-smi` and persists role to `state.json`. Enhancement needed:

- Profile name should be persisted in `state.json` (e.g., `"profile": "helper-gpu-8gb"`)
- Export `CLAUDE_PROFILE_TIER` env var (0-5) for sub-agents to inherit
- Sub-agents inherit the main session's tier (never above — per 00-capability-boundaries.md)

______________________________________________________________________

## 8. Integration Map

| Integration              | What It Does                                          | Existing Artifact                           |
| ------------------------ | ----------------------------------------------------- | ------------------------------------------- |
| `orchestrate` skill      | Reads workflow JSON, dispatches stages with templates | `.claude/skills/orchestrate/SKILL.md`       |
| `code-audit` skill       | Sub-workflow: runs 7 auditors in parallel             | `.claude/skills/code-audit/SKILL.md`        |
| `inception-monitor`      | Tracks worktree status, aggregates SUMMARY.md         | `.claude/skills/inception-monitor/SKILL.md` |
| `deconstruct-plan` skill | Phase 1: master plan → deconstructed plan             | `.claude/skills/deconstruct-plan/SKILL.md`  |
| `schedule-tasks` skill   | Phase 1: plan → atomic task JSONs                     | `.claude/skills/schedule-tasks/SKILL.md`    |
| `ship` skill             | Phase 9: PR creation + CI monitoring                  | `.claude/skills/ship/SKILL.md`              |
| `fix-ci` skill           | Phase 9 fallback: CI diagnosis + fix                  | `.claude/skills/fix-ci/SKILL.md`            |
| 18 sub-agent defs        | Worker units dispatched by workflow stages            | `.claude/agents/*.md`                       |
| 6 profiles               | Gate conditions reference profile tier                | `.claude/profiles/*.json`                   |
| 00-capability-boundaries | Tier inheritance: sub-agents ≤ main tier              | `.claude/rules/00-capability-boundaries.md` |
| 03-bus-coordination      | Session collab: meetings, partnerships, HITL          | `.claude/rules/03-bus-coordination.md`      |
| your-agent-economy       | Economy-aware dispatch costs                          | MCP server (tools)                          |
| your-agent-knowledge     | KB query + compile (Phases 2, 8, 10)                  | MCP server (tools)                          |

______________________________________________________________________

## 9. Files to Create

| File                                       | Purpose                                  |
| ------------------------------------------ | ---------------------------------------- |
| `.claude/workflows/feature-implement.json` | Default full-stack feature workflow      |
| `.claude/workflows/fix-bug.json`           | Bug fix workflow (audit → fix → ship)    |
| `.claude/workflows/train-model.json`       | ML training workflow (trainer-tier gate) |
| `.claude/workflows/document-only.json`     | Documentation-only workflow              |
| `.claude/skills/dispatch/SKILL.md`         | Tier-aware dispatcher skill              |

## 10. Files to Modify

| File                                              | Changes                                                                                                 |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `.claude/skills/orchestrate/SKILL.md`             | Add workflow-reading step, tier gate checks, economy-aware dispatch section, trainer template reference |
| `.claude/skills/orchestrate/templates/trainer.md` | New template for ML/training sub-agents                                                                 |
| `docs/AGENTS.md`                                  | Document workflow pipeline and tier matrix                                                              |
| `CLAUDE.md`                                       | Document workflows directory and dispatch skill                                                         |
