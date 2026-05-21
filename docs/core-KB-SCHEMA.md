# AGENTS.md — Knowledge Base Schema & Agent Guide

**Schema reference and collaboration instructions for the personal knowledge base system.**

This file is read by the LLM compiler (`scripts/compile.py`) and by Claude hooks to understand how to structure and maintain the knowledge base.

---

## Directory Layout (Current)

```bash
~/.claude/                          # User tools & agent files
├── AGENTS.md                       # ← This file (schema + instructions)
├── scripts/
│   ├── compile.py                  # Daily → knowledge
│   ├── query.py                    # Ask questions
│   ├── lint.py                     # Health checks
│   ├── flush.py                    # Background memory extraction
│   ├── config.py                   # Paths & settings
│   └── utils.py
├── hooks/
│   ├── session-start.py            # Injects KB context
│   ├── session-end.py              # Extracts conversation → daily log
│   └── pre-compact.py              # Pre-compaction safety
└── settings.json                   # Claude hook configuration

~/knowledge-base/                   # ← Your ROOT_DIR (project content)
├── daily/                          # Raw conversation logs (git-ignored)
├── knowledge/
│   ├── index.md                    # Master catalog
│   ├── log.md                      # Build & query history
│   ├── concepts/                   # Atomic knowledge
│   ├── connections/                # Cross-concept insights
│   └── qa/                         # Filed answers
├── reports/                        # Lint & health reports
└── .gitignore
```

---

## Structural Files

### `knowledge/index.md` — Master Catalog

The primary retrieval document. The LLM **always reads this first**.

```markdown
# Knowledge Base Index

| Article                          | Summary                          | Compiled From              | Updated    |
|----------------------------------|----------------------------------|----------------------------|------------|
| [[concepts/supabase-rls]]        | Row Level Security in Supabase   | daily/2026-04-01.md        | 2026-04-03 |
```

### `knowledge/log.md` — Build & Activity Log

Append-only chronological record.

```markdown
# Build Log

## [2026-05-05T21:15:42+00:00] compile | 2026-05-05.md
- Source: daily/2026-05-05.md
- New concepts: 3
- Updated concepts: 2
- New connections: 1
- QA filed: 0
```

---

## Article Types & Templates

### 1. Concept Articles (`knowledge/concepts/`)

Atomic, focused pieces of knowledge.

> **Frontmatter**

```yaml
---
title: "Concept Name"
aliases: ["short-name", "abbr"]
tags: ["area", "topic"]
sources:
  - "daily/2026-05-01.md"
  - "daily/2026-05-03.md"
created: 2026-05-01
updated: 2026-05-05
---
```

> **Body**

```markdown
# Concept Name

[2–4 sentence clear definition/explanation]

## Key Points
- Bullet 1 (self-contained)
- Bullet 2

## Details
Deeper explanation in encyclopedia style.

## Related Concepts
- [[concepts/related-concept]] — Brief connection note

## Sources
- [[daily/2026-05-01.md]] — Initial discovery
```

### 2. Connection Articles (`knowledge/connections/`)

Synthesize relationships between concepts.

> **Frontmatter**

```yaml
---
title: "Connection: X and Y"
connects:
  - "concepts/concept-x"
  - "concepts/concept-y"
sources:
  - "daily/2026-05-04.md"
created: 2026-05-04
updated: 2026-05-04
---
```

> **Body** — Focus on the *non-obvious* insight.

### 3. Q&A Articles (`knowledge/qa/`)

Permanent storage of high-value answers.

> **Frontmatter**

```yaml
---
title: "Q: Original question text"
question: "Exact question asked"
consulted:
  - "concepts/article-1"
  - "concepts/article-2"
filed: 2026-05-05
---
```

---

## Core Agent Operations

### Compile (`scripts/compile.py`)

1. Read today’s daily log
2. Read `knowledge/index.md` for current state
3. Identify new knowledge, updates, and connections
4. Create or update articles (concepts first, then connections)
5. Update `index.md` and append to `log.md`

**Rules**:

- Never duplicate information — merge into existing concepts when possible
- Always add the daily log as a source
- Create connections only when the relationship is valuable and non-obvious

### Query (`scripts/query.py`)

Index-guided retrieval (no vector search):

1. Read `index.md`
2. Select 3–10 most relevant articles
3. Read full content of selected articles
4. Synthesize answer with `[[wikilink]]` citations
5. (Optional `--file-back`) → create QA article

### Lint (`scripts/lint.py`)

Health checks:

- Broken wikilinks
- Orphan articles
- Orphan daily logs (not compiled)
- Stale articles (daily changed after last compile)
- Missing backlinks
- Sparse articles (< 200 words)
- Potential contradictions

---

## Conventions & Style

- **Wikilinks**: `[[concepts/my-concept]]` or `[[daily/2026-05-05.md]]` — **no** `.md` extension
- **File names**: `kebab-case.md`, lowercase, no spaces
- **Dates**:
  - `YYYY-MM-DD` for frontmatter
  - Full ISO 8601 with timezone for log entries
- **Tone**: Clear, factual, encyclopedia-style. Prefer third person for concepts.
- **Frontmatter**: Required on every compiled article
- **Sources**: Always link back to originating daily log(s)
- **Links**: Prefer `[[concepts/...]]` over raw URLs when referencing internal knowledge

---

## Best Practices for Claude / Agents

- When extracting knowledge, think **atomic** — one concept per article unless a clear connection justifies a new file.
- Always check `index.md` before creating something new.
- When in doubt, create a concept article first, then a connection later.
- Use `lint` regularly and address issues promptly.
- Daily logs are **source of truth** for provenance — never delete them.

---

This file is the single source of truth for how the entire knowledge system should behave. Keep it up to date as you evolve the system.

**Last updated**: 2026-05-05
