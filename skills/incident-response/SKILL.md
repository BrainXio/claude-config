---
name: incident-response
description: Error mitigation and incident response skill
tools: Read, Write, Bash, Glob, Grep
model: sonnet
permissionMode: default
color: red
---

# Incident Response Skill

**Trigger:** Any command, tool, or API call fails with an error.

**Purpose:** Rapid mitigation + documented learning. No silent workarounds.

## Invocation

```bash
/skill incident-response "<error context>"
```

Or auto-triggered when:

- Shell command returns non-zero exit
- Tool call fails (Edit/Write/Bash error)
- API returns 4xx/5xx
- Test suite fails

## Response Protocol

### Phase 1: Triage (2 min)

1. **Capture error context**

   - Command/tool that failed
   - Full error output
   - Environment state (cwd, branch, recent changes)

1. **Classify severity**

   | Severity | Response                              | Example                               |
   | -------- | ------------------------------------- | ------------------------------------- |
   | SEV1     | Block all work, escalate immediately  | Data loss, security breach, prod down |
   | SEV2     | Mitigate within 10 min, continue work | Test failures, build broken           |
   | SEV3     | Document workaround, continue         | Lint errors, missing docs             |

1. **Check for existing mitigation**

   ```bash
   grep -r "<error signature>" docs/mitigations/
   ```

### Phase 2: Mitigation (5-10 min)

**Immediate actions** (pick fastest):

- **Rollback**: `git revert HEAD` or `git checkout <last-good>`
- **Bypass**: Feature flag, skip flag, alternative path
- **Isolate**: Narrow scope, reduce blast radius
- **Escalate**: Spawn subagent with higher capability

**Rule:** If mitigation >10 min, escalate to Sonnet/Opus subagent.

### Phase 3: Document (3 min)

Create `docs/mitigations/<YYYY-MM-DD>-<slug>.md`:

```markdown
---
date: 2026-05-28
severity: SEV2
error_type: <category>
affected_component: <path or module>
mitigation_time: 8m
---

# <Error Title>

## Error Signature
```

<exact error message>
```

## Context

- Command: `failed command`
- Branch: `branch-name`
- Recent changes: <what changed before this broke>

## Root Cause

\<One paragraph: why did this happen?>

## Mitigation Applied

1. \<step 1>
1. \<step 2>
1. <verification>

## Long-term Fix

\<What permanent fix is needed? Link to issue if created.>

## Related

- \<links to related docs/issues>

````

### Phase 4: Consume (Ongoing)

**Before retrying similar work:**
1. Search mitigations: `grep -r "<pattern>" docs/mitigations/`
2. Read relevant mitigation docs
3. Apply documented workaround or verify fix is in place

**When spawning subagents:**
- Attach relevant mitigation docs to prompt
- Verify subagent acknowledges constraints

## Patterns

### Command Failure
```bash
# Error: npm command fails
/skill incident-response "npm install failed with ENOENT"

# Mitigation: Check node version, clear cache, retry
# Document: docs/mitigations/2026-05-28-npm-enoent.md
````

### Tool Failure

```text
# Error: Edit tool fails on file
/skill incident-response "Edit tool failed: file locked or syntax error"

# Mitigation: Read file, check syntax, manual edit
# Document: docs/mitigations/2026-05-28-edit-failure.md
```

### API Failure

```text
# Error: GitHub API rate limit
/skill incident-response "gh API 403 rate limited"

# Mitigation: Wait 60s, retry with backoff, use auth token
# Document: docs/mitigations/2026-05-28-gh-rate-limit.md
```

## Anti-patterns

- **Silent retry**: Never retry >3 times without documenting
- **Workaround without docs**: Temporary fix must have mitigation doc
- **Blame attribution**: Focus on system failure, not human error
- **Over-documentation**: Mitigation doc ≤1 page; full RCA only for SEV1

## Integration

| System            | Integration                                      |
| ----------------- | ------------------------------------------------ |
| Inter-session bus | Post SEV1/SEV2 incidents to bus                  |
| Knowledge base    | Mitigation docs indexed for search               |
| Issue tracker     | Create issues for long-term fixes                |
| Pre-commit hook   | Block if mitigation doc missing for recent error |

## Related

- `rules/20-essential/08-issue-reporting.md` — External issue filing
- `agents/incident-response.md` — Incident commander agent
- `docs/mitigations/` — Mitigation document archive
