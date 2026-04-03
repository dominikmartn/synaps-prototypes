# Multi-Layer Frame v2 Refinements — Design Spec

Based on team review meeting 2026-04-03. Changes to the existing prototype at `/Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html`.

Working branch: `v2-refinements` in `synaps-prototypes` repo.

---

## 1. Floor Selector Bar

**Replaces:** The current free-floating floor navigation (lines/dashes left of frame).

**New component:** A vertical toolbar-styled bar positioned left of the Multilayer Frame with 8px gap (Option C).

### Visual
- **Styling:** Same as Main Toolbar — `var(--surface--subtle)` bg, `var(--border--default)` border, `var(--radius--container)` radius, `var(--shadow--toolbar)` shadow, `backdrop-filter: blur(12px)`
- **Width:** 36px
- **Height:** Matches frame height
- **Position:** Fixed overlay (like current floor nav), 8px gap left of frame

### Layout (top → bottom)
1. **+ Button** (top) — Ghost button (24×24), `var(--content--secondary)` color, hover → `var(--surface--controls--button-hover)`. Adds floor above active floor (existing logic).
2. **Floor Dots** (middle, flex-1) — Circles replace lines. Active dot: 8px, `var(--accent--canvas--crosshair)`. Inactive dots: 6px, `var(--border--default)`. Edge-fading: dots near viewport edges shrink to 4-5px and fade.
3. **Explode Button** (bottom) — Ghost button (24×24), same style as + button. One-way export (see §4).

### Interaction (unchanged from current)
- Windowed viewport (max 14 visible)
- Spring-physics scroll animation
- Counter "x / y" (only visible when >7 floors, clickable → input)
- Scroll-wheel navigation
- Hover/Active/Dimmed states (Active = frame selected/hovered, Dimmed = frame deselected)
- Labels appear on hover (floor name next to dot)

### New interaction
- **Click-Hold-Drag:** User clicks and holds on any dot, then drags up/down to scrub through floors rapidly. Floor changes on each dot crossed. Releases to stop.

---

## 2. Secondary Toolbar (reduced)

**Changes:** Remove Add Floor and Explode from Secondary Toolbar. They move to the Floor Selector Bar.

**Remaining buttons:**
1. **Section Cut** — Icon + Text label "Section"
2. **Elevation** — Icon + Text label "Elevation"

**Style:** Same as current — icon + text buttons (`p-6px 10px`, 11px font, `var(--content--primary)`), toolbar container styling. Icons remain the same (ToolbarDividerIcon, Elevation house icon).

---

## 3. Section Cut — Point-by-Point

**Replaces:** Current auto-extend behavior (2 clicks → line extends to frame edges).

### New flow
1. Click "Section" in Secondary Toolbar → mode activates (instruction bar + canvas dim)
2. **Click first point** on floor plan → dot appears
3. **Click second point** → line drawn between the two points exactly as placed (NO auto-extension)
4. **Direction indicator:** A directional arrow symbol appears on the line (like AutoCAD section marks) indicating which side is being viewed. User clicks the arrow to flip direction, or clicks to confirm.
5. Section frame generates on canvas

### Line behavior
- Line stays exactly where user drew it — endpoints are NOT extended to frame boundary
- Line can be at any angle
- Section mark (permanent) shows on floor plan with label (A-A, B-B, etc.)

### Side selection
- Replace current area-highlight polygon approach
- Use **directional arrow** perpendicular to the line, pointing to the viewing side
- Arrow appears automatically on one side after drawing
- Click arrow → confirms that direction
- Click the other side of the line → flips the arrow

---

## 4. Explode — One-Way Export

**Replaces:** Current toggle behavior (Explode/Collapse).

### New behavior
- Click Explode → **new independent frames** appear on canvas (one per floor)
- Original Multilayer Frame remains **unchanged and visible**
- No "Collapse" button — this is a one-time export action
- No codependency between exported frames and original
- Exported frames are regular frames (selectable, moveable, deletable)
- To re-demonstrate: refresh the page

### Button behavior
- Button in Floor Selector Bar (bottom)
- After click: frames appear to the right of the original frame
- Button stays as "Explode" (no toggle to "Collapse")

---

## 5. Elevation — North Point Step

**Replaces:** Current direct compass overlay.

### New flow
1. Click "Elevation" in Secondary Toolbar → mode activates
2. **Step 1 — Set North Point:** Instruction bar shows "Click to set the North direction". User clicks a point on/near the frame, then moves mouse to define the North angle. Second click confirms.
3. **Step 2 — Compass:** Compass overlay appears on the frame, rotated to match the defined North direction. N/S/E/W markers are positioned relative to the actual North.
4. Click a direction → Elevation frame generates

### North Point persistence
- Once set, the North direction is remembered for the frame
- Subsequent Elevation requests skip Step 1 and go directly to the rotated compass
- North direction can be reset via a small indicator on the frame

---

## 6. Unchanged

These stay exactly as they are:
- Floor Selector interaction logic (windowing, spring animation, counter, hover states)
- Properties Panel structure and content (SidebarPanel pattern)
- Infinite Canvas (pan, zoom, counter-zoom)
- Frame selection/hover states (2px border, handles on selected)
- PDF Frame and Page Navigation
- Main Toolbar
- Tooltips with warm-up behavior
- Design system tokens
- Light theme floor plan content
- Generated frame content (section cut / elevation visuals)
