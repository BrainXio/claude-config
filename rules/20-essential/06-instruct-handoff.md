# Session Instruct Handoff — STRICT

## weight: 7 category: essential description: Every active plan must have a corresponding instruct file for inter-session handoff

**Every plan with `status: active` or `status: planned` must have a corresponding
`instructions/INSTRUCT_<SLUG>.md` file. No exceptions.**

## Rule

When a session activates a plan (reads it, starts working on it, or references
it as a parent), the agent must verify the corresponding instruct file exists.

### If the instruct file exists

Read it. Append a new paragraph describing what this session intends to achieve
or needs from future sessions. Do not overwrite existing paragraphs — each
session is an author.

### If the instruct file does NOT exist

Create it immediately at `instructions/INSTRUCT_<SLUG>.md` with:

1. **YAML frontmatter**: `slug`, `plan`, `created`, `priority` (copy from plan)
1. **Current session paragraph**: What this session is doing, what it needs from
   the next session, and any decisions or blockers worth surfacing
1. **Plan relationship summary**: If this plan depends on or blocks other plans,
   state the relationship explicitly

## Content Guidelines

### What to include

- Session intent: "This session is implementing Phase 3 of the authentication
  refactor."
- Handoff needs: "Next session needs to wire the new middleware into the router."
- Blockers or decisions: "We chose Redis over Memcached after load testing."
- Plan relationships: "Blocked by PLAN_INFRA_RESTRUCTURE.md (submodules not yet
  moved)."

### What to skip

- Raw git diffs or commit hashes (those belong in the plan or reports)
- Content that is already in `reports/session-*.md`
- Generic pleasantries or AI attribution

## Format

```markdown
---
slug: auth-refactor
plan: plans/PLAN_AUTH_REFACTOR.md
created: 2026-05-26
priority: high
---

# INSTRUCT: Authentication Refactor

## Session 2026-05-26 — workspace-redesign

This session established the directory layout for the new auth package and
migrated the legacy password hash utilities. The middleware interface is
defined but not yet wired into the router. Next session should integrate the
middleware and update all route guards.

Blocked by: PLAN_INFRA_RESTRUCTURE (waiting on CI pipeline changes).
```

## Anti-patterns

- Skipping the instruct file because "there's nothing to hand off" — there is
  always intent worth recording
- Overwriting previous session paragraphs — history is additive
- Creating an instruct file without reading the plan first — the instruct must
  be grounded in the plan's actual state

## Enforcement

- The session must not be considered complete until the instruct file is
  verified or created
- Missing instruct files are flagged in session reports under Maintained vs
  Patched
