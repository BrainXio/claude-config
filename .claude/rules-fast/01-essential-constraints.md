# Essential Constraints — 8B Optimized

**Weight: 4-8**

## Weight 4: Conventional Commits

Format: `<type>[(scope)]: <description>`
Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `ci`, `build`, `perf`
Rules: lowercase description, no trailing period, scope encouraged.

## Weight 5: Branch Strategy

- `main` is source of truth, CI must pass before merge
- Feature branches: `<type>/<description>` (e.g., `feat/auth-flow`)
- CI gate: lint, format, type-check, tests, commitlint

## Weight 6: Context Preservation

Delegate to sub-agents when:

- Independent research (audits, greps)
- Parallel execution (multiple files)
- Lengthy tasks (>50% context or >10 tool calls)
- Large outputs (log parsing, bulk reads)

Max 2 levels deep. Return file paths, line numbers, conclusions — not raw dumps.

## Weight 6: Anti-Overengineering

Seven principles:

1. **Ground in physical reality** — Denominate in real currency/features shipped
1. **Solve current failures** — What breaks today without this?
1. **Measure outcomes** — Not activity, not proxy metrics
1. **No academic-validation** — Operational need, not citations
1. **Design for current capabilities** — Match what exists today
1. **Trust upward** — Enable growth, don't punish
1. **Complexity carries tax** — Who maintains it? How tested?

## Weight 7: Session Reporting

At session end, produce:

1. `reports/session-YYYY-MM-DD-<slug>-speed-vs-coverage.md` — Lines changed, coverage delta, time elapsed
1. `reports/session-YYYY-MM-DD-<slug>-maintained-vs-patched.md` — Structural improvements vs defensive fixes

Backfill missing reports before new work.

## Weight 7: Instruct Handoff

Every active/planned plan needs `instructions/<plan-slug>/INSTRUCT.md`:

- YAML frontmatter: slug, plan, created, priority
- Session paragraph: what this session did, what next session needs
- Plan relationships: blocked by / blocks

## Weight 8: Issue Reporting

When encountering bugs in external packages:

1. Verify it's not user error
1. File at origin (GitHub repo)
1. Link locally
   Do not duplicate, do not silently workaround.
