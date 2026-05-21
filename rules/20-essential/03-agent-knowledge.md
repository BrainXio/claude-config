# KB Engine Rule

## weight: 4 category: essential description: All agents must use the native knowledge engine for knowledge management

**Use `claude-knowledge` CLI or import from `claude_knowledge` for all knowledge operations.**

## CLI Commands

| Command                               | Purpose                          |
| ------------------------------------- | -------------------------------- |
| `claude-knowledge ingest <dir>`       | Ingest markdown files into KB    |
| `claude-knowledge compile`            | Compile daily logs into articles |
| `claude-knowledge query "<question>"` | TF-IDF semantic search           |
| `claude-knowledge validate`           | Check KB health                  |

## Module Imports

```python
from claude_knowledge.ingest import ingest_dir
from claude_knowledge.compile import compile_logs
from claude_knowledge.query import query_kb
from claude_knowledge.validate import validate_kb
```

## Data Architecture

Knowledge is stored per-repo under `./.knowledge/` (or `CLAUDE_KNOWLEDGE_DIR`):

- `articles/` — Structured KB articles
- `daily/` — Raw session logs
- `ingest_state.json` — Ingestion change tracking
- `compile_state.json` — Compilation state
- `index_cache.json` — TF-IDF search index

## When to Use

- **Before ingesting** — check mode thresholds via `claude_quality.modes`
- **After writing daily logs** — compile to articles
- **When searching** — use `query_kb()` or `claude-knowledge query`
- **Periodically** — validate KB health
