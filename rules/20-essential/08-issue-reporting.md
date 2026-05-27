# Issue Reporting Discipline — STRICT

## weight: 8 category: essential description: Agents must file issues at package origin repos when encountering bugs

**No silent workarounds. No issue backlog in bus comments.**

## Rule

When an agent encounters a bug, error, or unexpected behavior in an external package or tool:

1. **Investigate first** — Verify it's not user error, version mismatch, or misconfiguration
1. **File at origin** — Create an issue at the package's GitHub repository
1. **Link locally** — Reference the external issue in local docs or bus messages
1. **Do not duplicate** — Check for existing issues before filing new ones

## When to File

| Scenario                          | Action                            |
| --------------------------------- | --------------------------------- |
| Package bug (crash, wrong output) | File at package repo              |
| Missing documentation             | File at package repo or PR a fix  |
| Feature request                   | File at package repo              |
| Local misconfiguration            | Fix locally, no external issue    |
| Known issue with workaround       | Document workaround locally       |
| Issue in BrainXio-owned repo      | File + assign to relevant project |

## Issue Template

```markdown
## Summary
[One sentence]

## Steps to Reproduce
1. [First step]
2. [Second step]
3. [And so on]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happened]

## Environment
- Package version: X.Y.Z
- Python version: 3.X
- OS: Linux/macOS/Windows

## Additional Context
[Logs, screenshots, or related issues]
```

## Anti-patterns

- **Silent workaround** — Working around a bug without reporting it
- **Bus-only reporting** — Posting about a bug on the bus but not filing at origin
- **Duplicate issues** — Not checking existing issues before filing
- **Vague reproduction steps** — "It doesn't work" without steps to reproduce
- **Blaming the package** — Filing before verifying local configuration

## Enforcement

- Agents must reference external issue numbers when discussing known bugs
- Session reports must list any new issues filed during the session
- Workarounds for unreported bugs are flagged in code review

## Related

- `skills/issue-triage/SKILL.md` — Triage and respond to incoming issues
- `rules/20-essential/07-meeting-discipline.md` — When bugs require design review
