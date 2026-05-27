# Template: Debug Huddle

**Use when:** Bug investigation, production incident, flaky test

**Participants:** Architect, Critic

**Duration:** 15 minutes

______________________________________________________________________

## Pre-Meeting (Driver)

1. Load the bug report / incident details
1. Gather logs, error messages, reproduction steps
1. Dispatch Round 1

______________________________________________________________________

## Round 1: Position Loading (5 min)

### Architect

State hypotheses:

- Where the bug might be
- What recent changes could have caused it
- What system components are involved

### Critic

State investigation angles:

- What logs to check
- What edge cases to test
- What assumptions to verify

______________________________________________________________________

## Round 2: Challenge & Defense (5 min)

### Critic → Architect

- What evidence supports each hypothesis
- What would disprove each hypothesis
- What's the simplest explanation

### Architect → Critic

- Test plan for each hypothesis
- What we can rule out
- What needs deeper investigation

______________________________________________________________________

## Round 3: Decision & Commit (5 min)

### Both

Agree on investigation plan:

- **Primary hypothesis** — most likely cause
- **Tests to run** — how to confirm
- **Fallback** — if primary is wrong, what next

### Architect

Document findings:

- Root cause (once found)
- Fix approach
- Prevention (how to avoid recurrence)

______________________________________________________________________

## Output

```markdown
## Debug: <bug description>

**Date:** YYYY-MM-DD
**Participants:** Architect, Critic

### Hypotheses
1. [hypothesis 1] — likelihood: High/Med/Low
2. [hypothesis 2] — likelihood: High/Med/Low

### Investigation Plan
- [test 1] → confirms/rules out [hypothesis]
- [test 2] → confirms/rules out [hypothesis]

### Root Cause
[Once identified]

### Fix
[What change resolves it]

### Prevention
[How to avoid recurrence]
```
