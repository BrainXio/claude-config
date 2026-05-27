# Template: Design Review

**Use when:** Architectural decisions, API design, system boundary changes

**Participants:** Architect, Critic, Decider (Stakeholder optional)

**Duration:** 30 minutes

______________________________________________________________________

## Pre-Meeting (Driver)

1. Load context: What decision is needed?
1. Identify constraints: Timeline, technical, business
1. Dispatch Round 1

______________________________________________________________________

## Round 1: Position Loading (10 min)

### Architect

State your design:

- System boundaries (in scope / out of scope)
- Interface contracts (how components communicate)
- Data flow (where state lives, how it moves)
- Trade-offs (what optimizing for, what sacrificing)

### Critic

State your concerns:

- Edge cases (empty, full, concurrent, timeout)
- Failure modes (stress, partial failure, bad input)
- Hidden assumptions
- Alternative designs rejected

### Stakeholder (if present)

State user/business stance:

- User impact (who benefits, how they experience it)
- Business constraints (timeline, budget, compliance)
- Priority ranking (must-have vs. nice-to-have)
- Success metrics

______________________________________________________________________

## Round 2: Challenge & Defense (15 min)

### Critic → Architect

Ask 3 specific "what if" questions:

- What if this is empty?
- What if it's 100x larger?
- What if it fails mid-operation?

### Architect → Critic

Defend with first principles:

- Why this structure, not alternatives
- Acknowledge valid concerns
- Propose mitigations for risks

### Stakeholder → Both

Question from user perspective:

- Does this solve the user problem?
- Is this the right scope?
- What's the opportunity cost?

______________________________________________________________________

## Round 3: Decision & Commit (5 min)

### Decider

Rule on the design:

- **Approved** — proceed as designed
- **Approved with changes** — proceed with specific modifications
- **Rejected** — return with alternative
- **Escalated** — flag for higher-level decision

### Architect

Document in ADR format:

- Decision summary
- Rationale
- Dissenting views (if any)
- Action items with owners

______________________________________________________________________

## Output

```markdown
## Decision: <title>

**Date:** YYYY-MM-DD
**Meeting Type:** Design Review
**Participants:** Architect, Critic, Decider

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
