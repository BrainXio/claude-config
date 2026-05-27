# DrawIO Diagram Skill

## Mission

Create well-structured DrawIO (`.drawio`) diagrams for BrainXio documentation.

## When Dispatched

- Architecture diagrams for ADRs
- Flowcharts for process documentation
- Sequence diagrams for system interactions
- Updating existing diagrams in `docs/diagrams/`

## MCP Server Integration

The workspace has the official draw.io MCP server configured at `https://mcp.draw.io/mcp`.

### Available Tools

| Tool             | Purpose                    |
| ---------------- | -------------------------- |
| `create_diagram` | Create new diagrams inline |
| `search_shapes`  | Find shape definitions     |

### Configuration

The MCP server is configured in `.mcp.json`:

```json
{
  "mcpServers": {
    "drawio": {
      "url": "https://mcp.draw.io/mcp"
    }
  }
}
```

**Note:** For generating `.drawio` XML files directly, follow the XML generation guidelines below. The MCP server is best for preview and iteration.

## XML Structure Reference

DrawIO uses XML format. **Always generate uncompressed XML** — never use compressed content.

### Required Structure

```xml
<mxfile host="app.diagrams.net" version="21.0.0">
  <diagram name="Diagram Name" id="unique-id">
    <mxGraphModel dx="1400" dy="700" grid="1" gridSize="10">
      <root>
        <!-- REQUIRED: Root container -->
        <mxCell id="0" />
        <!-- REQUIRED: Default layer -->
        <mxCell id="1" parent="0" />
        <!-- Your shapes here -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

**Mandatory elements:**

- `<mxCell id="0"/>` — Root container (must exist)
- `<mxCell id="1" parent="0"/>` — Default layer (must exist)
- All IDs must be unique

**Key elements:**

- `mxCell` with `vertex="1"` — Shapes/boxes
- `mxCell` with `edge="1"` — Connectors/arrows
- `mxGeometry` — Position (x, y) and size (width, height)
- `style` — Semicolon-separated key=value pairs

### Style Format

```xml
<mxCell value="Box Label" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#ffffff;" vertex="1" parent="1">
```

**Style string rules:**

- Use `key=value;` format (semicolon-separated)
- `vertex="1"` required for shapes
- `edge="1"` required for connectors
- Non-rectangular shapes need matching `perimeter=` value

### Coordinate System

- `(0,0)` is top-left corner
- Children of groups use **relative coordinates** (relative to parent container)
- Top-level elements use **absolute coordinates**

### HTML Escaping

HTML in `value` attributes must be XML-escaped:

```xml
<!-- Use &lt; &gt; &amp; not < > & -->
<mxCell value="A &amp; B&lt;C" ... />
```

## Layout Best Practices

### 1. Minimum Spacing Rules

| Element                      | Minimum Gap | Recommended |
| ---------------------------- | ----------- | ----------- |
| Between boxes in same group  | 20px        | 30px        |
| Between rows                 | 30px        | 50px        |
| Box to container edge        | 30px        | 50px        |
| Connection to non-target box | 15px        | 30px        |

### 2. Box Sizing Guidelines

| Text Lines                                   | Min Width | Min Height |
| -------------------------------------------- | --------- | ---------- |
| 1 line (short label)                         | 100px     | 50px       |
| 2 lines (label + description)                | 140px     | 70px       |
| 3 lines                                      | 160px     | 90px       |
| Long names (e.g., `agent-prefrontal-cortex`) | 160px     | 90px       |

**Rule:** Always add 20px padding to text width. Test by counting characters: ~15 chars fits 120px, ~20 chars needs 160px.

### 3. Container Sizing

Calculate container size:

```
container_width = max_content_width + 2 * edge_padding
container_height = max_content_height + 2 * edge_padding
```

**Edge padding:** Minimum 30px on all sides.

### 4. Connection Routing

**For straight connections:**

```xml
<mxCell id="a-to-b" value="" 
  style="endArrow=classic;html=1;entryX=0;entryY=0.5;" 
  source="a" target="b" edge="1">
```

**For connections crossing other elements (use orthogonal):**

```xml
<mxCell id="a-to-b" value="" 
  style="endArrow=classic;html=1;edgeStyle=orthogonalEdgeStyle;curved=1;" 
  source="a" target="b" edge="1">
```

**Entry/Exit points:** Use `entryX`, `entryY`, `exitX`, `exitY` with values 0, 0.5, or 1 for clean connections.

## Step-by-Step Workflow

### Step 1: Plan Layout on Paper

Before writing XML:

1. List all boxes needed
1. Group related boxes (containers)
1. Sketch approximate positions
1. Identify connections that might cross other boxes

### Step 2: Create Container Groups First

```xml
<!-- Define container first to establish boundaries -->
<mxCell id="group1" value="Group Name" 
  style="rounded=1;whiteSpace=wrap;html=1;fillColor=#d5e8d4;" 
  vertex="1" parent="1">
  <mxGeometry x="50" y="50" width="350" height="250" as="geometry" />
</mxCell>
```

### Step 3: Add Boxes with Consistent Spacing

```xml
<!-- Box 1: Start at x+30 from container edge -->
<mxCell id="box1" value="Label&#xa;Description" 
  style="rounded=0;whiteSpace=wrap;html=1;" 
  vertex="1" parent="1">
  <mxGeometry x="80" y="100" width="140" height="70" as="geometry" />
</mxCell>

<!-- Box 2: Add box_width + gap (140 + 30 = 170) -->
<mxCell id="box2" value="Label&#xa;Description" 
  style="rounded=0;whiteSpace=wrap;html=1;" 
  vertex="1" parent="1">
  <mxGeometry x="250" y="100" width="140" height="70" as="geometry" />
</mxCell>
```

### Step 4: Add Connections Last

```xml
<!-- Simple connection -->
<mxCell id="conn1" value="" 
  style="endArrow=classic;html=1;" 
  source="box1" target="box2" edge="1">
  <mxGeometry width="50" height="50" relative="1" as="geometry">
    <mxPoint x="220" y="135" as="sourcePoint" />
    <mxPoint x="250" y="135" as="targetPoint" />
  </mxGeometry>
</mxCell>
```

### Step 5: Validate Before Commit

Checklist:

- [ ] No text overflow (count chars vs width)
- [ ] Minimum 20px gaps between boxes
- [ ] Containers have 30px+ padding
- [ ] Connections don't clip through unrelated boxes
- [ ] Orthogonal routing for crossing connections
- [ ] Consistent box sizes within groups

## Color Palette

| Type                  | Fill Color | Stroke Color | Use Case                   |
| --------------------- | ---------- | ------------ | -------------------------- |
| Code Only             | `#ffffff`  | `#000000`    | Deterministic logic        |
| ML Model              | `#ffe6cc`  | `#d79b00`    | Prediction, classification |
| LLM Agent             | `#fff2cc`  | `#d6b656`    | Language reasoning         |
| VLM Agent             | `#ffe6cc`  | `#d79b00`    | Vision-language            |
| Container (default)   | `#d5e8d4`  | `#82b366`    | Grouping boxes             |
| Container (attention) | `#f8cecc`  | `#b85450`    | Critical groups            |

## Naming Convention

- Use kebab-case: `system-overview.drawio`, `auth-flow.drawio`
- Prefix with ADR number if applicable: `adr-001-architecture.drawio`
- Store in correct folder: `docs/diagrams/architecture/`, `docs/diagrams/flowcharts/`

## Common Mistakes & Fixes

| Mistake              | Symptom                        | Fix                                 |
| -------------------- | ------------------------------ | ----------------------------------- |
| Box too narrow       | Text clipped/overflow          | Increase width by 20-40px           |
| Boxes too close      | Visual crowding                | Add 10-20px gaps                    |
| Container too tight  | Boxes touch edge               | Add 30px padding                    |
| Arrow through box    | Connection clips unrelated box | Use `edgeStyle=orthogonalEdgeStyle` |
| Inconsistent sizes   | Amateur appearance             | Standardize box dimensions          |
| No entry/exit points | Arrows hit random edges        | Use `entryX/Y`, `exitX/Y`           |

## Export for Documentation

```bash
# Export to PNG for markdown embedding
drawio-export --format png docs/diagrams/architecture/system-overview.drawio
```

## Reference in Documentation

```markdown
![System Overview](../diagrams/architecture/system-overview.drawio)
```

## Anti-patterns

- **Raster exports in git** — Commit `.drawio` source, export PNGs as needed
- **Unlabeled connectors** — All arrows should have clear labels
- **Inconsistent styling** — Use BrainXio color palette and shapes
- **No source control** — Always commit the `.drawio` XML file
- **Skipping validation** — Always review spacing before committing

## Related

- `skills/diag-mermaid/SKILL.md` — Mermaid text-based diagrams
- `docs/diagrams/README.md` — Diagrams repository structure
- `docs/diagrams/architecture/agent-brain-regions.drawio` — Reference implementation
