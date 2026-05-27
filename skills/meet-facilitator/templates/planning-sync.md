# Template: Planning Sync

**Use when:** Implementation planning, sprint planning, feature scoping

**Participants:** Architect, Stakeholder (Critic optional)

**Duration:** 20 minutes

______________________________________________________________________

## Pre-Meeting (Driver)

1. Load the feature/epic to plan
1. Identify known constraints (timeline, resources)
1. Dispatch Round 1

______________________________________________________________________

## Round 1: Position Loading (5 min)

### Architect

State the technical approach:

- Components to build/modify
- Dependencies (internal, external)
- Estimated complexity (S/M/L/XL)
- Technical risks

### Stakeholder

State the outcome requirements:

- User problem being solved
- Must-have features (MVP)
- Nice-to-have features (v2)
- Success metrics

______________________________________________________________________

## Round 2: Challenge & Defense (10 min)

### Stakeholder → Architect

- Is this the smallest thing that works?
- What can we cut to ship faster?
- What's the opportunity cost?

### Architect → Stakeholder

- What's feasible in the timeline
- What technical debt we're taking on
- What we can't cut (foundational work)

### Critic (if present)

- What could go wrong with this plan
- What dependencies are risky
- What testing is needed

______________________________________________________________________

## Round 3: Decision & Commit (5 min)

### Both

Agree on the plan:

- **Scope** — what's in, what's out
- **Timeline** — when it ships
- **Risks acknowledged** — what we're watching

### Architect

Document the plan:

- Implementation phases
- Dependencies to resolve
- Tests to write

______________________________________________________________________

## Output

```markdown
## Plan: <feature name>

**Date:** YYYY-MM-DD
**Participants:** Architect, Stakeholder

### Scope
**In:** [features to build]
**Out:** [explicitly excluded]

### Timeline
- Phase 1: [date]
- Phase 2: [date]
- Ship: [date]

### Risks
- [risk 1] → [mitigation]
- [risk 2] → [mitigation]

### Action Items
| Item | Owner | Deadline |
|------|-------|----------|
```
