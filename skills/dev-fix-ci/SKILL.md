______________________________________________________________________

## name: fix-ci description: Diagnose CI failures, apply fixes, and push with proper commit messages argument-hint: <error-source>

# Fix CI

Resolve CI failures systematically with diagnosis, fix, verification, and proper commit.

## Procedure

1. Accept error context: CI failure log, linting output, type error, or test failure
1. Run error-handler agent to diagnose root cause and apply fix
1. Verify the fix passes all checks locally: ruff, mypy, mdformat
1. If tests are affected, run test-writer agent to assess coverage impact
1. Craft a conventional commit message for the fix
1. Push and verify CI passes on the remote branch
