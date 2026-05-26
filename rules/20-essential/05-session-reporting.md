# Session Reporting Discipline — STRICT

## weight: 7 category: essential description: Every planned session produces a Speed vs Coverage and Maintained vs Patched report

**No report backlog may accumulate across sessions.**

## Rule

At the end of every planned session (Phase, sprint task, or bounded work unit), the agent must produce three artifacts:

Two in `reports/` (tactical):

1. **Speed vs Coverage** — `reports/session-YYYY-MM-DD-<slug>-speed-vs-coverage.md`

   - Lines changed
   - Test coverage delta
   - Lint / type error delta
   - Time elapsed

1. **Maintained vs Patched** — `reports/session-YYYY-MM-DD-<slug>-maintained-vs-patched.md`

   - Structural improvements (maintained)
   - Defensive fixes (patched)
   - Deviations from checklist with justification and time impact

One in `knowledge-base/` (strategic, per-phase only):

3. **First-Principles Retrospective** — `knowledge-base/<plan-slug>/phase-N-<slug>.md`
   - Diataxis framework: Explanation, How-to Guide, Reference, Tutorial
   - Trap encountered → first-principles cause → generalized detection → prevention pattern
   - Goal: zero-regression scaling; future phases encountering a documented trap must flag it as deviation

## Backfill Protocol

If reports are missing for prior sessions, backfill them **before** starting new work. This is not optional. A missing report is treated as unclosed session scope.

## Anti-patterns

- Skipping reports because "the work speaks for itself"
- Letting the `reports/` directory grow stale across multiple sessions
- Writing reports only for phases that went well
- Including raw git diffs instead of summarized deltas

## Enforcement

- The `pre-commit` hook checks that `reports/` is not empty if `git diff --cached` shows source changes
- Missing reports are flagged in the SessionEnd summary
