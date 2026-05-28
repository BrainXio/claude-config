# Standards Checklist — 8B Optimized

**Weight: 9-10**

## Weight 9: Alphabetical Ordering

Sort by primary key (first column) for:

- Tables, lists, directory listings
- Imports (enforced by ruff)
- Config entries, package manifests, `__all__`

Exceptions: dependency order matters, priority is the point (severity tables), chronological narrative (changelogs).

## Weight 10: Markdown Formatting

**Required command:**

```bash
uvx --with mdformat-frontmatter --with mdformat-gfm mdformat <files>
```

Without plugins, mdformat corrupts YAML frontmatter (`---` → `___`).

Applies to: agents, rules, skills, docs, README.

## Python Rules

- Use `uv` over pip/poetry
- Always use type hints
- Ruff + mypy strict
- No bare except
