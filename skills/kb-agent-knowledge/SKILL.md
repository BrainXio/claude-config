# KB Engine Skill

Interact with a {knowledge-mcp} MCP server to manage the knowledge base:
ingest, compile, search, validate, and scan.

## Actions

| Action | Argument | Description |
| ------------ | -------------------- | ---------------------------------------------------------------- |
| `mode` | `<mode>` | Switch operational mode (developer/research/review/ops/personal) |
| `get-mode` | — | Return current mode, thresholds, and available modes |
| `ingest` | `[source]` | Ingest markdown files into cleaned KB artifacts |
| `compile` | `[--all] [--file X]` | Compile daily logs into structured articles |
| `query` | `<question>` | Search KB via TF-IDF with optional version filtering |
| `validate` | — | Run all 6 structural validation checks |
| `status` | — | KB health report with sync and lint counts |
| `scan` | `[--dir DIR]` | Scan for prototype projects and produce shortlist |
| `shortlist` | `[--domain X]` | Load prototype ingestion shortlist with optional filtering |
| `rules` | — | Return structured KB rules and schema |
| `create` | `<type> <title>` | Create a KB article with template scaffolding |
| `stub` | `[--dry-run]` | Auto-stub all broken \[[wikilinks]\] in the KB |
| `template` | `<type>` | Return article type template with section descriptions |
| `inject` | `[target]` | Inject the KB Engine rule into target agent systems |
| `inject-all` | — | Inject the KB Engine rule into all known agent systems |

## Targets for `inject`

| Target | File path |
| ------------- | ------------------------------------------------------ |
| `agents` | `AGENTS.md` |
| `claude-code` | `.claude/rules/01-agent-knowledge.md` |
| `windsurf` | `.windsurf/rules/agent-knowledge.md` |
| `cursor` | `.cursor/rules/agent-knowledge.mdc` |
| `copilot` | `.github/instructions/agent-knowledge.instructions.md` |
| `continue` | `.continue/rules/agent-knowledge.md` |
| `roo` | `.roo/rules/agent-knowledge.md` |
| `kiro` | `.kiro/steering/agent-knowledge.md` |
| `cline` | `.clinerules/agent-knowledge.md` |

## Procedure

1. Determine the action from the argument
1. Invoke the corresponding MCP tool on the {knowledge-mcp} server:
   - `mode <name>` → `set_mode` tool
   - `get-mode` → `get_mode` tool
   - `ingest` → `ingest` tool
   - `compile` → `compile` tool
   - `query <question>` → `query` tool
   - `validate` → `validate` tool
   - `status` → `status` tool
   - `scan` → `scan_prototypes` tool
   - `shortlist` → `get_shortlist` tool
   - `rules` → `get_rules` tool
   - `create <type> <title>` → `create_article` tool
   - `stub [--dry-run]` → `stub_broken_links` tool
   - `template <type>` → `get_template` tool
   - `inject [target]` → `inject_rules` tool with optional target parameter
   - `inject-all` → `inject_rules` tool with target `"all"`
1. Present results with article counts, validation issues, and search results
