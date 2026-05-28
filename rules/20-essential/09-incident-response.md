# Incident Response — STRICT

## weight: 8 category: essential description: All errors must trigger incident response protocol with documented mitigation

**No silent failures. No undocumented workarounds.**

## Rule

When any command, tool, or API call fails:

1. **Invoke incident response** — `/skill incident-response "<error context>"`
1. **Triage severity** — SEV1/SEV2/SEV3 per matrix below
1. **Mitigate** — Execute fastest recovery path (rollback, bypass, isolate)
1. **Document** — Write mitigation doc at `docs/mitigations/<YYYY-MM-DD>-<slug>.md`
1. **Consume** — Search mitigations before retrying similar work

## Severity Matrix

| Severity | MTTM Target | Escalation        | Bus Post | Mitigation Doc | Example                               |
| -------- | ----------- | ----------------- | -------- | -------------- | ------------------------------------- |
| SEV1     | 2 min       | Immediate to Opus | Required | Required       | Data loss, security breach, prod down |
| SEV2     | 10 min      | Sonnet if >10 min | Optional | Required       | Build broken, tests failing           |
| SEV3     | Best effort | None              | None     | Recommended    | Lint errors, docs, cosmetic           |

## Mitigation Doc Requirements

```markdown
---
date: 2026-05-28
severity: SEV2
error_type: <category>
affected_component: <path>
mitigation_time: <duration>
---

# <Title>

## Error Signature
```

<exact error message>
```

## Context

- Command: `<failed command>`
- Branch: `<branch-name>`
- Recent changes: <what changed>

## Root Cause

<one paragraph explanation>

## Mitigation Applied

1. \<step 1>
1. \<step 2>
1. <verification>

## Long-term Fix

\<permanent fix needed, link to issue>

````

## Enforcement

- **Pre-commit hook** — Blocks if SEV1/SEV2 error occurred and no mitigation doc exists
- **SessionEnd summary** — Lists incidents without mitigation docs
- **Code review** — Flags workarounds without linked mitigation docs

## Consumption Protocol

Before retrying work similar to previous failures:

1. **Search mitigations**
   ```bash
   grep -rl "<error pattern>" docs/mitigations/
````

2. **Read relevant docs** — Understand root cause and workaround

1. **Apply documented fix** — Or verify permanent fix is in place

1. **Update mitigation** — If workaround evolved, amend the doc

## Escalation Path

```
SEV3 → Handle autonomously
SEV2 → Spawn Sonnet subagent if >10 min
SEV1 → Spawn Opus subagent immediately, post to bus
```

## Anti-patterns

- **Silent retry** — Never retry >3 times without documenting
- **Workaround without docs** — Temporary fix must have mitigation doc
- **Blame attribution** — Focus on system failure, not human error
- **Over-documentation** — Mitigation doc ≤1 page; full RCA only for SEV1
- **Stale mitigations** — Update docs when permanent fix is deployed

## Integration

| System                              | Integration              |
| ----------------------------------- | ------------------------ |
| `skills/incident-response/SKILL.md` | Skill invocation         |
| `agents/incident-response.md`       | Incident commander agent |
| `docs/mitigations/`                 | Mitigation archive       |
| Inter-session bus                   | SEV1/SEV2 status posts   |

## Related

- `rules/20-essential/08-issue-reporting.md` — External issue filing for long-term fixes
- `rules/10-critical/01-no-secrets.md` — SEV1 security incidents
- `skills/core-dispatch/SKILL.md` — Subagent spawning for escalation
