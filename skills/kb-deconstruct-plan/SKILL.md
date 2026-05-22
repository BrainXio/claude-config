# Deconstruct Plan Skill

Execute the R.D.P. (Read/Deconstruct/Plan) ritual on a master plan document
and output a deconstructed planning markdown to `~/.config/claude/planning/<repo>/`.

## Procedure

1. Resolve the master plan path from the argument:
   - If the argument starts with `/`, use as absolute path
   - If the argument is a relative slug like `master-plans/cpr-self-healing.md`,
     resolve from `~/.config/claude/docs/` first, then fall back to
     `~/dev/.meta/`
   - If `--from-mcp` is passed, use the documentation MCP server
     `get_document` tool instead of filesystem reads
1. Read the master plan in full (Phase 1: Read)
1. Deconstruct into atomic first-principles elements (Phase 2: Deconstruct):
   - One file per element unless tightly coupled
   - Every element has a testable pass/fail verification criterion
   - Dependencies between elements are explicit
   - Ambiguities are flagged for clarification
1. Determine the target repo:
   - If `--repo` is provided, use it
   - Otherwise infer from plan content (affected files, branch conventions)
1. Plan — write the output to `~/.config/claude/planning/<repo>/<slug>-planning.md` using the
   R.D.P. template (Phase 3: Plan)
1. Report the output path and element count

## Output Template

```markdown
---
title: <Plan Title — Deconstructed>
repo: <target-repo>
branch: <type>/<short-description>
source: <master-plan-path>
depends_on: []
created: <YYYY-MM-DD>
---

# <Plan Title>

## Goal

<One sentence from R.D.P. Read phase>

## Scope

| In scope | Out of scope |
|-----------|--------------|
| <item> | <item> |

## Elements

### 1. <Element Name>

| Field | Value |
|-------|-------|
| Files | `<paths>` |
| Purpose | <one sentence> |
| Depends on | <element IDs or "none"> |
| Verification | <pass/fail criterion> |

### 2. <Element Name>

...

## Execution Order

<Numbered list reflecting the dependency graph. Parallelizable groups bracketed.>

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| <risk> | Low/Med/High | Low/Med/High | <mitigation> |

## Ambiguities

<Flagged items from the Read phase that require clarification before scheduling.>
```

## Related

- [R.D.P. Ritual](~/.config/claude/docs/rules-and-regulations/rdp-ritual.md) —
  canonical ritual definition
- [schedule-tasks](../schedule-tasks/SKILL.md) — next step in the pipeline
- [A.E.D. Ritual](~/.config/claude/docs/rules-and-regulations/aed-ritual.md) —
  complementary backward-looking assessment ritual
