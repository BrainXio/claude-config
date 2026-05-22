# Orchestrate Skill

Decompose work into role-specific sub-agent tasks, dispatch them in parallel
when possible, and aggregate results. The main session is the sole bus
participant — sub-agents are extensions of the main session, not independent
sessions.

## Role Taxonomy

| Role             | subagent_type        | Responsibility                        | Context Scope                                     |
| ---------------- | -------------------- | ------------------------------------- | ------------------------------------------------- |
| Implementer      | `coder-worker`       | Write code, make changes              | Focused: file paths + change spec                 |
| Tester           | `test-writer`        | Write/run tests, verify coverage      | Focused: implementation files + expected behavior |
| Reviewer         | `bug-hunter`         | Find bugs, dead code, inconsistencies | Broad: target files + patterns                    |
| Researcher       | `Explore`            | Read-only codebase exploration        | Broad: search queries + scope                     |
| Documenter       | `docs-writer`        | Update docs, create articles          | Focused: topic + target files                     |
| Planner          | `Plan`               | Design implementation approach        | Broad: requirements + constraints                 |
| Auditor          | `complexity-reducer` | Simplify overly complex code          | Focused: target files                             |
| Meeting delegate | `general-purpose`    | Attend bus meetings, vote, report     | Meeting context + positions                       |

## Dispatch Rules

### Workflow Pipeline

For standard work pipelines, use the `/dispatch` skill which reads workflow
definitions from `.claude/workflows/*.json`:

| Workflow            | File                                       | Stages                                                   |
| ------------------- | ------------------------------------------ | -------------------------------------------------------- |
| `feature-implement` | `.claude/workflows/feature-implement.json` | research → plan → implement → audit → test → docs → ship |
| `fix-bug`           | `.claude/workflows/fix-bug.json`           | audit → implement → test → ship                          |
| `train-model`       | `.claude/workflows/train-model.json`       | research → prep → train → eval → docs (tier-gated)       |
| `document-only`     | `.claude/workflows/document-only.json`     | research → write → review → ship                         |

```bash
/dispatch <workflow-name> [--stage <name>] [--dry-run]
```

The dispatch skill handles:

- Tier gate evaluation (always vs. tier >= N)
- Dependency-ordered stage execution
- Parallel dispatch up to max_concurrent
- Worktree isolation for implementation stages
- Failure retry per workflow retry_policy

### When to use sub-agents vs. direct work

| Work type                              | Approach                                               | Why                                    |
| -------------------------------------- | ------------------------------------------------------ | -------------------------------------- |
| Single-file edit (< 20 lines)          | Direct work                                            | Overhead exceeds benefit               |
| Multi-file implementation              | dispatch feature-implement                             | Tier-gated, worktree-isolated workflow |
| Bug fix                                | dispatch fix-bug                                       | Audit-first streamlined pipeline       |
| Independent features (no file overlap) | coder-worker x N in parallel                           | Each gets own worktree                 |
| Testing (after implementation)         | test-writer                                            | Independent perspective                |
| Code audit                             | dispatch fix-bug --stage audit                         | Full 7-auditor parallel run            |
| Research/exploration                   | Explore or dispatch feature-implement --stage research | Read-only, safe in parallel            |
| Documentation                          | docs-writer or dispatch document-only                  | Worktree-isolated, mdformat enforced   |
| Meeting attendance (focused)           | general-purpose                                        | Main session continues working         |

### Parallelism rules

1. **Implementers always use worktrees.** Each coder-worker gets `--isolation worktree` to avoid file conflicts.
1. **Only dispatch parallel coders on independent file sets.** Check for file overlap before dispatching.
1. **Testers and reviewers run AFTER implementers complete.** Never test code that hasn't been written yet.
1. **Researchers and documenters can run in parallel with implementers** (read-only or different files).
1. **Maximum 3 concurrent sub-agents** to avoid rate limits and context fragmentation.

### Dispatch sequence (manual)

For manual dispatch without the workflow runner:

```bash
1. Research (Explore)          — parallel with everything
2. Plan (Plan)                 — if architectural decisions needed
3. Implement (coder-worker)    — sequential or parallel with worktrees
4. Test (test-writer)          — after implementation
5. Review (bug-hunter)         — after implementation
6. Document (docs-writer)      — parallel with test/review
7. Commit (main session)       — after all sub-agents complete
```

## Meeting Delegation

| Meeting type                             | Who attends           | Why                                         |
| ---------------------------------------- | --------------------- | ------------------------------------------- |
| Focused (specific topic, < 30 min)       | Sub-agent delegate    | Main session continues implementation       |
| Vision (architecture, roadmap, > 30 min) | Main session directly | Foundational decisions require full context |
| Fire alarm (P0/P1)                       | Main session directly | Urgency requires immediate response         |

The sub-agent delegate receives:

- Meeting context (agenda, relevant bus messages)
- Approval authority for non-foundational decisions
- Voting position for the main session (majority rules at odd quorums)
- Escalation instructions for foundational decisions

## Failure Handling

### Economic value evaluation

When a sub-agent fails (rate limit, error, poor results):

1. **Check economic value** — Evaluate:

   - What was the expected value of this task?
   - What is the cost of retrying (tokens, time)?
   - What is the cost of not completing this task?

1. **Decision matrix**

| Value/Cost                  | Action                              |
| --------------------------- | ----------------------------------- |
| High value, low retry cost  | Retry with refined instructions     |
| High value, high retry cost | Main session does the work directly |
| Low value, any cost         | Skip the task, post status on bus   |
| Unknown value               | Evaluate against sprint priorities  |

1. **Retry budget** — Maximum 2 retries per sub-agent task. After 2 failures:

   - If main session can do it: do it directly
   - If main session can't: escalate to HITL via bus decision

1. **Never retry with the same instructions.** If a sub-agent fails, diagnose the failure first:

   - Rate limit → wait and retry later, or do directly
   - Context overflow → split the task smaller
   - Wrong output → clarify the prompt, be more specific
   - Error/crash → check the error, fix the prompt

### Escalation to HITL

Only escalate when:

- The main session cannot make a decision (ambiguous economic value)
- The task requires foundational decision authority
- Multiple retry strategies have failed
- The work item is blocked and cannot proceed

Use `claim_decision` on the bus to escalate.

## Prompt Templates

Sub-agent prompts follow a standard structure. Templates are in
`.claude/skills/orchestrate/templates/`:

```text
templates/
  implementer.md
  tester.md
  reviewer.md
  researcher.md
  documenter.md
  planner.md
  meeting-delegate.md
```

Each template has these sections:

1. **Role definition** — Who you are and your constraints
1. **Task specification** — Exactly what to do, with file paths
1. **Context injection** — Relevant schemas, patterns, conventions
1. **Output format** — What to return
1. **Constraints** — What NOT to do (no commits, no bus, no MCP tools)

## Actions

### status

Show current orchestration state: active sub-agents, pending work items,
and dispatch recommendations.

### decompose `<sprint-item>`

Read the sprint item and decompose it into sub-agent tasks with role
assignments, dependencies, and parallelism hints.

### dispatch `<role> <task>`

Create a sub-agent prompt from the template and dispatch it via the Agent
tool. The main session waits for the result.

### `<work-description>`

Full orchestration cycle: decompose the work, dispatch sub-agents in
dependency order, aggregate results, and commit.
