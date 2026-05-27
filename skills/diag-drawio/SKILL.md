# DrawIO Diagram Skill

## Mission

Create and edit DrawIO (`.drawio`) diagrams for BrainXio documentation.

## When Dispatched

- Architecture diagrams for ADRs
- Flowcharts for process documentation
- Sequence diagrams for system interactions
- Updating existing diagrams in `docs/diagrams/`

## Diagram Workflow

### Step 1: Check Existing Diagrams

```bash
# List available diagrams
ls -la docs/diagrams/architecture/
ls -la docs/diagrams/flowcharts/
ls -la docs/diagrams/sequence/
```

### Step 2: Create or Edit Diagram

DrawIO files are XML-based. Use one of:

1. **DrawIO Desktop** — Open `.drawio` file, edit, save
1. **DrawIO Online** — https://app.diagrams.net/, save to local file
1. **CLI export** — Use `drawio-export` for PNG/SVG generation

### Step 3: Store in Correct Location

| Diagram Type | Location                      |
| ------------ | ----------------------------- |
| Architecture | `docs/diagrams/architecture/` |
| Flowcharts   | `docs/diagrams/flowcharts/`   |
| Sequence     | `docs/diagrams/sequence/`     |
| Other        | `docs/diagrams/misc/`         |

### Step 4: Export for Documentation

```bash
# Export to PNG for markdown embedding
drawio-export --format png docs/diagrams/architecture/system-overview.drawio
```

### Step 5: Reference in Documentation

```markdown
![System Overview](../diagrams/architecture/system-overview.png)
```

## Naming Convention

- Use kebab-case: `system-overview.drawio`, `auth-flow.drawio`
- Prefix with ADR number if applicable: `adr-001-architecture.drawio`

## Anti-patterns

- **Raster exports in git** — Commit `.drawio` source, export PNGs as needed
- **Unlabeled connectors** — All arrows should have clear labels
- **Inconsistent styling** — Use BrainXio color palette and shapes
- **No source control** — Always commit the `.drawio` XML file

## Related

- `skills/diag-mermaid/SKILL.md` — Mermaid text-based diagrams
- `docs/diagrams/README.md` — Diagrams repository structure
