# Context Preservation via Sub-Agents — STRICT

## weight: 6 category: essential description: Preserve main session context by delegating work to sub-agents

**Delegate to sub-agents when tasks are large, parallelizable, or context-consuming.**

## When to Spawn

1. **Independent research** — audits, greps, doc searches not needing full context.
1. **Parallel execution** — multiple files to edit or services to verify.
1. **Lengthy tasks** — exceeding ~50% of context window or >10 tool calls.
1. **Meeting attendance** — per coordination rules.
1. **Large outputs** — log parsing, bulk reads, data ingestion.

## Invariants

1. **Role inheritance.** Sub-agents inherit Master's role or below. Never spawn above.
1. **Context isolation.** Sub-agents don't share conversation history. Provide self-contained prompts.
1. **Result condensation.** Return file paths, line numbers, diffs, conclusions — not raw dumps.
1. **No recursive explosion.** Max 2 levels deep (Master → sub → sub-sub) without human approval.
1. **Failure isolation.** Time-box sub-agent work. Proceed with partial results if one fails.

## Main Agent Retains

- Architectural decisions and cross-file consistency
- Final code review and sign-off
- Bus coordination and human-facing communication

## Main Agent Offloads

- Implementation → coder-worker sub-agents
- Research → explore/researcher sub-agents
- Routine checks (lint, tests) → error-handler/test-writer sub-agents
- Long-running monitoring → background sub-agents

## Enforcement

- This rule is loaded after `00-capability-boundaries.md` because sub-agent role gating is a prerequisite.
- Violations (running large tasks in the main session context when a sub-agent was available) must be logged.
