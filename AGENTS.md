<!-- pre-commit: bypass-attribution -->

# AGENTS.md

claude-config is the configuration backbone of the Claude Code extension
framework. It provides settings, rules, agent definitions, skills, and
hooks that together form a bootstrapped `.claude/` directory.

## Quick Start Decision Tree

Ask the human two questions to determine the setup path:

1. **"Anthropic API or Ollama?"**
   - **Anthropic API** → standard setup, no extra files needed.
   - **Ollama** → ask the next question.
1. **"Ollama Cloud or local Ollama?"**
   - **Ollama Cloud** → copy `settings/env-ollama-cloud.json`.
   - **Local Ollama** → a hardware check is required before setup.
     See [SETUP-OLLAMA.md](SETUP-OLLAMA.md) for the full procedure
     (VRAM detection, quantization selection, human confirmation).

## Directory Layout

| Source            | Target                  | Purpose                 |
| ----------------- | ----------------------- | ----------------------- |
| `settings/*.json` | `.claude/settings.json` | Config layers           |
| `rules/`          | `.claude/rules/`        | Agent rules             |
| `agents/`         | `.claude/agents/`       | Sub-agent definitions   |
| `skills/`         | `.claude/skills/`       | Slash-command skills    |
| `.githooks/`      | `.git/hooks/`           | Pre-commit quality gate |

## Setup

### Step 1: Compose settings files

Settings files live under `settings/` and are layered parts of a single
`settings.json` (or `settings.local.json`). Copy them into `.claude/` in
the order shown below. Later files override earlier ones. Do not
reorder.

**Mandatory** — always copy these files:

```text
settings/core.json          →  Schema, autoMemory, cleanup period
settings/env.json           →  Environment, timeouts, telemetry
settings/permissions.json   →  Read/Write/Run allow and deny rules
settings/sandbox.json       →  Sandbox isolation, filesystem/network
settings/hooks.json         →  SessionStart, SessionEnd, PreCompact,
                               PreToolUse hooks
settings/ui.json            →  Editor mode, progress bars, auto-scroll
settings/statusline.json    →  Status line command
```

**Provider choice:**

- **Anthropic API** — also copy `settings/model.json` after
  `settings/env.json`. This sets the model to `claude-sonnet-4-6` with
  thinking enabled at high effort. No other additional files needed.
  Skip to Step 2.
- **Ollama Cloud** — do NOT copy `settings/model.json`. The
  `:cloud`-tagged models in `settings/env-ollama-cloud.json` handle
  model selection via environment variables
  (`ANTHROPIC_DEFAULT_OPUS_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL`,
  `ANTHROPIC_DEFAULT_HAIKU_MODEL`, `CLAUDE_CODE_SUBAGENT_MODEL`).
  Copy `settings/env-ollama-cloud.json` after `settings/env.json`.
- **Local Ollama** — do NOT copy `settings/model.json` (same reason
  as Cloud). Do NOT copy the env file yet. Follow the hardware check
  procedure in [SETUP-OLLAMA.md](SETUP-OLLAMA.md) first. If the
  hardware is sufficient (or the human confirms despite a warning),
  copy `settings/env-ollama-local.json` after `settings/env.json`,
  patching model tags if a different quantization was chosen.

All Ollama variants set `ANTHROPIC_BASE_URL` to `http://localhost:11434`
and `ANTHROPIC_AUTH_TOKEN` to `ollama`.

**Attribution** — skip `settings/attribution.json`. It is empty by
default and only needed if the human explicitly requests commit/PR
attribution.

### Step 2: Copy rules

Copy the entire `rules/` directory to `.claude/rules/`. Rules are loaded
in numeric order:

| Directory              | Wt   | Category                                     |
| ---------------------- | ---- | -------------------------------------------- |
| `00-no-attribution.md` | 0    | No AI attribution                            |
| `10-critical/`         | 1–3  | Sovereignty, no secrets, boundaries, vision  |
| `20-essential/`        | 4–6  | Conv. commits, context, anti-overengineering |
| `30-standards/`        | 9–10 | MCP tools first, formatting, ordering        |
| `python.md`            | —    | uv, type hints, ruff+mypy strict             |

Lower weight = higher priority. Critical rules must never be violated.

### Step 3: Copy agents

Copy `agents/` to `.claude/agents/`. Each file is a sub-agent definition
with YAML frontmatter followed by a markdown prompt body. Key agents:

| Agent                   | Model  | Purpose                            |
| ----------------------- | ------ | ---------------------------------- |
| `core-researcher`       | haiku  | Read-only codebase exploration     |
| `dev-coder-worker`      | sonnet | Isolated implementation (worktree) |
| `dev-code-reviewer`     | sonnet | Design/maintainability review      |
| `ops-security-engineer` | sonnet | Threat modeling                    |
| `ops-standards-guard`   | haiku  | Block forbidden doc content        |

### Step 4: Copy skills

Copy `skills/` to `.claude/skills/`. Each skill is a directory containing
a `SKILL.md` file with YAML frontmatter and a markdown prompt body.

### Step 5: Hook dependencies

Hooks invoke commands from the `claude-cli` package via `uvx`. No
separate install step is needed — `uvx` resolves and caches the package
on first use:

```bash
uvx --from git+https://github.com/BrainXio/claude-cli.git <command>
```

Required commands (defined in `claude-cli`'s `pyproject.toml`):

| Command                     | Hook         | Purpose                        |
| --------------------------- | ------------ | ------------------------------ |
| `claude-bootstrap`          | SessionStart | Detect role/VRAM, sync docs    |
| `claude-session-start`      | SessionStart | Inject KB context, mode, log   |
| `claude-check-model-vision` | SessionStart | Verify vision capability       |
| `claude-pre-compact`        | PreCompact   | Capture pre-compact transcript |
| `claude-session-end`        | SessionEnd   | Flush transcript to daily log  |
| `claude-standards-guard`    | PreToolUse   | Validate Edit/Write calls      |

### Step 6: Git hooks

The `.githooks/` directory contains lightweight, generic hooks:

- **`pre-commit`** — runs before every commit and enforces formatting and
  linting:
  - **Markdown formatting** — via `mdformat` with the required
    `mdformat-frontmatter` and `mdformat-gfm` plugins.
  - **YAML linting** — via `yamllint`.
  - **JSON validation** — via `jq`.
  - **Standards enforcement** — delegated to `claude-standards-guard` from
    `claude-cli` (see [Hook dependencies](#step-5-hook-dependencies)), which
    handles attribution, philosophy sludge, manifesto tone, phantom repo
    links, and workflow security checks. No BrainXio-specific rules are
    hard-coded in the hook itself.
- **`commit-msg`** — runs after the commit message is written and blocks
  AI attribution patterns (for example, `Co-authored-by`, `Generated by AI`,
  `Written by Claude/ChatGPT/Copilot/Gemini`).

**When to set up:**

- If a `.git` directory is present in the target project, advise the
  human: **"This project is a git repository. Set up the pre-commit
  hook to enforce quality gates before every commit?"**
- If you're unsure whether the human wants hooks, ask. Never silently
  skip hook setup when `.git` is present.
- If no `.git` directory exists, skip this step — hooks cannot be
  activated without git.

**How to activate:**

```bash
git config core.hooksPath .githooks
```

This points git at the `.githooks/` directory instead of the default
`.git/hooks/`. Verify with a dry run:

```bash
.githooks/pre-commit
```

## Bootstrap Sequence

When a session starts, SessionStart hooks fire in order:

1. **`claude-bootstrap`** — Detects hardware role and available VRAM,
   pulls models if needed, synchronizes documentation, writes bootstrap
   state files.
1. **`claude-session-start`** — Injects knowledge base context, current
   operational mode, and references to today's daily log.
1. **`claude-check-model-vision`** — Queries the active model for vision
   capability and writes `model-capabilities.json` so the vision
   safeguard rule can enforce correct model routing.

During the session:

- **PreToolUse** (`claude-standards-guard`) fires on every Edit or Write
  call and blocks forbidden content patterns.
- **PreCompact** (`claude-pre-compact`) saves the transcript before
  automatic context compaction.

When the session ends, **SessionEnd** (`claude-session-end`) flushes the
conversation transcript to the daily log.

## Key Conventions

- **Sandbox is on by default** (`enabled: true`). All bash commands run
  sandboxed unless explicitly excluded (ssh, gpg, gh, docker, sudo).
- **Secrets are blocked** by permissions deny rules: no `.env` files,
  `.pem`/`.key`/`.crt` files, or paths containing `secrets`.
- **Sudo and Docker are denied** in both permissions and sandbox
  exclusions.
- **Conventional commits are required** (rule weight 4): `feat:`, `fix:`,
  `chore:`, `docs:`, `refactor:`, `test:`, `ci:`, etc.
- **No AI attribution** in commits or documentation (rule weight 0):
  blocks patterns like "Co-authored-by", "Generated by",
  "AI-generated", "Written by (Claude|ChatGPT|Copilot|Gemini)".
- **Sub-agents are the norm** for large or parallelizable tasks. Max 2
  levels deep (master → sub-agent → sub-sub-agent). Dispatch up to 3
  concurrent sub-agents.
- **Settings compose in a fixed order**. Do not reorder the settings
  files. Core must be first, attribution last.
