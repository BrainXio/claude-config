---
weight: 9
category: standards
description: Prefer strict alphabetical ordering for lists, tables, and ordered collections
---

# Alphabetical Ordering

**Prefer alphabetical order for lists, tables, and ordered collections.**

## Rationale

Alphabetical ordering is the only sort order that requires zero context to understand. Chronological, priority, and importance-based orders drift silently as the codebase evolves. Alphabetical order is self-maintaining and prevents the cognitive load of guessing why item X appears before item Y.

## What This Covers

- **Tables** — sort by primary key (first column)
- **Lists** — sort by item text
- **Directory listings** — sort by path
- **Imports** — enforced by ruff
- **Config entries** — keys, server names, dependencies
- **Package manifests** — `__all__`, scripts, optional-dependencies

## Exceptions

1. **Dependency order** — step N must precede N+1
1. **Priority is the point** — severity tables (critical → info)
1. **Chronological narrative** — changelogs, history

When overriding, add a brief note explaining why.

## Enforcement

- `ruff check .` catches import ordering
- Prefer `sorted()` or `sorted(..., key=str.lower)` in code
