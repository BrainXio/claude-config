---
name: code-audit
description: Run 7 code quality auditors and merge findings into a single prioritized report
argument-hint: '[scope]'
---

# Code Audit

Orchestrate a full code quality audit across 7 specialized read-only agents.

## Procedure

1. Determine scope from argument (`all`, `changed`, or a specific directory path)
1. Run each audit agent sequentially against the scope:
   - `bug-hunter` — race conditions, type errors, logic bugs, edge cases
   - `complexity-reducer` — high-cyclomatic-complexity functions
   - `dead-code-hunter` — unused functions, variables, configs
   - `dry-enforcer` — duplicated logic blocks
   - `kiss-auditor` — unnecessarily complex implementations
   - `single-source-auditor` — duplicated constants and config values
   - `yagni-auditor` — over-engineered code, speculative features
1. Merge findings into a single consolidated report
1. Deduplicate overlapping issues across agents
1. Prioritize by severity: critical > high > medium > low
1. Present an executive summary, detailed findings table, and remediation roadmap
