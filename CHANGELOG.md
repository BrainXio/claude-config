# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-05-22

### Added

- **AGENTS.md** — comprehensive setup guide with provider decision tree
  (Anthropic API / Ollama Cloud / Local Ollama), VRAM-aware model
  selection, and full bootstrap instructions.
- **SETUP-OLLAMA.md** — hardware-aware local Ollama setup with VRAM
  detection, quantization selection, and human confirmation flow.
- **PLAN.md** — pre-commit hook improvement plan covering uvx-based
  tooling, broader attribution guard, JSON validation, and porting
  `claude-standards-guard` logic.
- **workflows/** — JSON workflow definitions (`document-only`,
  `feature-implement`, `fix-bug`, `train-model`) merged from the
  now-deleted `claude-workflows` repository.
- **rules** — conventional commits (`20-essential/01`), branch strategy
  (`20-essential/03`), and markdown formatting (`30-standards/01`)
  rules.
- **agents** — `kb-markdown-formatter` agent for mdformat with
  required plugins.
- **`.githooks/commit-msg`** — blocks AI attribution patterns in commit
  messages.
- **`.githooks/pre-commit`** — uvx-based quality gate with markdown
  formatting (mdformat + frontmatter + GFM), YAML linting, JSON
  validation, attribution guard, standards guard (delegated to
  `claude-standards-guard`), and multi-language auto-detection
  (Python, Rust, Go, JavaScript/TypeScript, Shell).

### Changed

- **Settings files** moved from repo root into `settings/` directory
  and renamed to drop the redundant `settings.` prefix (e.g.
  `settings/settings.core.json` → `settings/core.json`).
- **model.json** is now provider-specific — mandatory for Anthropic API
  only; explicitly skipped for both Ollama Cloud and Local Ollama.
- **Permissions** — `Run(...)` deny entries corrected to `Bash(...)`
  with proper `command: *` patterns. `Run` is not a valid Claude Code
  tool name.
- **Sandbox** — removed `failIfUnavailable` property (rejected by the
  settings JSON schema).
- **Pre-commit hook** — replaced `command -v` tool checks with `uvx`
  invocations; attribution guard now scans all staged `.md`, `.yaml`,
  `.yml`, and `.json` files (was only 3 hardcoded files); added JSON
  validation and standards guard delegation.
- **`.gitignore`** — collapsed 12 individual `!settings.*.json`
  allowlist entries into `!settings/` + `!settings/**`.

### Removed

- **`claude-workflows`** repository — all workflow definitions now
  live in `workflows/` within this repository.
- **`failIfUnavailable`** from sandbox settings (schema violation).

### Added

- **AGENTS.md** — comprehensive setup guide with provider decision tree
  (Anthropic API / Ollama Cloud / Local Ollama), VRAM-aware model
  selection, and full bootstrap instructions.
- **SETUP-OLLAMA.md** — hardware-aware local Ollama setup with VRAM
  detection, quantization selection, and human confirmation flow.
- **PLAN.md** — pre-commit hook improvement plan covering uvx-based
  tooling, broader attribution guard, JSON validation, and porting
  `claude-standards-guard` logic.
- **workflows/** — JSON workflow definitions (`document-only`,
  `feature-implement`, `fix-bug`, `train-model`) merged from the
  now-deleted `claude-workflows` repository.
- **rules** — conventional commits (`20-essential/01`), branch strategy
  (`20-essential/03`), and markdown formatting (`30-standards/01`)
  rules.
- **agents** — `kb-markdown-formatter` agent for mdformat with
  required plugins.
- **`.githooks/commit-msg`** — blocks AI attribution patterns in commit
  messages.
- **`.githooks/pre-commit`** — uvx-based quality gate with markdown
  formatting (mdformat + frontmatter + GFM), YAML linting, JSON
  validation, attribution guard, standards guard (delegated to
  `claude-standards-guard`), and multi-language auto-detection
  (Python, Rust, Go, JavaScript/TypeScript, Shell).

### Changed

- **Settings files** moved from repo root into `settings/` directory
  and renamed to drop the redundant `settings.` prefix (e.g.
  `settings/settings.core.json` → `settings/core.json`).
- **model.json** is now provider-specific — mandatory for Anthropic API
  only; explicitly skipped for both Ollama Cloud and Local Ollama.
- **Permissions** — `Run(...)` deny entries corrected to `Bash(...)`
  with proper `command: *` patterns. `Run` is not a valid Claude Code
  tool name.
- **Sandbox** — removed `failIfUnavailable` property (rejected by the
  settings JSON schema).
- **Pre-commit hook** — replaced `command -v` tool checks with `uvx`
  invocations; attribution guard now scans all staged `.md`, `.yaml`,
  `.yml`, and `.json` files (was only 3 hardcoded files); added JSON
  validation and standards guard delegation.
- **`.gitignore`** — collapsed 12 individual `!settings.*.json`
  allowlist entries into `!settings/` + `!settings/**`.

### Removed

- **`claude-workflows`** repository — all workflow definitions now
  live in `workflows/` within this repository.
- **`failIfUnavailable`** from sandbox settings (schema violation).
