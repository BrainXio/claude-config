---
name: doc-sync
description: Synchronize documentation with codebase changes using drift detection
argument-hint: '[scope]'
---

# Doc Sync

Synchronize documentation with the current state of the codebase.

## Procedure

1. Analyze what changed: new agents, skills, hooks, entry points, deny rules, or CI jobs
1. Run drift-detector agent to find documentation/code mismatches
1. Check reference docs for missing entries in affected tables
1. Check for items needing status updates
1. Identify new user-facing workflows needing how-to additions
1. Apply markdown-formatter agent on changed documentation
1. Report what was updated and what still needs manual attention
