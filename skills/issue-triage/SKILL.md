# Issue Triage Skill

## Mission

Triage incoming issues at BrainXio repositories: validate reproduction, assign labels, route to owners, and respond to reporters.

## When Dispatched

- New issues filed on BrainXio repos
- Issues needing `needs-reproduction` or `needs-more-info` labels
- Weekly triage rotation for maintainers
- Before sprint planning (backlog grooming)

## Triage Workflow

### Step 1: Validate Issue Exists

```bash
# Check if issue is already filed
gh issue view <number> --repo BrainXio/<repo>
```

### Step 2: Categorize

| Category        | Labels                                     | Action                          |
| --------------- | ------------------------------------------ | ------------------------------- |
| Bug             | `bug`, `needs-reproduction` or `confirmed` | Verify reproduction steps       |
| Feature Request | `enhancement`                              | Assess scope, assign milestone  |
| Documentation   | `documentation`                            | Assign to docs owner            |
| Question        | `question`                                 | Answer or convert to discussion |
| Invalid         | `invalid`                                  | Close with explanation          |
| Duplicate       | `duplicate`                                | Close, link to original         |

### Step 3: Respond to Reporter

**Template for valid bugs:**

```markdown
Thanks for reporting this, @<user>. I've confirmed the issue and added it to 
our <milestone> milestone. We'll investigate and provide an update within 
<timeframe>.

In the meantime, a workaround is: <workaround if available>
```

**Template for needs-more-info:**

```markdown
Thanks for filing this, @<user>. To help us investigate, could you provide:

1. [Specific missing info, e.g., "Python version"]
2. [Specific missing info, e.g., "Full stack trace"]
3. [Specific missing info, e.g., "Minimal reproduction"]

We'll mark this as `needs-reproduction` until we can reproduce locally.
```

### Step 4: Route to Owner

| Component     | Owner         | Label             |
| ------------- | ------------- | ----------------- |
| claude-cli    | low-effort    | `area:cli`        |
| claude-config | medium-effort | `area:config`     |
| agents-core   | high-effort   | `area:core`       |
| bus tool      | low-effort    | `area:bus`        |
| model-tool    | TBD           | `area:model-tool` |

### Step 5: Update Project Board

```bash
# Add to sprint backlog or icebox
gh issue edit <number> --add-project "BrainXio Sprint Board" --field "Status" "Backlog"
```

## Output Format

Triage produces a comment on the issue:

```markdown
## Triage Summary

| Field | Value |
|-------|-------|
| Category | bug | enhancement | docs | question |
| Severity | critical | high | medium | low |
| Area | cli | config | core | bus |
| Milestone | v0.3.0 | v0.4.0 | backlog |
| Owner | @assignee |
| Status | confirmed | needs-reproduction | needs-more-info |

## Next Steps
- [ ] Reproduce locally
- [ ] Assign to owner
- [ ] Add to sprint backlog
```

## Anti-patterns

- **Label spam** — Adding 10+ labels without action
- **Silent triage** — Categorizing without responding to reporter
- **Premature close** — Closing without confirming it's truly invalid
- **Owner-less issues** — Triage without assignment is incomplete
- **Stale needs-reproduction** — Issues stuck in `needs-reproduction` > 14 days

## Related

- `rules/20-essential/08-issue-reporting.md` — Rule requiring agents to file issues
- `rules/20-essential/07-meeting-discipline.md` — Bugs requiring design review
- `skills/core-dispatch/SKILL.md` — Dispatch bugs to appropriate agents
