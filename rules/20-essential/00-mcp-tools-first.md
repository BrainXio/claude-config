# MCP Tools First

**Prefer MCP tool interfaces over raw file reads for well-known services.**

## Principle

When a service is available through an MCP server, use its tools rather than reading raw files. MCP tools provide type-safe, version-aware interfaces that are less brittle than file path assumptions.

## When This Applies

- Documentation servers → prefer `get_document` over `cat`
- Knowledge bases → prefer `query` over `grep`
- Coordination/state → prefer the service's MCP interface over raw file reads

## When Raw Reads Are OK

- Files that are the direct subject of the current task
- Services with no MCP server configured
- Debugging MCP server issues themselves

## Enforcement

- Prefer `Read` for task-relevant files, MCP tools for service queries
- Never write directly to files managed by an MCP server
