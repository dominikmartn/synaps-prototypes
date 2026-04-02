# Phase 2 — Section Cut & Elevation Flows

## Context

Extending the Synaps Odin Toolbar prototype (`index.html`) with two generation workflows. Both produce new frames from a Multilayer Frame. The prototype is a single-file HTML/CSS/JS app — no build step, no framework.

## Decisions Made

| Question | Decision |
|---|---|
| Section Cut: direction selection | Hover-to-Select on the surface (Option C) |
| Section Cut: generated frame placement | Eigenständiger Frame rechts daneben, keine Verbindung (A+C hybrid) |
| Elevation: direction selection | Kompass-Overlay (Option C) |
| Generated frame naming | Auto-Name, Architektur-Standard, nur Text (Option A) |

---

## Section Cut Flow

### Interaction Steps

1. **Activate** — User clicks "Section" button in Secondary Toolbar
2. **Draw line** — Cursor changes to crosshair. User clicks start point, then end point on the floor plan. A blue line renders between the two points with dots at each end.
3. **Choose direction** — The two sides of the line become hover targets. Moving the mouse to one side highlights that area with a subtle blue tint (`rgba(0,153,255,0.06)`) and a dashed border. The other side stays neutral.
4. **Confirm** — User clicks on the highlighted side. This confirms the viewing direction.
5. **Generate** — A new frame appears to the right of the Multilayer Frame (with ~40px gap). The frame is a static snapshot showing the building section from the chosen direction.
6. **Done** — Section mode deactivates. The cut line stays visible on the original frame. The generated frame is fully independent.

### Naming Convention

- First section: "Section A-A"
- Second: "Section B-B"
- Alphabetical sequence (A-A, B-B, C-C, ...)

### Generated Frame Content

The section frame shows:
- Horizontal floor slabs (one per floor in the Multilayer Frame)
- Vertical side walls
- Floor labels on the left (GF, F1, F2, ...)
- Floor heights on the right (2.80m, 2.60m, ...)

### States

| State | Visual |
|---|---|
| Drawing (line not placed) | Crosshair cursor, blue preview line follows mouse |
| Line placed, choosing side | Blue line with dots, one side highlights on hover |
| Generating | Brief flash, new frame appears |
| Complete | Line stays on original, new frame is independent |

### Cancellation

- Escape at any point cancels and returns to normal mode
- Clicking outside the frame while drawing cancels

---

## Elevation Flow

### Interaction Steps

1. **Activate** — User clicks "Elevation" button in Secondary Toolbar
2. **Compass appears** — A compass overlay renders centered on the Multilayer Frame. Ring with 4 markers: N (top), S (bottom), E (right), W (left).
3. **Choose direction** — Hover over a compass marker highlights it (blue fill). The corresponding frame edge could subtly glow to reinforce the spatial relationship.
4. **Confirm** — User clicks a compass marker.
5. **Generate** — A new frame appears to the right of the Multilayer Frame. Shows the building facade from the chosen direction.
6. **Done** — Compass disappears. Generated frame is fully independent.

### Naming Convention

- "Elevation North", "Elevation South", "Elevation East", "Elevation West"
- If same direction is generated twice: "Elevation North 2"

### Generated Frame Content

The elevation frame shows:
- Building outline (simple rectangular facade)
- Floor slab lines (horizontal, matching floor count)
- Floor labels on the left
- Ground line at bottom

### Compass UI Spec

- Circle ring: `1.5px solid rgba(255,255,255,0.1)`, centered on frame
- Ring diameter: ~120px (scales with frame, but not larger than 160px)
- 4 markers: 24x24px circles on the ring at N/S/E/W positions
- Default marker: `rgba(255,255,255,0.03)` fill, `rgba(255,255,255,0.08)` border
- Hover marker: `rgba(0,153,255,0.15)` fill, `rgba(0,153,255,0.4)` border, blue text
- Letter labels: 8px, font-weight 700
- Escape or click outside compass = cancel, compass disappears

---

## Shared Patterns

### Generated Frames

- Appear to the right of the source frame, ~40px horizontal gap
- Multiple generated frames stack: first at right, second below it, etc.
- Fully independent — no visual connection to source
- Selectable, movable (in real app — prototype shows selection only)
- Same selection handles as any other frame (white squares, blue border)

### Prototype Scope

This is a prototype — generated frames show representative content, not computed geometry. The interaction flow is the deliverable, not the architectural accuracy.

### What We're NOT Building

- Actual geometry computation
- Drag-to-reposition generated frames
- Live updating when source frame changes
- Multiple cut lines in a single session
