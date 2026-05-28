# CLAUDE.md

Claude Code framework configuration — rules, agents, skills, workflows merged into `.claude/`.

## Structure

| Directory    | Contents                                                                     | Count |
| ------------ | ---------------------------------------------------------------------------- | ----- |
| `agents/`    | Sub-agent definitions (YAML frontmatter + prompt)                            | 22    |
| `rules/`     | Hierarchical rules: `00-*`, `10-critical/`, `20-essential/`, `30-standards/` | 15    |
| `skills/`    | Slash-command skills                                                         | 10    |
| `settings/`  | Composable settings JSON files                                               | 21    |
| `workflows/` | Multi-stage agent pipeline JSON                                              | —     |
| `decisions/` | Architecture Decision Records (ADRs)                                         | —     |

## Rules Hierarchy

| Prefix          | Category               | Enforcement                                                                                                                                                 |
| --------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `00-*`          | Universal              | No attribution                                                                                                                                              |
| `10-critical/`  | Critical (weight 1-3)  | Human sovereignty, no secrets, capability boundaries, vision safeguard                                                                                      |
| `20-essential/` | Essential (weight 4-9) | Conventional commits, context preservation, branch strategy, anti-overengineering, session reporting, instruct handoff, meeting discipline, issue reporting |
| `30-standards/` | Standards (weight 10+) | Markdown formatting (mdformat with plugins), alphabetical ordering                                                                                          |

## Agents (22)

**Development:** `dev-bug-hunter`, `dev-code-reviewer`, `dev-coder-worker`, `dev-complexity-reducer`, `dev-dead-code-hunter`, `dev-dry-enforcer`, `dev-error-handler`, `dev-kiss-auditor`, `dev-qa-engineer`, `dev-single-source-auditor`, `dev-test-writer`, `dev-yagni-auditor`

**Knowledge:** `kb-docs-writer`, `kb-drift-detector`, `kb-markdown-formatter`

**Meeting:** `meet-architect` (sonnet), `meet-critic` (haiku), `meet-stakeholder` (haiku)

**Ops:** `ops-dependency-graph-analyzer`, `ops-protocol-compliance-auditor`, `ops-security-engineer`, `ops-standards-guard`

## Skills (10)

`core-dispatch`, `core-orchestrate`, `core-tools`, `dev-code-audit`, `dev-fix-ci`, `dev-ship`, `issue-triage`, `kb-agent-knowledge`, `kb-claude-hooks`, `kb-doc-sync`, `meet-facilitator`

## Usage

```bash
# Format all markdown (required plugins)
uvx --with mdformat-frontmatter --with mdformat-gfm mdformat agents/ rules/ skills/

# Validate settings compose
cat settings/*.json | jq -s 'add'
```
