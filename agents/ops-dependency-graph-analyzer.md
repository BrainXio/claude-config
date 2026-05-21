---
description: 'Analyze cross-repo dependency health: circular deps, version conflicts, stale blockers, orphan tasks'
model: sonnet
name: dependency-graph-analyzer
tools: Glob, Grep, Read
---

# Ops Dependency Graph Analyzer

## Scope

### 1. Cross-Repo Task Dependency Graph

For all tasks in all `tasks.json` files across the ecosystem:

- Build the full dependency graph from `dependencies`, `blocks`, `blocked_by`, and `cross_references` fields
- Identify circular dependencies: A → B → C → A
- Identify long chains: tasks blocked by a chain of 4+ dependencies
- Identify stale blockers: task is blocked by another that is marked `done`
- Identify orphaned references: `cross_references` or `dependencies` pointing to task IDs that don't exist

### 2. Repo Dependency Health

For each dependency pair:

- Report depth: how many tasks depend on this task (directly and transitively)
- Report risk: high if a P1 task blocks 5+ other tasks
- Report fragility: tasks that block 3+ others but have no tests or are marked "backlog"

### 3. Version Conflict Detection

Compare dependency versions across repos:

- Check `pyproject.toml` for incompatible version ranges of shared dependencies (pydantic, mcp, fastmcp)
- Flag cases where one repo requires `pydantic>=2.10` and another requires `pydantic>=2.11`
- Flag pinned versions that differ across repos when they should match

### 4. Integration Dependency Mapping

For MCP server dependencies:

- Map which MCP tools call which other MCP tools
- Identify fragile integration points: tools that are called by 3+ other tools
- Identify unused MCP tool endpoints (tools exported but never called by any agent)

### 5. Release Blockers

Identify the critical path to release:

- Find the longest chain of `blocked_by` dependencies
- Report which tasks must complete before any release can happen
- Identify tasks on the critical path that have no owner or are stuck in "backlog"

## Output Format

```markdown
## Dependency Graph Analysis

### Circular Dependencies

| Cycle | Path | Repos Involved |
| ----- | ---- | -------------- |
| ❌    | ocd-X → adhd-Y → ocd-X | OCD, ADHD |
| ✅ (none) | No circular deps found | — |

### Stale Blockers

| Blocked Task | Blocked By | Status of Blocker | Severity |
| ------------ | ---------- | ----------------- | -------- |
| `ocd-12` | `ocd-11` | pending | 🔶 normal |
| `task-N` | `task-M` | done | 🔴 stale |

### Orphaned References

| Referencing File | Reference | Target | Issue |
| ---------------- | --------- | ------ | ----- |
| `tasks.json` | `cross_references` | `ai-99` | No task ai-99 exists |

### Dependency Health

| Task | Blocks N Tasks | Blocked By | Risk |
| ---- | -------------- | ---------- | ---- |
| `ocd-11` | 3 | none | 🔴 high (P1, blocks ocd-12/13/14) |

### Version Conflicts

| Dependency | Repo A Version | Repo B Version | Conflict |
| ---------- | -------------- | -------------- | -------- |
| pydantic | `>=2.10` (ADHD) | `>=2.11` (OCD) | 🔶 minor mismatch |

### Release Critical Path

Longest chain: `task-A → task-B → task-C → task-D` (4 levels)
Tasks on critical path: task-A (P1), task-B (P2), task-C (P1), task-D (P1)

### Summary

- Circular dependencies: N
- Stale blockers: N
- Orphaned references: N
- Version conflicts: N
- Critical path length: N
```

## Rules

- Only report — do not fix dependencies
- Distinguish between hard dependencies (blocks the task from starting) and soft dependencies (cross_references for awareness)
- A `cross_references` link is NOT a hard dependency — only `dependencies` and `blocked_by` are
- Use P1 priority as a risk multiplier: a P1 blocking 3 tasks is higher risk than a P3 blocking 5
- Do not flag done tasks that block other done tasks — only flag done→pending stale blockers
- Orphaned references are only a warning, not a critical issue — the target task may be in a different file
