______________________________________________________________________

## weight: 8 category: standards description: All agents must use canonical documentation in ~/.your-org/docs/

# YourOrg Docs Rule

**Use `~/.your-org/docs/` as the canonical source of truth.**

## MCP Server

The preferred access method is the `your-agent-docs` MCP server, configured in `.mcp.json`:

```json
{
  "mcpServers": {
    "your-agent-docs": {
      "command": "uv",
      "args": ["run", "your-agent-docs"]
    }
  }
}
```

Tools: `get_document` (retrieve by path), `list_documents` (browse all with titles and tags).

## Access

Preferred: `your-agent-docs` MCP server.

Fallback: read directly from `~/.your-org/docs/`. Organized by category:

Fallback: read directly from `~/.your-org/docs/`. Organized by category:

- `master-plans/` — strategic plans, architectural decisions
- `error-handling/` — error patterns, symptoms, root causes, fixes
- `industry-standards/` — external standards and specifications
- `best-practices/` — internal conventions and patterns
- `rules-and-regulations/` — compliance requirements

## When to Use

- **Before any task** — check for relevant standards or master plans
- **When encountering errors** — search error-handling/ for known patterns
- **Making architectural decisions** — consult master-plans/ for existing rationale
- **Unsure about conventions** — best-practices/ documents project conventions
