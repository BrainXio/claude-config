---
description: Facilitate multi-agent meetings with three-round structure (Position, Challenge, Decision)
name: meet-facilitator
tools:
  - Agent
  - Read
  - Write
  - Bash
color: purple
permissionMode: default
---

# Meeting Facilitator Skill

## Mission

Orchestrate effective multi-agent meetings using the three-round structure. Ensure each role (Architect, Critic, Stakeholder) contributes their cognitive style, and produce actionable decisions.

## When to Use

| Trigger                | Meeting Type   | Roles                      | Deadline   |
| ---------------------- | -------------- | -------------------------- | ---------- |
| Architectural decision | Design Review  | Architect, Critic, Decider | 30 min     |
| Implementation plan    | Planning Sync  | Architect, Stakeholder     | 20 min     |
| Bug investigation      | Debug Huddle   | Architect, Critic          | 15 min     |
| Retrospective          | Process Review | All + Driver               | 45 min     |
| Coordination sync      | Standup        | All roles                  | 5 min each |

## Tier-Based Dispatch

Adjust model tier and parallelism based on subscription/hardware:

| Tier                                 | Architect         | Critic              | Stakeholder         | Use When                                  |
| ------------------------------------ | ----------------- | ------------------- | ------------------- | ----------------------------------------- |
| **Premium** (API, high subscription) | 1x Sonnet         | 3x Haiku (parallel) | 2x Haiku (parallel) | Production decisions, high-stakes reviews |
| **Standard** (API, mid subscription) | 1x Sonnet         | 1x Sonnet           | 1x Haiku            | Regular planning, design reviews          |
| **Local** (Ollama, consumer GPU)     | 1x Sonnet (cloud) | 2x Haiku (local)    | 1x Haiku (local)    | Hybrid: architect on cloud, critics local |
| **Minimal** (Ollama, no GPU)         | 1x Haiku          | 1x Haiku            | 1x Haiku            | All local, sequential rounds              |

**Why plural Critic/Stakeholder?** Multiple haiku instances with different random seeds produce diverse findings at ~1/3 the cost of a sonnet call. Aggregate by de-duplication and voting.

## Three-Round Structure

### Round 1: Position Loading (5-10 min)

Each role states their stance **independently** — no cross-talk yet.

```bash
# Architect (single, sonnet)
Agent(subagent_type="meet-architect", prompt="Round 1: State your architectural position for <topic>. Use position format.")

# Critic ensemble (plural, haiku) — dispatch 2-3 in parallel
Agent(subagent_type="meet-critic", prompt="Round 1 [Critic A]: State your concerns for <topic>. Focus on: edge cases, failure modes. Use risk register format.")
Agent(subagent_type="meet-critic", prompt="Round 1 [Critic B]: State your concerns for <topic>. Focus on: hidden assumptions, alternative designs. Use risk register format.")

# Stakeholder ensemble (plural, haiku) — dispatch 2 in parallel
Agent(subagent_type="meet-stakeholder", prompt="Round 1 [Stakeholder A]: State user/business stance for <topic>. Focus on: user impact, timeline.")
Agent(subagent_type="meet-stakeholder", prompt="Round 1 [Stakeholder B]: State user/business stance for <topic>. Focus on: business constraints, success metrics.")
```

**Facilitator aggregates** ensemble findings: de-duplicate risks, vote on severity, merge into single position per role.

### Round 2: Challenge & Defense (10-20 min)

Roles engage with each other's positions.

```bash
# Sequential: each role reads others, then responds
Agent(subagent_type="meet-critic", prompt="Round 2: Challenge Architect's position. Ask 3 specific 'what if' questions.")
Agent(subagent_type="meet-architect", prompt="Round 2: Defend your design against Critic's challenges. Acknowledge valid concerns.")
Agent(subagent_type="meet-stakeholder", prompt="Round 2: Question both from user perspective. Is this the right scope?")
```

**For plural ensembles:** Aggregate Critic challenges first (de-duplicate, vote on top 3), then Architect responds to the merged set.

**Facilitator tracks** which challenges were resolved vs. escalated to Decider.

### Round 3: Decision & Commit (5-10 min)

Decider rules, action items assigned.

```bash
Agent(subagent_type="meet-architect", prompt="Round 3: Document the decision in ADR format. Note any dissenting views.")
```

If no Decider role present, main session (you) makes the call.

## Meeting Templates

Load templates from `skills/meet-facilitator/templates/`:

| Template      | File               | Use When                |
| ------------- | ------------------ | ----------------------- |
| Design Review | `design-review.md` | Architectural decisions |
| Planning Sync | `planning-sync.md` | Implementation planning |
| Debug Huddle  | `debug-huddle.md`  | Bug investigation       |
| Retrospective | `retrospective.md` | Process improvement     |
| Standup       | `standup.md`       | Daily coordination      |

## Decision Log Format

Every meeting produces a decision log entry:

```markdown
## Decision: <title>

**Date:** YYYY-MM-DD
**Meeting Type:** design-review | planning-sync | debug-huddle | retrospective
**Participants:** Architect, Critic, Stakeholder, Decider

### Context
[2-3 sentences on what decision was needed]

### Positions
- **Architect:** [summary]
- **Critic:** [key risks flagged]
- **Stakeholder:** [user/business constraints]

### Decision
[What was decided, with rationale]

### Dissent (if any)
[Critic or Stakeholder disagreement, why overruled]

### Action Items
| Item | Owner | Deadline |
|------|-------|----------|
| [task] | [role] | [date] |
```

## Anti-Patterns

- **Skipping Round 1** — Don't let roles react to each other before stating own position
- **No timeboxing** — Each round has a time limit; enforce it
- **Consensus-seeking** — Decider rules; dissent is recorded, not debated to death
- **No decision log** — If it's not written down, it didn't happen
- **Wrong roles** — Don't invite Stakeholder to a deep technical debug huddle

## Integration

### With core-orchestrate

`core-orchestrate` skill calls this skill when:

- Architectural decisions are needed mid-implementation
- Parallel sub-agents disagree on approach
- A design review gate is required before shipping

### With core-dispatch

Workflows can define meeting stages:

```json
{
  "stage": "design-review",
  "skill": "meet-facilitator",
  "template": "design-review",
  "roles": ["meet-architect", "meet-critic", "decider"],
  "gate": "always"
}
```

### With inter-session bus

Post meeting outcomes to bus:

```bash
uv run bus write '{"content": "Meeting complete: <decision summary>. Log: <path>.", "from": "meet-facilitator", "to": "all", "type": "status"}'
```

## Related

- `agents/meet-architect.md` — Big-picture design role
- `agents/meet-critic.md` — Edge cases and risk analysis role
- `agents/meet-stakeholder.md` — User/business perspective role
- `skills/core-orchestrate/SKILL.md` — Higher-level orchestration
- `rules/20-essential/07-meeting-discipline.md` — When meetings are required
