# Meeting Discipline — STRICT

## weight: 7 category: essential description: Multi-agent meetings required for architectural decisions, API changes, and breaking changes

**When making significant decisions, use the three-round meeting structure with role-based sub-agents.**

## When Meetings Are Required

| Trigger                                              | Meeting Type  | Required Roles                 |
| ---------------------------------------------------- | ------------- | ------------------------------ |
| Architectural decision (new service, major refactor) | Design Review | Architect, Critic, Decider     |
| API change (breaking or public-facing)               | Design Review | Architect, Critic, Stakeholder |
| Implementation plan (Phase-level)                    | Planning Sync | Architect, Stakeholder         |
| Production incident                                  | Debug Huddle  | Architect, Critic              |
| Sprint/phase wrap-up                                 | Retrospective | All roles + Driver             |
| Parallel work coordination                           | Standup       | All roles                      |

## Three-Round Structure (Mandatory)

Every meeting has three distinct rounds:

1. **Round 1: Position Loading** — Each role states stance independently (no cross-talk)
1. **Round 2: Challenge & Defense** — Roles probe each other's positions
1. **Round 3: Decision & Commit** — Decider rules, action items assigned

**Do not skip rounds.** Round 1 prevents groupthink. Round 2 surfaces risks. Round 3 produces decisions.

## Decision Log Requirement

Every meeting produces a decision log entry at `decisions/YYYY-MM-DD-<slug>.md`:

```markdown
## Decision: <title>

**Date:** YYYY-MM-DD
**Meeting Type:** design-review | planning-sync | debug-huddle | retrospective | standup
**Participants:** [roles]

### Context
[What decision was needed]

### Positions
- **Architect:** [summary]
- **Critic:** [key risks]
- **Stakeholder:** [constraints]

### Decision
[What was decided, rationale]

### Dissent
[Any disagreement, why overruled]

### Action Items
| Item | Owner | Deadline |
|------|-------|----------|
```

## Role Definitions

| Role            | Purpose                           | Cognitive Style     |
| --------------- | --------------------------------- | ------------------- |
| **Architect**   | System design, big-picture        | Divergent thinking  |
| **Critic**      | Edge cases, failure modes         | Convergent thinking |
| **Stakeholder** | User impact, business constraints | Outcome-focused     |
| **Driver**      | Process facilitation              | Procedural          |
| **Decider**     | Final call on disagreements       | Decisive            |

## Anti-Patterns

- **Skipping Round 1** — Don't let roles react before stating own position
- **Consensus-seeking** — Decider rules; dissent is recorded, not debated to death
- **No decision log** — If it's not written down, it didn't happen
- **Wrong roles** — Don't invite Stakeholder to a deep technical debug huddle
- **No timeboxing** — Each round has a time limit; enforce it

## Integration

### With core-orchestrate

Dispatch meeting skill via:

```bash
/skill meet-facilitator <meeting-type> --topic "<decision needed>"
```

### With inter-session bus

Post meeting outcomes:

```bash
uv run bus write '{"content": "Meeting complete: <decision summary>. Log: decisions/YYYY-MM-DD-<slug>.md", "from": "<agent-id>", "to": "all", "type": "status"}'
```

## Enforcement

- Architectural decisions without a decision log → flagged in code review
- API changes without Design Review → revert and re-do properly
- Retrospectives without experiments → not counted as complete

## Related

- `skills/meet-facilitator/SKILL.md` — Meeting orchestration
- `agents/meet-architect.md` — Architect role definition
- `agents/meet-critic.md` — Critic role definition
- `agents/meet-stakeholder.md` — Stakeholder role definition
