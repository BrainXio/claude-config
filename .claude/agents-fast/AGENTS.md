# Agent Definitions — 8B Optimized

| Agent                             | Model  | Purpose                                 |
| --------------------------------- | ------ | --------------------------------------- |
| `meet-architect`                  | sonnet | System design, big-picture              |
| `meet-critic`                     | haiku  | Edge cases, failure modes               |
| `meet-stakeholder`                | haiku  | User impact, business constraints       |
| `dev-code-reviewer`               | sonnet | Design, correctness, maintainability    |
| `dev-bug-hunter`                  | sonnet | Race conditions, logic bugs, edge cases |
| `dev-qa-engineer`                 | sonnet | Test generation, coverage gaps          |
| `dev-test-writer`                 | sonnet | Test case generation                    |
| `dev-complexity-reducer`          | sonnet | High-cyclomatic functions               |
| `dev-dry-enforcer`                | sonnet | Duplicated logic blocks                 |
| `dev-kiss-auditor`                | sonnet | Unnecessarily complex code              |
| `dev-yagni-auditor`               | sonnet | Over-engineered abstractions            |
| `dev-dead-code-hunter`            | sonnet | Unused functions, variables             |
| `dev-single-source-auditor`       | sonnet | Duplicated constants/configs            |
| `ops-security-engineer`           | sonnet | Threat modeling, compliance             |
| `ops-dependency-graph-analyzer`   | sonnet | Circular deps, version conflicts        |
| `ops-protocol-compliance-auditor` | sonnet | MCP server compliance                   |
| `kb-drift-detector`               | haiku  | Documentation drift across repos        |
| `kb-markdown-formatter`           | haiku  | Format/lint markdown                    |

**Invocation:** Use sub-agents for parallel work, large tasks, or when task exceeds role threshold.
