# Multi-Layer Frame v2 Refinements — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Refine the Multi-Layer Frame prototype based on team review feedback — new Floor Selector Bar with dots, reduced Secondary Toolbar, point-by-point Section Cut, one-way Explode, and North Point Elevation flow.

**Architecture:** Single-file HTML/CSS/JS prototype at `/Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html` (~6630 lines). All changes are CSS + JS edits within this file. No build step, no framework.

**Tech Stack:** Vanilla HTML/CSS/JS, inline SVG, CSS custom properties (design tokens from Synaps repo).

---

### Task 1: Floor Selector Bar — CSS + HTML Structure

**Files:**
- Modify: `/Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html`

Replace the current `.floor-nav` CSS (lines ~1109-1210) and HTML (the two `floorNavMain`/`floorNavTower` divs at lines ~2161-2166) with the new Floor Selector Bar.

- [ ] **Step 1: Replace floor-nav CSS with floor-selector-bar CSS**

Find the CSS block starting at `.floor-nav {` (line ~1109) through `.floor-nav-counter-input {` (line ~1213). Replace the entire block with new CSS:

```css
/* ─── Floor Selector Bar (toolbar-styled, left of frame) ─── */
.floor-selector-bar {
  position: fixed;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 6px;
  width: 36px;
  background: var(--surface--subtle);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid var(--border--default);
  border-radius: var(--radius--container);
  box-shadow: var(--shadow--toolbar);
  z-index: 85;
  opacity: 0;
  transition: opacity 0.25s;
  pointer-events: none;
}
.floor-selector-bar.visible { opacity: 1; pointer-events: auto; cursor: pointer; }
.floor-selector-bar.visible.active { cursor: default; }
/* Dimmed state */
.floor-selector-bar.visible:not(.active) .floor-dot-label { opacity: 0 !important; }
.floor-selector-bar.visible:not(.active) .floor-dot.active { background: var(--border--strong) !important; width: 6px !important; height: 6px !important; }
.floor-selector-bar.visible:not(.active) .floor-dot-item { pointer-events: none; }
.floor-selector-bar.visible:not(.active) .floor-counter { visibility: hidden; }
.floor-selector-bar.visible:not(.active) .fsb-btn { opacity: 0; pointer-events: none; }

/* + and Explode buttons */
.fsb-btn {
  width: 24px;
  height: 24px;
  border-radius: var(--radius--control);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--content--secondary);
  background: transparent;
  border: none;
  cursor: pointer;
  transition: background 0.12s, color 0.12s;
  flex-shrink: 0;
}
.fsb-btn:hover { background: var(--surface--controls--button-hover); color: var(--content--primary); }
.fsb-btn svg { width: 14px; height: 14px; }

/* Dots viewport */
.floor-dots-viewport {
  flex: 1;
  overflow: hidden;
  position: relative;
  min-height: 0;
  width: 100%;
}
.floor-dots-track {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

/* Individual dot item */
.floor-dot-item {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 6px;
  height: 14px;
  cursor: pointer;
  position: relative;
  flex-shrink: 0;
  flex-direction: row-reverse;
}
.floor-dot-item::after {
  content: '';
  position: absolute;
  inset: -2px -8px -2px -16px;
}

/* Dot */
.floor-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--border--default);
  flex-shrink: 0;
  transition: width 0.3s cubic-bezier(0.25,1,0.5,1), height 0.3s cubic-bezier(0.25,1,0.5,1), background 0.3s, opacity 0.3s;
}
.floor-dot-item:hover .floor-dot { width: 7px; height: 7px; background: var(--border--strong); }
.floor-dot-item.active .floor-dot { width: 8px; height: 8px; background: var(--accent--canvas--crosshair); }

/* Label (appears on hover) */
.floor-dot-label {
  font-size: 10px;
  font-weight: 500;
  color: rgba(255,255,255,0);
  transition: color 0.2s, transform 0.2s cubic-bezier(0.4,0,0.2,1);
  transform: translateX(4px);
  white-space: nowrap;
  user-select: none;
}
.floor-dot-item:hover .floor-dot-label { color: var(--content--secondary); transform: translateX(0); }
.floor-dot-item.active .floor-dot-label { color: var(--accent--canvas--crosshair); transform: translateX(0); }

/* Counter */
.floor-counter {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 4px;
  font-size: 10px;
  font-weight: 500;
  color: var(--content--secondary);
  font-family: 'Inter', sans-serif;
  cursor: pointer;
  user-select: none;
  white-space: nowrap;
  transition: color 0.12s;
  padding: 2px 4px;
  border-radius: var(--radius--control);
}
.floor-counter:hover { color: var(--content--primary); background: var(--surface--controls--button-hover); }
.floor-counter .counter-current { color: var(--content--primary); }
.floor-counter .counter-sep { margin: 0 2px; color: var(--content--muted); }
.floor-counter-input {
  width: 32px;
  height: 20px;
  padding: 0 4px;
  background: var(--surface--controls--input-default);
  border: 1px solid var(--border--focus);
  border-radius: var(--radius--control);
  color: var(--content--primary);
  font-size: 10px;
  font-weight: 500;
  font-family: 'Inter', sans-serif;
  text-align: center;
  outline: none;
}
```

- [ ] **Step 2: Replace floor-nav HTML with floor-selector-bar HTML**

Find the two floor nav divs (`id="floorNavMain"` and `id="floorNavTower"`, lines ~2161-2166). Replace with:

```html
<!-- Floor Selector Bars (fixed overlays, one per multilayer frame) -->
<div class="floor-selector-bar" id="floorNavMain">
  <div class="fsb-btn" id="fsbAddFloorMain" title="Add Floor">
    <svg viewBox="0 0 16 16" fill="none"><path d="M8 1.14V14.86M1.14 7.95H14.86" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/></svg>
  </div>
  <div class="floor-dots-viewport"><div class="floor-dots-track"></div></div>
  <div class="fsb-btn" id="fsbExplodeMain" title="Explode">
    <svg viewBox="0 0 16 16" fill="none"><path d="M15.43.57h-.77c-1.43 0-2.72.89-3.22 2.24l-.02.05M4 8.57c0-.91.36-1.78 1-2.42.65-.65 1.52-1 2.43-1m0 10.28c3.79 0 6.86-3.07 6.86-6.86S11.22 1.71 7.43 1.71 .57 4.78.57 8.57s3.07 6.86 6.86 6.86z" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/></svg>
  </div>
</div>
<div class="floor-selector-bar" id="floorNavTower">
  <div class="fsb-btn" id="fsbAddFloorTower" title="Add Floor">
    <svg viewBox="0 0 16 16" fill="none"><path d="M8 1.14V14.86M1.14 7.95H14.86" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/></svg>
  </div>
  <div class="floor-dots-viewport"><div class="floor-dots-track"></div></div>
  <div class="fsb-btn" id="fsbExplodeTower" title="Explode">
    <svg viewBox="0 0 16 16" fill="none"><path d="M15.43.57h-.77c-1.43 0-2.72.89-3.22 2.24l-.02.05M4 8.57c0-.91.36-1.78 1-2.42.65-.65 1.52-1 2.43-1m0 10.28c3.79 0 6.86-3.07 6.86-6.86S11.22 1.71 7.43 1.71 .57 4.78.57 8.57s3.07 6.86 6.86 6.86z" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/></svg>
  </div>
</div>
```

- [ ] **Step 3: Update JS references**

In the JS, update all references from old class/element names to new ones:
- `floorNav` → references `floorNavMain` element (keep variable name `floorNav`)
- `floorNavGhost` → references `floorNavTower` element
- `floorNavTrack` → `floorNav.querySelector('.floor-dots-track')`
- `floorNavViewport` → `floorNav.querySelector('.floor-dots-viewport')`
- All `.floor-nav` class checks → `.floor-selector-bar`
- All `floor-nav-item` → `floor-dot-item`
- All `floor-nav-label` → `floor-dot-label`
- All `floor-nav-line` → `floor-dot` (element becomes a circle div)

In `buildFloorDom()` (line ~5025): change line creation from `div.floor-nav-line` to `div.floor-dot`, and item from `div.floor-nav-item` to `div.floor-dot-item`, label from `span.floor-nav-label` to `span.floor-dot-label`.

In `applyFloorNav()`: change width/height manipulations from line dimensions to dot dimensions:
- Active: `width: 8px; height: 8px;` (instead of `width: 22px; height: 3px;`)
- Inactive edge-fade: `width/height: Math.max(3, Math.round(6 * scale))px` and circular

- [ ] **Step 4: Wire + and Explode buttons in the bar**

The `fsbAddFloorMain`/`fsbAddFloorTower` buttons should call the existing `secAddFloor` click handler logic (add floor above active). The `fsbExplodeMain`/`fsbExplodeTower` buttons should call the new one-way explode (Task 4).

- [ ] **Step 5: Update `updateOverlayPositions` for new bar**

The `_syncOverlayPositions` function positions the floor nav. Update it:
- Width is now 36px (fixed) instead of dynamic
- Gap is 8px from frame left edge
- Bar height should match frame height (read from `getBoundingClientRect`)
- `floorNav.style.height = r.height + 'px'`
- `floorNav.style.left = (r.left - 36 - 8) + 'px'`
- `floorNav.style.top = r.top + 'px'`

- [ ] **Step 6: Verify and commit**

Open prototype locally, verify:
- Bar appears left of Tower B frame with toolbar styling
- Dots replace lines, active dot is larger and blue
- + and Explode buttons visible at top/bottom
- Dimmed state works (deselect → buttons hidden, active dot gray)
- Hover activates bar

```bash
git add multilayer-frame/index.html
# Also copy to repo
cp /Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html multilayer-frame/index.html
git commit -m "feat: floor selector bar with dots replacing line nav"
```

---

### Task 2: Secondary Toolbar — Reduce to Section + Elevation

**Files:**
- Modify: `/Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html`

- [ ] **Step 1: Remove Add Floor and Explode from Secondary Toolbar HTML**

Find the Secondary Toolbar HTML (line ~2183). Remove the `secAddFloor` and `secExplode` buttons. Keep only `secSectionCut` and `secElevation`. Change them to icon + text format:

```html
<div class="secondary-toolbar" id="secondaryToolbar">
  <div class="sec-tb-btn" id="secSectionCut" title="Section Cut">
    <svg viewBox="0 0 16 16" fill="none"><path d="M14.86 5V2.29C14.86 1.65 14.35 1.14 13.71 1.14H2.29C1.65 1.14 1.14 1.65 1.14 2.29V5M14.86 11V13.71C14.86 14.35 14.35 14.86 13.71 14.86H2.29C1.65 14.86 1.14 14.35 1.14 13.71V11M.6 8H2.31M4.89 8H6.6M9.46 8H11.17M13.74 8H15.46" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/></svg>
    Section
  </div>
  <div class="sec-tb-btn" id="secElevation" title="Elevation">
    <svg viewBox="0 0 16 16" fill="none"><path d="M1.14 13.71h13.72M3.43 13.71V6.86l4.57-3.43 4.57 3.43v6.85" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/><path d="M6.29 13.71v-3.43h3.43v3.43" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/></svg>
    Elevation
  </div>
</div>
```

- [ ] **Step 2: Update sec-tb-btn CSS for icon + text**

Change the `.sec-tb-btn` CSS (line ~1396) back to icon + text layout:

```css
.sec-tb-btn {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 6px 10px;
  font-size: 11px;
  font-weight: 500;
  color: var(--content--primary);
  border-radius: var(--radius--control);
  cursor: pointer;
  transition: background 0.12s, color 0.12s;
  white-space: nowrap;
  user-select: none;
}
```

- [ ] **Step 3: Update JS — remove old secAddFloor/secExplode handlers from Secondary Toolbar**

The `secAddFloor` click handler (line ~5098) and the `secExplodeBtn` click handler (line ~5432) should be removed from the Secondary Toolbar context. The Add Floor logic moves to the FSB buttons (Task 1 Step 4). The Explode logic is rewritten in Task 4.

- [ ] **Step 4: Update `syncToolbars()`**

The function (line ~4750) checks `isExploded` to keep the toolbar visible. Since Explode is no longer a toggle, remove the `isExploded` check. The toolbar should show when a multilayer frame is selected or hovered:

```js
function syncToolbars() {
  if (focusedGroup === 'ml' || (selectedFrame === 'main' && isMultilayer) || selectedFrame === 'tower') {
    secondaryToolbar.classList.add('visible');
  } else {
    secondaryToolbar.classList.remove('visible');
  }
  // PDF toolbar unchanged
  ...
}
```

- [ ] **Step 5: Commit**

```bash
git commit -m "feat: reduce secondary toolbar to section + elevation with text labels"
```

---

### Task 3: Section Cut — Point-by-Point (no auto-extend)

**Files:**
- Modify: `/Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html`

- [ ] **Step 1: Remove `extendLineToFrame` function**

Delete the `extendLineToFrame` function (lines ~5642-5680). It's no longer used.

- [ ] **Step 2: Simplify `handleSectionClick` — no extension**

In `handleSectionClick` (line ~5682), after the user clicks the second point, draw the line exactly between `sectionStart` and `{x, y}` without calling `extendLineToFrame`:

```js
} else {
  sectionEnd = { x, y };
  sectionLineEl.innerHTML = '';
  const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
  line.setAttribute('x1', sectionStart.x); line.setAttribute('y1', sectionStart.y);
  line.setAttribute('x2', sectionEnd.x); line.setAttribute('y2', sectionEnd.y);
  sectionLineEl.appendChild(line);
  [sectionStart, sectionEnd].forEach(p => {
    const d = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    d.setAttribute('cx', p.x); d.setAttribute('cy', p.y); d.setAttribute('r', '4');
    d.setAttribute('fill', '#6ba5f5');
    sectionLineEl.appendChild(d);
  });
  document.body.classList.remove('mode-section-cut');
  sectionMode = 'choosing';
  updateModeText('Click the arrow to confirm direction, or click the other side to flip');
  showDirectionArrow();
}
```

- [ ] **Step 3: Simplify `handleSectionMousemove` — no extension**

The preview line should draw from `sectionStart` to current mouse position, without extending:

```js
function handleSectionMousemove(e) {
  if (sectionMode !== 'drawing' || !sectionStart) return;
  const rect = sectionTargetFrame.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  const old = sectionLineEl.querySelector('.section-cut-preview');
  if (old) old.remove();
  const preview = document.createElementNS('http://www.w3.org/2000/svg', 'line');
  preview.classList.add('section-cut-preview');
  preview.setAttribute('x1', sectionStart.x); preview.setAttribute('y1', sectionStart.y);
  preview.setAttribute('x2', x); preview.setAttribute('y2', y);
  sectionLineEl.appendChild(preview);
}
```

- [ ] **Step 4: Replace `showSectionSides` with `showDirectionArrow`**

Remove the `showSectionSides` function and its clip-path polygon approach. Replace with a directional arrow:

```js
let directionArrowEl = null;
let currentDirection = 'A'; // 'A' or 'B'

function showDirectionArrow() {
  const dx = sectionEnd.x - sectionStart.x;
  const dy = sectionEnd.y - sectionStart.y;
  const midX = (sectionStart.x + sectionEnd.x) / 2;
  const midY = (sectionStart.y + sectionEnd.y) / 2;
  const len = Math.sqrt(dx * dx + dy * dy);
  // Perpendicular direction (normalized)
  const px = -dy / len;
  const py = dx / len;
  const arrowDist = 20; // pixels from line center

  // Arrow triangle pointing to side A
  const ax = midX + px * arrowDist;
  const ay = midY + py * arrowDist;

  directionArrowEl = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
  directionArrowEl.style.cssText = 'position:absolute;inset:0;width:100%;height:100%;pointer-events:auto;cursor:pointer;z-index:57;';

  function renderArrow(side) {
    directionArrowEl.innerHTML = '';
    const sign = side === 'A' ? 1 : -1;
    const tipX = midX + px * arrowDist * sign;
    const tipY = midY + py * arrowDist * sign;
    // Triangle arrow
    const baseL = 8;
    const bx1 = tipX - dx / len * baseL - px * baseL * sign;
    const by1 = tipY - dy / len * baseL - py * baseL * sign;
    const bx2 = tipX + dx / len * baseL - px * baseL * sign;
    const by2 = tipY + dy / len * baseL - py * baseL * sign;

    const arrow = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
    arrow.setAttribute('points', `${tipX},${tipY} ${bx1},${by1} ${bx2},${by2}`);
    arrow.setAttribute('fill', 'var(--accent--canvas--crosshair)');
    arrow.setAttribute('opacity', '0.8');
    directionArrowEl.appendChild(arrow);

    // Clickable area (larger circle)
    const hitArea = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    hitArea.setAttribute('cx', midX); hitArea.setAttribute('cy', midY);
    hitArea.setAttribute('r', '30');
    hitArea.setAttribute('fill', 'transparent');
    directionArrowEl.appendChild(hitArea);
  }

  renderArrow(currentDirection);

  directionArrowEl.addEventListener('click', (e) => {
    e.stopPropagation();
    const rect = sectionTargetFrame.getBoundingClientRect();
    const clickX = e.clientX - rect.left;
    const clickY = e.clientY - rect.top;
    // Determine which side of the line was clicked
    const cross = (clickX - sectionStart.x) * dy - (clickY - sectionStart.y) * dx;
    const clickedSide = cross > 0 ? 'A' : 'B';
    if (clickedSide !== currentDirection) {
      currentDirection = clickedSide;
      renderArrow(currentDirection);
    } else {
      // Confirm direction — generate section frame
      generateSectionFrame(currentDirection);
      directionArrowEl.remove();
      directionArrowEl = null;
    }
  });

  sectionTargetFrame.appendChild(directionArrowEl);
}
```

- [ ] **Step 5: Clean up old section-side-highlight CSS**

Remove the `.section-side-highlight` CSS rules (lines ~1715-1726) — no longer needed.

- [ ] **Step 6: Commit**

```bash
git commit -m "feat: section cut point-by-point with direction arrow"
```

---

### Task 4: Explode — One-Way Export

**Files:**
- Modify: `/Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html`

- [ ] **Step 1: Rewrite explode handler for one-way export**

Replace the current toggle-based explode handler (the `secExplodeBtn.addEventListener` block at line ~5432). The new handler in the FSB:

```js
function handleExplode() {
  const af = getActiveFrame();
  const posTop = parseInt(af.style.top) || af.offsetTop;
  const posLeft = parseInt(af.style.left) || af.offsetLeft;
  const items = FLOORS.slice().sort((a,b) => a.id - b.id).map(f => ({ label: f.name, floorId: f.id }));
  const name = getActiveFrameName();

  // Create independent frames to the right of the original (original stays visible)
  const startX = posLeft + af.offsetWidth + 40;
  items.forEach((item, i) => {
    const frame = document.createElement('div');
    frame.className = 'generated-frame';
    frame.style.cssText = `left:${startX + i * 230}px;top:${posTop}px;width:200px;height:${af.offsetHeight}px;`;

    const label = document.createElement('div');
    label.className = 'generated-frame-label';
    label.textContent = item.label;
    frame.appendChild(label);

    // Light theme floor plan content
    const content = document.createElement('div');
    content.style.cssText = 'position:absolute;inset:8px;';
    content.innerHTML = `<svg viewBox="0 0 184 ${af.offsetHeight - 16}" fill="none" style="width:100%;height:100%;">
      <rect x="2" y="2" width="180" height="${af.offsetHeight - 20}" stroke="#2a2a2a" stroke-width="2" rx="1"/>
      <line x1="60" y1="2" x2="60" y2="${af.offsetHeight - 18}" stroke="#999" stroke-width="1"/>
      <line x1="120" y1="2" x2="120" y2="${af.offsetHeight - 18}" stroke="#bbb" stroke-width=".5"/>
      <line x1="2" y1="${(af.offsetHeight-16)/2}" x2="182" y2="${(af.offsetHeight-16)/2}" stroke="#999" stroke-width="1"/>
      <rect x="10" y="15" width="35" height="50" rx="2" stroke="#ccc" stroke-width=".5" fill="none"/>
      <rect x="10" y="${(af.offsetHeight-16)/2 + 10}" width="50" height="8" rx="1" stroke="#ccc" stroke-width=".5" fill="#e8e8e6"/>
      <text x="92" y="${af.offsetHeight - 26}" text-anchor="middle" fill="#bbb" font-size="9" font-family="Inter">${item.label}</text>
    </svg>`;
    frame.appendChild(content);

    // Click handler — select this frame, show floor properties
    frame.addEventListener('click', (e) => {
      e.stopPropagation();
      deselectAll();
      frame.classList.add('selected');
      propertiesPanel.classList.add('visible');
      const floorObj = FLOORS.find(f => f.id === item.floorId);
      panelContent.innerHTML = renderFrameSection(name, { showGhost: false, showFormat: false })
        + renderFloorSection(floorObj)
        + renderExportSection();
      wireFloorActions();
      wireExportActions();
    });

    document.querySelector('.canvas').appendChild(frame);
    generatedFrames.push(frame);
  });
}
```

- [ ] **Step 2: Remove `collapseIfExploded` function**

Remove the function and all calls to it (`collapseIfExploded()` in `selectMainFrame`, `selectTowerFrame`, `selectPdfFrame`). Since explode is no longer a toggle, there's nothing to collapse.

- [ ] **Step 3: Remove `isExploded`, `explodedSourceFrame`, `explodedSourceKey` variables**

These toggle-state variables are no longer needed. Remove their declarations and all references.

- [ ] **Step 4: Remove `COLLAPSE_ICON` constant**

No longer used. Keep `EXPLODE_ICON` for reference in the FSB button.

- [ ] **Step 5: Clean up `mlExplodedGroup`, `createExplodedGroup`, `removeExplodedGroup`**

The old exploded group system (dotted border group with sub-frame selection) is replaced by independent generated frames. Remove:
- `createExplodedGroup` function
- `removeExplodedGroup` function
- `mlExplodedGroup` variable
- `selectExplodedGroup` function
- `focusedGroup` variable and all references
- `.frame-group` CSS and related styles

- [ ] **Step 6: Commit**

```bash
git commit -m "feat: one-way explode export (no collapse, independent frames)"
```

---

### Task 5: Elevation — North Point Step

**Files:**
- Modify: `/Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html`

- [ ] **Step 1: Add North Point state variables**

Near the elevation code (search for `function showCompass`):

```js
let northAngle = null; // radians, null = not set yet
let northPointFrame = null; // which frame has a north point set
let northPointStart = null; // first click position during north point setting
```

- [ ] **Step 2: Modify elevation click handler**

The `secElevation` click handler (line ~5757) currently calls `showCompass()` directly. Change it to check if North is set:

```js
document.getElementById('secElevation').addEventListener('click', (e) => {
  e.stopPropagation();
  if (compassEl) { hideCompass(); return; }
  sectionTargetFrame = getActiveFrame();
  if (northAngle !== null && northPointFrame === sectionTargetFrame) {
    // North already set — show rotated compass
    showCompass();
  } else {
    // Need to set North first
    startNorthPointFlow();
  }
});
```

- [ ] **Step 3: Implement `startNorthPointFlow`**

```js
function startNorthPointFlow() {
  northPointStart = null;
  showModeUI('Elevation', 'Click to place the North reference point');
  document.body.classList.add('mode-section-cut'); // reuse crosshair cursor

  function handleNorthClick(e) {
    const rect = sectionTargetFrame.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;

    if (!northPointStart) {
      northPointStart = { x, y };
      updateModeText('Move mouse to set North direction, then click to confirm');

      function handleNorthMove(e2) {
        // Show preview line from start to cursor
        // (reuse existing preview pattern if needed)
      }

      function handleNorthConfirm(e2) {
        const rect2 = sectionTargetFrame.getBoundingClientRect();
        const x2 = e2.clientX - rect2.left;
        const y2 = e2.clientY - rect2.top;
        northAngle = Math.atan2(-(y2 - northPointStart.y), x2 - northPointStart.x); // negative y because screen coords
        northPointFrame = sectionTargetFrame;
        document.body.classList.remove('mode-section-cut');
        sectionTargetFrame.removeEventListener('click', handleNorthConfirm, true);
        sectionTargetFrame.removeEventListener('mousemove', handleNorthMove);
        hideModeUI();
        // Now show rotated compass
        showCompass();
      }

      sectionTargetFrame.removeEventListener('click', handleNorthClick, true);
      sectionTargetFrame.addEventListener('mousemove', handleNorthMove);
      sectionTargetFrame.addEventListener('click', handleNorthConfirm, true);
    }
  }

  sectionTargetFrame.addEventListener('click', handleNorthClick, true);
}
```

- [ ] **Step 4: Modify `showCompass` to use `northAngle` for rotation**

In `showCompass()` (line ~5780), rotate the compass markers based on `northAngle`:

```js
function showCompass() {
  // ... existing setup code ...
  const rotation = northAngle || 0; // radians

  const markers = [
    { dir: 'North', label: 'N', angle: rotation },
    { dir: 'South', label: 'S', angle: rotation + Math.PI },
    { dir: 'East',  label: 'E', angle: rotation - Math.PI / 2 },
    { dir: 'West',  label: 'W', angle: rotation + Math.PI / 2 },
  ];

  markers.forEach(m => {
    const marker = document.createElement('div');
    marker.className = 'compass-marker';
    const dist = ringSize / 2;
    const mx = 50 + Math.cos(m.angle) * 50; // percentage
    const my = 50 - Math.sin(m.angle) * 50;
    marker.style.cssText = `left:${mx}%;top:${my}%;transform:translate(-50%,-50%);position:absolute;`;
    marker.textContent = m.label;
    // ... click handler ...
    compassEl.appendChild(marker);
  });
}
```

- [ ] **Step 5: Add North indicator on frame**

After North is set, show a small "N" indicator on the frame border so the user knows North is defined:

```js
function showNorthIndicator() {
  // Remove old indicator
  sectionTargetFrame.querySelector('.north-indicator')?.remove();
  const indicator = document.createElement('div');
  indicator.className = 'north-indicator';
  indicator.textContent = 'N';
  indicator.title = 'North direction set — click to reset';
  indicator.addEventListener('click', (e) => {
    e.stopPropagation();
    northAngle = null;
    northPointFrame = null;
    indicator.remove();
  });
  sectionTargetFrame.appendChild(indicator);
}
```

Add CSS for `.north-indicator`:
```css
.north-indicator {
  position: absolute;
  top: -20px;
  right: 30px;
  font-size: 9px;
  font-weight: 700;
  color: var(--accent--canvas--crosshair);
  background: var(--surface--subtle);
  border: 1px solid var(--border--default);
  border-radius: var(--radius--control);
  padding: 2px 5px;
  cursor: pointer;
  z-index: 2;
}
.north-indicator:hover { background: var(--surface--controls--button-hover); }
```

- [ ] **Step 6: Commit**

```bash
git commit -m "feat: elevation north point step before compass"
```

---

### Task 6: Click-Hold-Drag on Floor Selector

**Files:**
- Modify: `/Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html`

- [ ] **Step 1: Add drag state variables**

Near the floor nav code:

```js
let floorDragActive = false;
let floorDragStartY = 0;
```

- [ ] **Step 2: Add mousedown/mousemove/mouseup handlers on the dots viewport**

```js
const dotsViewport = floorNav.querySelector('.floor-dots-viewport');

dotsViewport.addEventListener('mousedown', (e) => {
  e.preventDefault();
  e.stopPropagation();
  floorDragActive = true;
  floorDragStartY = e.clientY;
  document.body.style.cursor = 'grabbing';
  document.body.style.userSelect = 'none';
});

document.addEventListener('mousemove', (e) => {
  if (!floorDragActive) return;
  // Find which dot is closest to the cursor
  const items = floorNav.querySelectorAll('.floor-dot-item');
  let closestId = activeFloor;
  let closestDist = Infinity;
  items.forEach(item => {
    const rect = item.getBoundingClientRect();
    const dist = Math.abs(e.clientY - (rect.top + rect.height / 2));
    if (dist < closestDist) {
      closestDist = dist;
      // Find the floor id for this item
      const floorEl = floorEls.find(el => el.item === item);
      if (floorEl) closestId = floorEl.id;
    }
  });
  if (closestId !== activeFloor) {
    setActiveFloor(closestId);
  }
});

document.addEventListener('mouseup', () => {
  if (floorDragActive) {
    floorDragActive = false;
    document.body.style.cursor = '';
    document.body.style.userSelect = '';
  }
});
```

- [ ] **Step 3: Commit**

```bash
git commit -m "feat: click-hold-drag to scrub through floors"
```

---

### Task 7: Final Integration + Deploy

**Files:**
- Modify: `/Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html`

- [ ] **Step 1: Verify all features work together**

Open prototype locally and test:
1. Floor Selector Bar appears with dots on Tower B
2. Convert Frame A → Bar appears with animation
3. + button adds floor, Explode exports frames
4. Secondary Toolbar shows only Section + Elevation (with text)
5. Section Cut: click two points, arrow appears, click to confirm direction
6. Elevation: first time asks for North Point, then shows rotated compass
7. Zoom/Pan: bar stays fixed size, hides at <50% zoom
8. Tooltips work on all new buttons

- [ ] **Step 2: Copy to repo, add auth, commit and push**

```bash
cd /Users/dominikmartin/Documents/claude/synaps/synaps-prototypes
cp /Users/dominikmartin/Documents/claude/synaps/multilayer-frame-prototype/index.html multilayer-frame/index.html
# Insert auth gate at <!-- AUTH:START --><!-- AUTH:END --> marker
git add multilayer-frame/
git commit -m "feat: v2 refinements — floor selector bar, point-by-point section cut, one-way explode, north point elevation"
git push origin v2-refinements
```

- [ ] **Step 3: Deploy to Vercel**

```bash
cd /Users/dominikmartin/Documents/claude/synaps/synaps-prototypes/multilayer-frame
vercel --prod --yes
```
