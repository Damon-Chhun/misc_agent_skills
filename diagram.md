---
name: diagram
description: >
  Generates clean, screenshot-ready HTML diagrams in five modes: flow (horizontal
  boxes + arrows), sequence (participants + vertical lifelines + labeled message arrows),
  flowchart (vertical branching layout with decision diamonds and Yes/No edge labels),
  multi-phase workflow (phase band backgrounds with vertical flow), and architecture
  (layered tier layout for system/component diagrams showing services and their relationships).
  All modes share the same light aesthetic: white background, Inter/JetBrains Mono fonts,
  pastel-colored boxes with colored borders.
  Use when the user wants to create any diagram, flow, sequence, process map, decision tree,
  route diagram, message-passing visual, system architecture diagram, or component diagram
  to screenshot into a doc or presentation.
  Trigger phrases: "diagram", "flow diagram", "sequence diagram", "flowchart", "create a diagram",
  "generate a flow", "draw this flow", "make a diagram", "visualize this flow",
  "sequence of calls", "message flow between services", "decision tree", "branching diagram",
  "architecture diagram", "component diagram", "system diagram", "service diagram",
  "infra diagram", "draw the system", "show the components", "how these services relate".
version: 4.0.0
---

# diagram

Generates a clean, screenshot-ready HTML diagram. Three modes: **flow**, **sequence**, **flowchart**.

---

## Mode 1 — Flow

Horizontal boxes connected by arrows. Use for: route flows, pipeline steps, redirect chains.

### Color palette (semantic roles)

| Role | Background | Border | Text |
|------|-----------|--------|------|
| `entry` — start node | `#edfaf3` | `#52c78a` | `#1a7a4a` |
| `process` — middleware / callback | `#eef4fd` | `#6aaee8` | `#1a4f8a` |
| `destination` — end node | `#fdf9ec` | `#d4a82a` | `#7a5a10` |
| `warning` — error / fallback | `#fef2f2` | `#e87a6a` | `#8a2a1a` |
| `neutral` — generic step | `#f5f5f5` | `#aaaaaa` | `#444444` |

### HTML template

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>{{title}}</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { background: #fff; display: flex; align-items: center; justify-content: center; padding: 48px 56px; font-family: 'Inter', sans-serif; }
  .flow { display: flex; align-items: center; gap: 0; }
  .box { display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 14px 22px; border-radius: 8px; min-width: 0; }
  .box-label { font-family: 'JetBrains Mono', monospace; font-size: 13.5px; font-weight: 500; letter-spacing: -0.2px; white-space: nowrap; }
  .box-sub { font-family: 'Inter', sans-serif; font-size: 11px; font-weight: 400; margin-top: 4px; white-space: nowrap; opacity: 0.7; }
  .box-entry       { background: #edfaf3; border: 1.5px solid #52c78a; }
  .box-entry       .box-label, .box-entry .box-sub { color: #1a7a4a; }
  .box-process     { background: #eef4fd; border: 1.5px solid #6aaee8; }
  .box-process     .box-label, .box-process .box-sub { color: #1a4f8a; }
  .box-destination { background: #fdf9ec; border: 1.5px solid #d4a82a; }
  .box-destination .box-label, .box-destination .box-sub { color: #7a5a10; }
  .box-warning     { background: #fef2f2; border: 1.5px solid #e87a6a; }
  .box-warning     .box-label, .box-warning .box-sub { color: #8a2a1a; }
  .box-neutral     { background: #f5f5f5; border: 1.5px solid #aaaaaa; }
  .box-neutral     .box-label, .box-neutral .box-sub { color: #444444; }
  .arrow { display: flex; align-items: center; padding: 0 4px; }
</style>
</head>
<body>
<div class="flow">
  <div class="box box-entry">
    <span class="box-label">/staging</span>
    <span class="box-sub">?deferredRoute=example</span>
  </div>
  <div class="arrow">
    <svg width="52" height="20" viewBox="0 0 52 20" fill="none">
      <line x1="2" y1="10" x2="44" y2="10" stroke="#aab" stroke-width="1.5"/>
      <polygon points="44,5 52,10 44,15" fill="#aab"/>
    </svg>
  </div>
  <div class="box box-process">
    <span class="box-label">requestIdleCallback</span>
    <span class="box-sub">+ setTimeout fallback</span>
  </div>
  <div class="arrow">
    <svg width="52" height="20" viewBox="0 0 52 20" fill="none">
      <line x1="2" y1="10" x2="44" y2="10" stroke="#aab" stroke-width="1.5"/>
      <polygon points="44,5 52,10 44,15" fill="#aab"/>
    </svg>
  </div>
  <div class="box box-destination">
    <span class="box-label">/next-experience</span>
  </div>
</div>
</body>
</html>
```

---

## Mode 2 — Sequence

Participants across the top, vertical dashed lifelines, labeled horizontal arrows. Use for: service call flows, message-passing between components, IPC/pubsub sequences.

### What it produces

- Participant boxes top + bottom mirror — same pastel color roles; default `neutral`
- Dashed vertical lifelines
- Solid arrows for requests, dashed for responses/callbacks
- Self-loop arcs for internal state transitions
- Optional stick-figure actor for human participants
- Multi-line message labels (stack `<text>` elements at +14px intervals)

### Layout math

- `participantWidth = 160`, evenly spaced across `totalWidth`
- `colX[i] = i * (participantWidth + gap) + participantWidth / 2`
- `headerHeight = 58`, `rowHeight = 52` (add 14px per extra label line)
- `totalHeight = headerHeight + steps * rowHeight + headerHeight`

### Key SVG snippets

```svg
<!-- lifeline -->
<line x1="{cx}" y1="58" x2="{cx}" y2="{totalH-58}" stroke="#d0d4de" stroke-width="1.5" stroke-dasharray="6 4"/>

<!-- request arrow (solid) -->
<line x1="{fromX}" y1="{y}" x2="{toX-8}" y2="{y}" stroke="#7a8aaa" stroke-width="1.5"/>
<polygon points="{toX-8},{y-4} {toX},{y} {toX-8},{y+4}" fill="#7a8aaa"/>
<text x="{midX}" y="{y-5}" text-anchor="middle" font-family="Inter" font-size="10.5" fill="#444">{label}</text>

<!-- return arrow (dashed) -->
<line x1="{toX+8}" y1="{y}" x2="{fromX}" y2="{y}" stroke="#9aaabb" stroke-width="1.5" stroke-dasharray="5 3"/>
<polygon points="{toX+8},{y-4} {toX},{y} {toX+8},{y+4}" fill="#9aaabb"/>

<!-- self-loop -->
<path d="M {cx} {y} C {cx+40} {y} {cx+40} {y+24} {cx} {y+24}" fill="none" stroke="#7a8aaa" stroke-width="1.5"/>
<polygon points="{cx},{y+20} {cx},{y+28} {cx-6},{y+24}" fill="#7a8aaa"/>
```

### Participant color roles

- Human actors → stick figure, `neutral` box
- Entry services → `entry` (green)
- Core orchestrators → `process` (blue)
- Terminal / leaf services → `destination` (gold)
- Default → `neutral`

---

## Mode 3 — Flowchart

Vertical branching layout with decision diamonds and Yes/No edge labels. Use for: decision trees, conditional logic flows, process maps with branching paths.

### What it produces

- Rectangular nodes (rounded corners) for process steps — same pastel color roles
- Diamond nodes for decision points — pink/salmon fill
- Vertical downward flow with left/right branches at decision points
- Edge labels on branches (`Yes` / `No`, or custom)
- Built as SVG with absolute positioning

### Color roles for flowchart nodes

| Role | Use for | Background | Border | Text |
|------|---------|-----------|--------|------|
| `entry` | Start node | `#edfaf3` | `#52c78a` | `#1a7a4a` |
| `process` | Standard step | `#eef4fd` | `#6aaee8` | `#1a4f8a` |
| `destination` | Terminal/end | `#fdf9ec` | `#d4a82a` | `#7a5a10` |
| `warning` | Error path | `#fef2f2` | `#e87a6a` | `#8a2a1a` |
| `neutral` | Generic step | `#f5f5f5` | `#aaaaaa` | `#444444` |
| `decision` | Diamond node | `#fde8e8` | `#e87a8a` | `#8a1a2a` |
| `alt` | Alternate branch | `#f3effe` | `#9a7ae8` | `#3a1a8a` |
| `accent` | Highlight step | `#fff8e8` | `#e8c84a` | `#7a5a00` |

### Layout math

- `nodeW = 180`, `nodeH = 44` — rectangle nodes
- `diamondHW = 70`, `diamondHH = 50` — diamond half-width/height
- Main column center x; branch columns offset ±220px from center
- `rowH = 100` — vertical gap between node centers
- Total width ~660px for two branches; scale up for more

### Key SVG snippets

```svg
<!-- rectangle node -->
<rect x="{cx-90}" y="{cy-22}" width="180" height="44" rx="8"
      fill="{bg}" stroke="{border}" stroke-width="1.5"/>
<text x="{cx}" y="{cy+5}" text-anchor="middle"
      font-family="'JetBrains Mono', monospace" font-size="12" font-weight="500" fill="{color}">{label}</text>

<!-- diamond decision node -->
<polygon points="{cx},{cy-50} {cx+70},{cy} {cx},{cy+50} {cx-70},{cy}"
         fill="#fde8e8" stroke="#e87a8a" stroke-width="1.5"/>
<text x="{cx}" y="{cy-6}" text-anchor="middle" font-family="Inter" font-size="11" fill="#8a1a2a">{line1}</text>
<text x="{cx}" y="{cy+10}" text-anchor="middle" font-family="Inter" font-size="11" fill="#8a1a2a">{line2}</text>

<!-- vertical connector -->
<line x1="{cx}" y1="{fromY}" x2="{cx}" y2="{toY-8}" stroke="#aab" stroke-width="1.5"/>
<polygon points="{cx-4},{toY-8} {cx+4},{toY-8} {cx},{toY}" fill="#aab"/>

<!-- diagonal branch line from diamond corner -->
<line x1="{cornerX}" y1="{diamondCY}" x2="{childCX}" y2="{childTopY}" stroke="#aab" stroke-width="1.5"/>
<polygon points="..." fill="#aab"/>

<!-- Yes/No edge label -->
<text x="{labelX}" y="{labelY}" font-family="Inter" font-size="11" fill="#888">{Yes|No}</text>
```

### Full HTML template

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>{{title}}</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { background: #fff; padding: 48px 56px; font-family: 'Inter', sans-serif; }
  svg { display: block; overflow: visible; }
</style>
</head>
<body>
<svg width="{{totalWidth}}" height="{{totalHeight}}" viewBox="0 0 {{totalWidth}} {{totalHeight}}">
  <!-- nodes, diamonds, connectors, edge labels -->
</svg>
</body>
</html>
```

### Flowchart construction rules

1. Start top-center with the first node (`entry` role).
2. Decision diamond: branch **left** for one path, **right** for the other. Place `Yes`/`No` labels near the diamond corners.
3. Use `accent` (yellow) or `alt` (purple) for nodes that need to stand out on a branch.
4. Branches that rejoin merge back to the center column. Branches that don't rejoin terminate with a `destination` node.
5. Keep branch offset ±220px from center (~660px total width). Scale up proportionally for more than two branches.

---

## Mode 4 — Multi-phase workflow

Vertical top-to-bottom layout with **phase band backgrounds** grouping related nodes.
Use for: skill/agent workflows, pipelines with named stages, processes with human gates.

### What it produces

- Full-width `<rect>` phase bands drawn FIRST (behind everything) — alternating fills:
  - Odd phases: `fill="#f7f8fa"` (light grey-blue)
  - Even phases: `fill="#ffffff"` (white)
  - Human gate phases: `fill="#fdf6f7"` (light pink)
  - Async/parallel phases: `fill="#f6fdf9"` (light green)
- Bold phase label at `x=16, y=(band_top+16)`, Inter 11px, `fill="#94a3b8"`
- Node color roles (same palette as Mode 3, plus):

| Role | Use for | Background | Border | Text |
|------|---------|-----------|--------|------|
| `human` | Human checkpoint / gate | `#fdf0f0` | `#e87a8a` | `#8a1a2a` |
| `artifact` | File/doc output | `#fff8e8` | `#e8c84a` | `#7a5a00` |

- Entry/exit nodes: pill shape (`rx="22"`), `fill="#edfaf3"`, `stroke="#52c78a"`, JetBrains Mono 12.5px `fill="#1a7a4a"`
- Loop-back connectors: dashed (`stroke-dasharray="6 3"`), routed along the left or right rail
- All connectors: `stroke="#aab"`, `stroke-width="1.5"`, arrowhead polygon `fill="#aab"`

### Construction order

1. Draw all phase band `<rect>` elements first (they sit behind everything)
2. Draw connector lines and polylines
3. Draw node shapes (`<rect>`, `<polygon>`)
4. Draw text labels last (they sit on top)

### Layout math

- Canvas width: 900–1100px depending on branch complexity
- Main column center: `cx = width / 2`
- Node width: 200px, height: 56px (two-line) or 44px (single-line)
- Diamond half-width: 88px, half-height: 58px
- Row gap: 100–120px between node centers
- Phase band padding: 20px above first node, 20px below last node

---

## Mode 5 — Architecture

Layered tier layout for system/component diagrams. Use for: service dependency maps, frontend/backend/infra breakdowns, plugin relationships, API topology.

### What it produces

- Horizontal **tier rows** (e.g., "Client", "API Layer", "Services", "Infrastructure") with a light band background
- Component boxes within each tier, centered horizontally
- Directed or undirected labeled edges between any two components across or within tiers
- Edge routing: straight lines with optional bend via midpoint; labels float above the line midpoint

### Color roles

| Role | Use for | Background | Border | Text |
|------|---------|-----------|--------|------|
| `entry` | Client / browser / user-facing | `#edfaf3` | `#52c78a` | `#1a7a4a` |
| `process` | Core service / orchestrator | `#eef4fd` | `#6aaee8` | `#1a4f8a` |
| `destination` | Data store / DB / queue | `#fdf9ec` | `#d4a82a` | `#7a5a10` |
| `warning` | Error boundary / fallback | `#fef2f2` | `#e87a6a` | `#8a2a1a` |
| `neutral` | Generic component | `#f5f5f5` | `#aaaaaa` | `#444444` |
| `external` | Third-party / CDN / external API | `#fff4ec` | `#e8924a` | `#7a3a10` |
| `infra` | Infrastructure / platform layer | `#f0f2f8` | `#7a8aaa` | `#2a3a5a` |

### Layout math

- Canvas width: 960px (default); scale up for wide tiers
- `tierH = 110` — height of each tier band
- `tierY[i] = i * tierH` — top y of tier i
- `nodeCY[i] = tierY[i] + tierH / 2` — vertical center of nodes in tier i
- Node width: 160px, height: 44px, `rx=8`
- Nodes in a tier evenly distributed: `nodeX[j] = (width / (count+1)) * (j+1)`
- Each node gets an `id` (e.g., `n0`, `n1`) for edge routing

### Edge routing

- Straight line from source center to target center when in different tiers
- For same-tier edges: arc via `<path d="M x1,y1 Q mx,my x2,y2">` where `my = nodeCY - 50`
- Arrowhead: `<polygon>` at target end, pointing in the direction of the line
- Bidirectional: draw two offset parallel lines (±4px perpendicular offset) each with their own arrowhead
- Edge label: `<text>` at line midpoint, `dy="-6"`, Inter 10px, `fill="#666"`

### Key SVG snippets

```svg
<!-- tier band -->
<rect x="0" y="{tierY}" width="{totalWidth}" height="{tierH}"
      fill="{odd:#f7f8fa, even:#ffffff}"/>
<text x="12" y="{tierY+18}" font-family="Inter" font-size="11"
      font-weight="600" fill="#94a3b8">{TIER LABEL}</text>

<!-- component node -->
<rect x="{cx-80}" y="{cy-22}" width="160" height="44" rx="8"
      fill="{bg}" stroke="{border}" stroke-width="1.5"/>
<text x="{cx}" y="{cy-4}" text-anchor="middle"
      font-family="'JetBrains Mono', monospace" font-size="12" font-weight="500" fill="{color}">{label}</text>
<text x="{cx}" y="{cy+12}" text-anchor="middle"
      font-family="Inter" font-size="10" fill="{color}" opacity="0.7">{sublabel}</text>

<!-- directed edge (cross-tier) -->
<line x1="{x1}" y1="{y1}" x2="{x2}" y2="{y2-8}"
      stroke="#aab" stroke-width="1.5"/>
<polygon points="{x2-4},{y2-8} {x2+4},{y2-8} {x2},{y2}" fill="#aab"/>
<text x="{(x1+x2)/2}" y="{(y1+y2)/2}" text-anchor="middle"
      font-family="Inter" font-size="10" fill="#888" dy="-6">{edge label}</text>

<!-- same-tier arc edge -->
<path d="M {x1} {cy} Q {(x1+x2)/2} {cy-50} {x2} {cy}"
      fill="none" stroke="#aab" stroke-width="1.5"/>
```

### Construction order

1. Draw tier band `<rect>` elements and tier labels first
2. Draw edges (lines/arcs) — they sit behind nodes
3. Draw node `<rect>` shapes
4. Draw node text labels last

### Architecture construction rules

1. **Infer tiers from the components**: group by role (UI → API → services → data → infra). Typical tier names: `Client`, `Gateway / API`, `Services`, `Data`, `Infrastructure`.
2. Within each tier, lay nodes out evenly. Max ~5 nodes per tier before widening canvas.
3. Default edge direction is top-to-bottom (caller → callee). Reverse or bidirectional edges are valid.
4. Label edges only when the relationship isn't obvious from position (e.g., `REST`, `pubsub`, `SQL`, `gRPC`, `CDN`).
5. Use `external` role for anything outside the system boundary (third-party APIs, CDNs, auth providers).
6. Use `infra` role for platform/infrastructure components (load balancers, message buses, container orchestrators).

---

## Intake (all modes)

> **Implementation rule (all modes):** Always produce hand-crafted SVG. Never use Mermaid,
> D3, or any external diagramming library. The only permitted CDN call is Google Fonts
> (Inter + JetBrains Mono). Every node, connector, and arrowhead is written as explicit
> SVG elements — `<rect>`, `<polygon>`, `<line>`, `<polyline>`, `<text>`.

1. **Infer the mode** — don't ask unless genuinely ambiguous:
   - Services/actors sending messages to each other → `sequence`
   - Decision points / Yes-No branching / conditional paths → `flowchart`
   - Linear chain of steps, routes, or pipeline stages → `flow`
   - Components/services with dependency or relationship edges across tiers → `architecture`
   - Named stages with parallel/async phases or human gates → `multi-phase workflow`
2. Extract nodes, participants, connections, and labels from the user's prose. Don't ask them to reformat.
3. Ask for **output path** only. Default: current working directory, kebab-case title derived from the content (e.g. `agent-lifecycle-flowchart.html`).

## Saving the file

Write the complete HTML to the specified path. Report the absolute path after writing.
