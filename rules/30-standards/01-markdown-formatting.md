# Markdown Formatting — STRICT

## weight: 10 category: standards description: Always use mdformat with mdformat-frontmatter and mdformat-gfm plugins

**Never run plain `mdformat` without plugins. It destroys YAML frontmatter.**

## Required Command

```bash
uvx --with mdformat-frontmatter --with mdformat-gfm mdformat <files>
```

## Why Plugins Are Mandatory

Without `mdformat-frontmatter`, mdformat interprets `---` YAML frontmatter delimiters as
markdown thematic breaks and converts them to `___` or `______________________________________________________________________`.
This corrupts agent, rules, and skills files that rely on frontmatter metadata.

The `mdformat-gfm` plugin provides GitHub Flavored Markdown support (tables, task lists,
strikethrough) that the project uses.

## What to Format

- Agent files (`agents/*.md`) — YAML frontmatter + markdown body
- Rules (`rules/**/*.md`) — may have frontmatter
- Skills (`skills/**/SKILL.md`) — may have frontmatter
- Docs (`docs/**/*.md`) — standard markdown
- README, CONTRIBUTING, etc.

## CI Enforcement

The CI workflow runs `mdformat --check .` with both plugins installed. Any file not
formatted with `mdformat-frontmatter` and `mdformat-gfm` will fail the gate.

## Recovery from Corruption

If frontmatter has been corrupted (horizontal rules instead of `---`):

1. Restore files from the last known-good commit: `git checkout <good-commit> -- agents/ rules/ skills/`
1. Re-format with plugins: `uvx --with mdformat-frontmatter --with mdformat-gfm mdformat agents/ rules/ skills/`
1. Verify: `uvx --with mdformat-frontmatter --with mdformat-gfm mdformat --check agents/ rules/ skills/`

## Anti-patterns

- Never run plain `uvx mdformat` or `mdformat` without plugins
- Never install mdformat without `mdformat-frontmatter` and `mdformat-gfm`
- Never skip mdformat --check in CI
- Never manually edit frontmatter delimiters — use mdformat with plugins
