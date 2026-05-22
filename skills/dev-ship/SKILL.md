# Ship Skill

Create a pull request with correct labels, body template, and follow-up monitoring.

## Options

- `--label <label>` — force a specific label instead of auto-detecting from commit prefix
- `--no-merge` — skip merge step (leave PR open for manual review)

## Procedure

1. Analyze current branch commits to determine PR title prefix from conventional commit types
1. Map prefix to primary label using unified `type:*` prefix: `feat:` → `type:enhancement`, `fix:` → `type:bug`, `docs:` → `type:documentation`, `ci:` → `type:ci`, `refactor:` → `type:debt`, `test:` → `type:ci`, `perf:` → `type:enhancement`, `chore:` → `type:debt`, `style:` → `type:debt`, `security:` → `type:security`
1. Detect secondary labels: `type:documentation` if `docs/` files changed, `area:workflows` if `.github/workflows/` changed; auto-detect `area:*` labels from changed file paths using generic path patterns
1. Run local checks: ruff, mypy, mdformat
1. Draft PR body using the standard template (Summary + Test plan)
1. Create PR with all labels applied at creation time
1. Monitor CI status and update test plan checkboxes on success
