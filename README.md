# Claude Config

Claude Code framework configuration: rules, agents, skills, and granular settings.

## What It Provides

| Category     | Contents                                                                                                              |
| ------------ | --------------------------------------------------------------------------------------------------------------------- |
| **Settings** | `settings.*.json` files compose into `settings.json` (core, env, hooks, permissions, sandbox, model, env-ollama-\*)   |
| **Agents**   | 19 sub-agent definitions (coder-worker, bug-hunter, test-writer, docs-writer, etc.)                                   |
| **Rules**    | Hierarchical rules: critical (human sovereignty, no secrets), essential (bus, KB engine), standards (MCP tools first) |
| **Skills**   | dispatch, orchestrate, code-audit, ship, fix-ci, schedule-tasks, inception-monitor, etc.                              |
| **Docs**     | Framework docs: AGENTS.md, PROTOCOLS.md, TOOLS.md, WORKFLOWS.md                                                       |

## Settings Files

| File                                                                        | Purpose                                                        |
| --------------------------------------------------------------------------- | -------------------------------------------------------------- |
| `settings.core.json`                                                        | Schema, autoMemory, awaySummary, cleanupPeriod                 |
| `settings.env.json`                                                         | Claude env vars (log level, telemetry, timeouts)               |
| `settings.env-ollama-gpu-8gb.json`                                          | Ollama GPU 8GB model aliases                                   |
| `settings.env-ollama-gpu-16gb.json`                                         | Ollama GPU 16GB model aliases                                  |
| `settings.env-ollama-gpu-24gb.json`                                         | Ollama GPU 24GB model aliases                                  |
| `settings.env-ollama-cloud-max.json`                                        | Ollama cloud-max tier model aliases                            |
| `settings.env-ollama-cloud-pro.json`                                        | Ollama cloud-pro tier model aliases                            |
| `settings.env-ollama-cloud-free.json`                                       | Ollama cloud-free tier model aliases                           |
| `settings.hooks.json`                                                       | SessionStart, PreCompact, SessionEnd, PreToolUse hooks via uvx |
| `settings.permissions.json`                                                 | Allowed/denied Read/Write/Run paths                            |
| `settings.sandbox.json`                                                     | Sandbox: excludedCommands, filesystem, network isolation       |
| `settings.model.json`                                                       | Model config (sonnet-4-6, effortLevel high)                    |
| `settings.statusline.json`, `settings.ui.json`, `settings.attribution.json` | Statusline, UI, attribution                                    |

## Usage

Copy or symlink into `.claude/` in a project workspace:

```bash
cp -r rules/ agents/ skills/ docs/ settings.*.json /path/to/project/.claude/
```

## Contributing

See BrainXio/.github/CONTRIBUTING.md for commit style, PR requirements, and organization-wide conventions.

For claude-config specifically:

- Rules, agents, and skills use YAML frontmatter followed by markdown body
- Settings files compose: `settings.core.json` is the base, others layer on top
- Run `uvx --with mdformat-frontmatter --with mdformat-gfm mdformat --check .` before committing

## Related Repositories

- `BrainXio/claude-cli` — Lifecycle hooks and CLI entry points
