# Development Foundation Strategy

Gap analysis against industry standards and a roadmap to close them.

______________________________________________________________________

## Current State Assessment

### Solid Foundation (don't touch)

| Area                       | What we have                                                               | Why it's solid                                         |
| -------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------ |
| **Agent coordination**     | 8 MCP servers, bus, sessions, work items                                   | Complete lifecycle: create → claim → transition → done |
| **Code quality**           | ruff, mypy, pytest, mdformat, gitleaks                                     | Standard tooling, enforced via pre-commit and CI       |
| **Multi-session protocol** | Bus coordination, meetings, partnerships, HITL                             | Production-proven, handles all collaboration modes     |
| **Knowledge persistence**  | KB Engine with TF-IDF search, daily log compilation                        | Survives context loss, cross-session accumulation      |
| **Security pipeline**      | Tier 0-4: input scan, blast-radius gating, output scan, audit              | Defense-in-depth with graceful degradation             |
| **Economic tracking**      | Value ledger, shadow pricing, session evaluation                           | Quantifies agent output, enables cost-aware decisions  |
| **Sub-agent architecture** | 18 specialized agents with tier-based model assignment, worktree isolation | Scales from haiku (cheap, fast) to sonnet (deep)       |

### Needs Work (iterate)

| Area            | Current                                                  | Problem                                                                                                                                                             |
| --------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Profiles**    | 6 files with inconsistent naming, empty env overrides    | `helper-16gb` falls back to cloud defaults while `helper-8gb` specifies local — backwards. `worker-gpu-8gb` has a GPU but uses cloud models. Hardcoded model names. |
| **Workflows**   | Design doc (WORKFLOWS.md) but no runtime                 | No `.claude/workflows/*.json`, no `/dispatch` skill, orchestrate skill not updated                                                                                  |
| **CI**          | Single `ci.yml` workflow                                 | No matrix testing, no cross-repo integration tests, no performance gates                                                                                            |
| **Proposals**   | 6 proposals, all unresolved                              | Decision bottleneck — operational patterns analysis flagged 16.7% meeting completion rate                                                                           |
| **Skills**      | 13 skills, mostly read-only or analysis                  | Only `ship`, `fix-ci`, `orchestrate` produce output — the rest are advisory                                                                                         |
| **Scaffolding** | `create_your-org` exists in tools/ but no skill wraps it | No standardized project generation, no templates for common project types                                                                                           |

### Missing Entirely (build)

| #   | Area                        | Industry Standard                                                                | Impact                                                                           |
| --- | --------------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 1   | **Testing pyramid**         | Unit + Integration + E2E + Mutation + Property-based + Benchmark                 | 46 tests across 8 servers (~6/server) is below professional threshold            |
| 2   | **Release management**      | Semantic versioning, automated changelog, release branches, deployment pipelines | No way to ship reliably, no rollback strategy                                    |
| 3   | **API documentation**       | Generated docs, OpenAPI specs, developer portal                                  | MCP servers are undocumented beyond source code                                  |
| 4   | **Performance governance**  | Latency budgets, token budgets, regression gates, profiling                      | No way to detect performance regressions                                         |
| 5   | **Environment management**  | Dev/staging/prod configs, secrets per environment, promotion gates               | Everything runs in one mode, no staging for MCP server changes                   |
| 6   | **Disaster recovery**       | Backups, recovery procedures, incident response playbooks                        | `~/.your-org/data/` has no backup, bus recovery is manual                        |
| 7   | **Definition of Done**      | Standardized completion criteria per task type                                   | Tasks end when the agent feels done, no objective gate                           |
| 8   | **Dependency governance**   | Lockfile verification, automated updates, compatibility matrix                   | Dependabot exists but no policy for update frequency or breaking change handling |
| 9   | **Technical debt tracking** | Debt budget, interest model, remediation sprints                                 | No systematic way to prioritize debt payoff                                      |

______________________________________________________________________

## Strategic Roadmap

### Phase 1 — Complete the Framework (now)

**Goal:** Turn the workflow design doc into working runtime. Fix the profiles.

| Step | What                                                              | Files                                             |
| ---- | ----------------------------------------------------------------- | ------------------------------------------------- |
| 1.1  | Redesign profiles with clean matrix (compute tier × model source) | `profiles/*.json` (replace all 6)                 |
| 1.2  | Create workflow JSONs for standard pipelines                      | `.claude/workflows/*.json` (4 files)              |
| 1.3  | Build `/dispatch` skill (tier-aware workflow executor)            | `.claude/skills/dispatch/SKILL.md`                |
| 1.4  | Update `orchestrate` skill to reference workflows and tier gates  | `.claude/skills/orchestrate/SKILL.md`             |
| 1.5  | Add trainer sub-agent template                                    | `.claude/skills/orchestrate/templates/trainer.md` |
| 1.6  | Resolve open proposals or close stale ones                        | `.claude/proposals/*.md`                          |

### Phase 2 — Testing Maturity (next)

**Goal:** Build a professional testing pyramid that catches regressions before they ship.

| Step | What                                                                | Tools/Approach                                                           |
| ---- | ------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| 2.1  | **Mutation testing** — Verify tests actually catch bugs             | `mutmut` or `pytest-mutagen` on each MCP server                          |
| 2.2  | **Property-based testing** — Find edge cases humans miss            | `hypothesis` for bus message formats, economy calculations               |
| 2.3  | **Integration test suite** — Multi-server scenarios                 | New `integration-test` sub-agent that spawns servers and runs end-to-end |
| 2.4  | **Benchmark suite** — Track latency and token consumption over time | `pytest-benchmark` for MCP tool calls                                    |
| 2.5  | **Coverage enforcement** — Per-module thresholds with CI gating     | Add to `pyproject.toml` and CI config                                    |
| 2.6  | **Fuzz testing** — Malformed inputs for bus and security servers    | `atheris` or `python-afl` for bus message parsing                        |

### Phase 3 — Release & Deployment (next)

**Goal:** Ship changes predictably with rollback capability.

| Step | What                                                                     | Files                           |
| ---- | ------------------------------------------------------------------------ | ------------------------------- |
| 3.1  | Define versioning strategy (semver for tools, calendar for workspace)    | `.claude/release-policy.md`     |
| 3.2  | Automate changelog generation from conventional commits                  | CHANGELOG.md + CI               |
| 3.3  | Create release workflow: tag → build → publish to PyPI/internal registry | `.github/workflows/release.yml` |
| 3.4  | Define environment promotion: dev → staging → prod with gates            | `.claude/environments/`         |
| 3.5  | Add rollback procedure for MCP server deployments                        | `.claude/deployment.md`         |

### Phase 4 — Performance & Observability (next+)

**Step:** Prevent regressions and understand system behavior.

| Step | What                                                                               |
| ---- | ---------------------------------------------------------------------------------- |
| 4.1  | Token budget per MCP tool call (alert when exceeded)                               |
| 4.2  | Latency budget per workflow stage (alert when stalled)                             |
| 4.3  | Profiling for botttleneck detection in bus, security pipeline                      |
| 4.4  | Structured logging across all MCP servers (already have `_config.py` with LOG_DIR) |
| 4.5  | Health dashboard showing all 8 servers + queue depths + error rates                |

### Phase 5 — Disaster Recovery & Incident Response (future)

**Step:** Survive failures gracefully.

| Step | What                                                                          |
| ---- | ----------------------------------------------------------------------------- |
| 5.1  | Backup strategy for `~/.your-org/data/` (ledger, KB, bus state)               |
| 5.2  | Bus recovery procedure when bus.jsonl is corrupted                            |
| 5.3  | Incident response playbooks: P0 (data loss), P1 (service down), P2 (degraded) |
| 5.4  | Postmortem template with root cause analysis and action items                 |

______________________________________________________________________

## Design Decisions

### Profile Redesign (Phase 1)

Replace the current 6 inconsistent profiles with a clean 2×3 matrix:

```text
                cloud           hybrid          local
───────         ─────           ──────          ─────
worker          worker:cloud    —               worker:local
helper          helper:cloud    helper:hybrid   helper:local
trainer         trainer:cloud   trainer:hybrid  trainer:local
```

Each profile is a JSON file that explicitly declares:

```json
{
  "tier": "helper",
  "model_source": "hybrid",
  "vram_gb": 16,
  "models": {
    "opus": "qwen3.5:14b",
    "sonnet": "qwen3.5:14b",
    "haiku": "deepseek-v4-flash:cloud",
    "subagent": "qwen3-coder-next:cloud"
  },
  "capabilities": {
    "training": false,
    "fine_tuning": false
  },
  "autonomy": "gated",
  "max_concurrent_subagents": 3
}
```

Profile file names: `worker-cloud.json`, `worker-local.json`, `helper-cloud.json`,
`helper-hybrid.json`, `helper-local.json`, `trainer-cloud.json`, `trainer-hybrid.json`,
`trainer-local.json`.

### Testing Pyramid Targets (Phase 2)

| Layer       | Tool                | Target Coverage      | Per MCP Server                               |
| ----------- | ------------------- | -------------------- | -------------------------------------------- |
| Unit        | pytest              | 90%+ line            | Every tool function tested in isolation      |
| Integration | pytest + subprocess | 80%+                 | Server starts, tools respond, errors handled |
| Mutation    | mutmut              | 80%+ survival killed | Tests actually assert behavior               |
| Property    | hypothesis          | Key schemas          | Bus messages, ledger entries, work items     |
| Benchmark   | pytest-benchmark    | Tracked over time    | Tool call latency + token counts             |
| Fuzz        | python-afl          | Bus parser           | Malformed message handling                   |

### Definition of Done (Phase 1)

Every work item must meet all criteria before transitioning to `done`:

- [ ] Code compiles without errors (mypy strict)
- [ ] Linter passes (ruff, no warnings)
- [ ] Formatter applied (ruff format)
- [ ] Tests pass (pytest, no skips)
- [ ] Coverage threshold met (90%+ for new code)
- [ ] API docs generated (if public interface changed)
- [ ] Changelog entry added (if user-facing change)
- [ ] CI passes on remote branch
- [ ] Integration tests pass (if cross-server change)
- [ ] KB updated (if new error pattern or decision)

______________________________________________________________________

## Missing Sub-Agents & Skills

From the gap analysis, new agents and skills needed:

### New Sub-Agents

| Agent                | Model  | Purpose                                                               | Phase |
| -------------------- | ------ | --------------------------------------------------------------------- | ----- |
| `integration-tester` | sonnet | Spawn MCP servers, run end-to-end tests, verify cross-server behavior | 2     |
| `benchmark-runner`   | haiku  | Run benchmark suite, compare against baselines, report regressions    | 2     |
| `release-manager`    | sonnet | Version bump, changelog update, tag, publish                          | 3     |
| `incident-responder` | sonnet | Diagnose production issues, apply hotfixes, write postmortem          | 5     |

### New Skills

| Skill                | Purpose                                                    | Phase     |
| -------------------- | ---------------------------------------------------------- | --------- |
| `/dispatch`          | Tier-aware workflow execution (from WORKFLOWS.md)          | 1         |
| `/profile <name>`    | Switch active profile mid-session                          | 1         |
| `/scaffold <type>`   | Generate new project from template (wraps create_your-org) | 2         |
| `/benchmark [scope]` | Run benchmarks and report against baselines                | 2         |
| \`/release \[patch   | minor                                                      | major\]\` |

______________________________________________________________________

## Risk Assessment

| Risk                                                | Likelihood | Impact                 | Mitigation                                                                       |
| --------------------------------------------------- | ---------- | ---------------------- | -------------------------------------------------------------------------------- |
| Scope creep — try to do all phases at once          | High       | Delays everything      | Phases are sequential with hard gates; Phase 1 must ship before Phase 2 starts   |
| Profile redesign breaks existing sessions           | Medium     | Sessions misconfigured | Keep old profiles during transition period; migration script for state.json      |
| Testing phase slows development velocity            | Medium     | Team skips tests       | Start with mutation + property on critical servers only (bus, security, economy) |
| Release automation encourages too-frequent releases | Low        | Version churn          | Define release cadence: weekly for tools, monthly for workspace config           |
