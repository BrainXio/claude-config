# Branch Strategy and CI Gate — STRICT

## weight: 5 category: essential description: Branch naming, protection, and CI pass gate before merge

**All changes must go through a CI-passing branch before merging to main.**

## Branch Strategy

### Main Branch

- `main` is the source of truth
- Direct push to `main` requires all CI checks to pass
- Force push to `main` is only allowed for exceptional cases (history rewrite, removing secrets). Never routine.

### Feature Branches

- Branch naming: `<type>/<description>` using kebab-case
- Types: `feat/`, `fix/`, `chore/`, `docs/`, `refactor/`, `test/`, `ci/`
- Example: `feat/session-start-hook`, `fix/mypy-strict-errors`

### Commit Strategy

- Prefer small, focused commits on feature branches
- Squash merge to `main` keeps history linear
- Each commit must pass CI independently

## CI Gate

### Must Pass Before Merge

1. **Lint** — ruff check (or equivalent)
1. **Format** — ruff format --check (or equivalent), mdformat --check with plugins
1. **Type check** — mypy --strict (or equivalent)
1. **Tests** — pytest (or equivalent)
1. **Commit message** — commitlint (conventional commits)

### CI Configuration

- CI workflow must run on every push and pull request
- Use shared workflows (BrainXio/cicd) for consistency across repos
- Do not disable CI checks without documented justification
- If a check fails, fix the issue — never skip with `--no-verify`

### Force Push Mitigations

- Force push of a new root history breaks commitlint (no common ancestor)
- In this specific case, the commit message check must be skipped or manually verified
- After the first post-force-push commit, commitlint returns to normal
- Do not leave repos in a state where CI can never pass

## Anti-patterns

- Never merge with failing CI
- Never disable branch protection to bypass CI
- Never force push to `main` without explicit justification
- Never use generic branch names like `fix`, `test`, or `update`
- Never push directly to `main` for non-trivial changes — use a branch
