# Properties Panel Redesign — Design Spec

## Context

The Odin Toolbar prototype (`index.html`) has a basic Properties Panel that needs to be redesigned to match the Figma design system and support contextual content based on what's selected. This is a single-file HTML/CSS/JS prototype.

## Decisions Made

| Question | Decision |
|---|---|
| Staircase-height coupling indicator | ⓘ Info-Icon with tooltip on hover |
| Ghost Opacity scope | Frame-level setting (not per floor) |
| Export system | Figma-style stackable presets (C) with inline config (B) |
| Section Cut line caps | Start/End Cap dropdowns always visible (not behind settings icon) |
| Style section | Removed — not relevant for frames |

---

## Contextual Panel — What Shows When

The Properties Panel content changes entirely based on the current selection:

| Selection | Panel Content |
|---|---|
| Multilayer Frame (collapsed) | Header → Frame Section → Floor Section → Export |
| Multilayer Frame (exploded, group selected) | Header → Frame Section → Export |
| Exploded individual frame | Header → Frame Section → Floor Section |
| Section Cut Line | Header → Line Style Section (replaces everything) |
| PDF Frame | Header → Frame Section → Export |
| Nothing selected | Panel hidden |

---

## Panel Header

Persistent across all selection types.

- Avatar group (3 circles) + Upload + Comment + **Share** button (blue)
- Matches Figma design exactly

---

## Frame Section

Shown when any frame type is selected. Non-editiable frame name as section title.

### Fields

- **Frame Name** — displayed as section title (e.g., "Residence Block A"), not editable. Action buttons to the right: Lock, Group, Copy.
- **Format** — Dropdown with page icon. Options: A0, A1, A2, A3, A4, Custom.
- **Size (cm)** — Two inputs: Width ⇄ Height (swap button between them). Values in centimeters.
- **Scale** — Compound input: number field + `:1` suffix (e.g., `100 :1`).

### Layout

```
Residence Block A          🔒 ⬚ ⊞
Format
[□ A3                        ▾]
Size (cm)                Scale
[1536] ⇄ [1024]         [100 :1]
```

---

## Floor Section

Shown below Frame Section when a Multilayer Frame is selected (collapsed or exploded individual frame). Content changes based on the active floor.

### Section Title

"Floor — **Ground Floor**" where the floor name is highlighted in blue (#0099FF). Changes when user navigates floors via the Floor Navigation.

### Fields

- **Name** — Editable text input. Floor name (e.g., "Ground Floor", "Basement 1"). Editing this updates the Floor Navigation labels and the frame header.
- **Story Height** — Input with `m` suffix. Default: 2.80m. Has ⓘ info icon next to the label. Tooltip on hover: "Changing story height affects staircase geometry on this floor".
- **Ghost Opacity** — Input with `%` suffix. Default: 20. This is a frame-level setting (applies to all floors, not per-floor). Controls the opacity of the ghost layer showing the floor below.

### Floor Actions

Two buttons side by side at the bottom of the section:

- **Duplicate** — Neutral style, `⊕ Duplicate` label. Creates a copy of the current floor (inserted above).
- **Delete** — Danger style (red tint), `✕ Delete` label. Removes the current floor. Disabled if only 1 floor remains.

### Layout

```
Floor — Ground Floor
Name
[Ground Floor                ]
Story Height               ⓘ
[2.80                     ] m
Ghost Opacity
[20                       ] %
[⊕ Duplicate] [✕ Delete    ]
```

---

## Section Cut Line Panel

Replaces the entire panel content (not additive) when a Section Cut line is selected on the canvas. The line is a selectable object after a Section Cut has been generated.

### Fields

- **Stroke** — Color swatch (24×24px, shows current color) + Hex input + Opacity % input. Default: #0099FF, 100%.
- **Weight** — Input with `m` suffix + **Solid/Dash** toggle (segmented control). Default: 0.8m, Solid.
- **Start Cap** — Dropdown. Options: None (∅), Dot (●), Arrow (▶), Square (■), Line (—). Default: Dot.
- **End Cap** — Dropdown. Same options as Start Cap. Default: Arrow.

### Layout

```
Section A-A
Stroke
[■] [0099FF         ] [100 %]
[≡ 0.8] m    [Solid | Dash]
Start                    End
[● Dot            ▾] [▶ Arrow         ▾]
```

---

## Export Section

Shown at the bottom of the panel when a frame is selected. Uses stackable presets — user can configure multiple exports.

### Structure

- Section title "Export" with **+** button to add new presets
- Each preset is a card with:
  - **Row 1:** Scope dropdown + Format dropdown + ✕ remove button
  - **Row 2:** Contextual options (see below)
  - **Row 3:** Export button with summary text

### Scope Dropdown Options

- **Current Floor** — exports only the active floor
- **All Floors** — exports each floor separately

### Format Dropdown Options

- **PDF**
- **PNG**
- **JPG**

### Contextual Options

These appear in Row 2 based on the Scope + Format combination:

| Scope + Format | Contextual Option |
|---|---|
| All Floors + PDF | "Merge into single PDF" toggle (on/off) |
| Any + PNG | Size multiplier: 1x / 2x / 4x (segmented control) |
| Any + JPG | Size multiplier: 1x / 2x / 4x (segmented control) |
| Current Floor + PDF | No extra options |

### Export Button

Blue (#0099FF), full width within the card. Label summarizes the configuration:

- "Export 25 floors as merged PDF"
- "Export Ground Floor as PNG @2x"
- "Export all floors as separate PDFs"

### Default State

Panel starts with no presets. User clicks + to add the first one. Hint text below: "Click + to add an export".

---

## What's Removed

- **Style section** (Fill/Stroke) — not relevant for architectural frames
- **Output section** (old Paper/Scale) — replaced by Format/Scale in Frame Section + Export presets
- **W/H in px** — replaced by Size in cm

---

## Prototype Scope

This is a prototype — all controls are visual representations. Dropdowns show the selected value but don't open real menus. Inputs are editable for feel but don't compute anything. Export buttons don't actually export. The goal is to validate the layout and information architecture.
