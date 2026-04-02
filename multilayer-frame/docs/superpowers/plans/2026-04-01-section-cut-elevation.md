# Section Cut & Elevation — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Section Cut and Elevation generation flows to the Odin Toolbar prototype — interactive drawing/selection → new frame generated on canvas.

**Architecture:** All changes go into the single `index.html` file. CSS at top (before `</style>`), HTML elements in the `<body>`, JS at bottom (before `</script>`). Each flow is a state machine: idle → drawing/selecting → generating → done. Generated frames reuse the existing `.canvas-frame` styling.

**Tech Stack:** Vanilla HTML/CSS/JS, inline SVG, no dependencies.

**Key file:** `index.html` (~4270 lines). CSS ends at line ~1306, HTML body ~1308-1650, JS ~1837-4270.

---

### Task 1: CSS for Section Cut interactions

**Files:**
- Modify: `index.html` (CSS section, before `</style>`)

- [ ] **Step 1: Add section-cut mode CSS**

Insert before `</style>`:

```css
/* ─── Section Cut Mode ─── */
body.mode-section-cut { cursor: crosshair; }
body.mode-section-cut .toolbar-container,
body.mode-section-cut .canvas-frame,
body.mode-section-cut .floor-nav,
body.mode-section-cut .secondary-toolbar,
body.mode-section-cut .properties-panel { cursor: default; }

.section-cut-line {
  position: absolute;
  pointer-events: none;
  z-index: 56;
}
.section-cut-line line {
  stroke: #0099FF;
  stroke-width: 2;
  stroke-linecap: round;
}
.section-cut-line circle {
  fill: #0099FF;
  r: 4;
}
.section-cut-preview {
  stroke: #0099FF;
  stroke-width: 1.5;
  stroke-dasharray: 4 4;
  opacity: 0.5;
}

/* Side highlight when choosing direction */
.section-side-highlight {
  position: absolute;
  z-index: 56;
  pointer-events: auto;
  cursor: pointer;
  transition: background 0.15s, border-color 0.15s;
  border: 1.5px dashed transparent;
  border-radius: 4px;
  background: transparent;
}
.section-side-highlight:hover {
  background: rgba(0,153,255,0.06);
  border-color: rgba(0,153,255,0.2);
}

/* Persistent cut line on the frame after generation */
.section-cut-mark {
  position: absolute;
  z-index: 2;
  pointer-events: none;
}
.section-cut-mark line {
  stroke: rgba(0,153,255,0.35);
  stroke-width: 1.5;
  stroke-linecap: round;
}
.section-cut-mark text {
  fill: rgba(0,153,255,0.5);
  font-size: 9px;
  font-weight: 600;
  font-family: 'Inter', sans-serif;
}
```

- [ ] **Step 2: Verify no syntax errors**

Open `index.html` in browser, check DevTools console for CSS parse errors.

---

### Task 2: CSS for Elevation compass and generated frames

**Files:**
- Modify: `index.html` (CSS section, before `</style>`)

- [ ] **Step 1: Add compass and generated frame CSS**

Insert after the section cut CSS:

```css
/* ─── Elevation Compass ─── */
.elevation-compass {
  position: absolute;
  z-index: 57;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.2s;
}
.elevation-compass.visible {
  opacity: 1;
  pointer-events: auto;
}
.compass-ring {
  position: absolute;
  border: 1.5px solid rgba(255,255,255,0.1);
  border-radius: 50%;
}
.compass-marker {
  position: absolute;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: rgba(255,255,255,0.03);
  border: 1.5px solid rgba(255,255,255,0.08);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.15s, border-color 0.15s;
  font-size: 8px;
  font-weight: 700;
  color: rgba(255,255,255,0.25);
  font-family: 'Inter', sans-serif;
}
.compass-marker:hover {
  background: rgba(0,153,255,0.15);
  border-color: rgba(0,153,255,0.4);
  color: #0099FF;
}

/* ─── Generated Frames (Section/Elevation results) ─── */
.generated-frame {
  position: absolute;
  z-index: 55;
  border: 1px solid rgba(255,255,255,0.05);
  background: rgba(255,255,255,0.015);
  cursor: default;
  transition: border-color 0.15s;
}
.generated-frame:hover { border-color: rgba(0,153,255,0.3); }
.generated-frame.selected { border-color: #0099FF; }
.generated-frame.selected::before,
.generated-frame.selected::after {
  content: '';
  position: absolute;
  width: 6px;
  height: 6px;
  background: #fff;
  border: 1.5px solid #0099FF;
}
.generated-frame.selected::before { top: -4px; left: -4px; }
.generated-frame.selected::after  { top: -4px; right: -4px; }
.generated-frame-label {
  position: absolute;
  top: -20px;
  left: 0;
  font-size: 11px;
  font-weight: 500;
  color: rgba(255,255,255,0.4);
  font-family: 'Inter', sans-serif;
  white-space: nowrap;
}
.generated-frame:hover .generated-frame-label { color: rgba(0,153,255,0.6); }
.generated-frame.selected .generated-frame-label { color: #0099FF; }
```

- [ ] **Step 2: Verify in browser**

Open `index.html`, check DevTools console — no errors.

---

### Task 3: Section Cut state machine (JS)

**Files:**
- Modify: `index.html` (JS section, before closing `</script>`)

- [ ] **Step 1: Add section cut state machine**

Insert before the closing `</script>` tag:

```javascript
// ══════════════════════════════════════════════════════════════════
// ── Section Cut Flow ─────────────────────────────────────────────
// ══════════════════════════════════════════════════════════════════

let sectionMode = 'idle'; // 'idle' | 'drawing' | 'choosing'
let sectionStart = null;  // {x, y} relative to canvasFrame
let sectionEnd = null;
let sectionLineEl = null; // SVG overlay for the cut line
let sectionSideEls = [];  // highlight divs for the two sides
let sectionCount = 0;     // for naming A-A, B-B, ...
const generatedFrames = []; // track all generated frames for selection

function startSectionCut() {
  sectionMode = 'drawing';
  sectionStart = null;
  sectionEnd = null;
  document.body.classList.add('mode-section-cut');

  // Create SVG overlay on the canvas frame for drawing
  sectionLineEl = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
  sectionLineEl.classList.add('section-cut-line');
  sectionLineEl.style.cssText = 'position:absolute;inset:0;width:100%;height:100%;';
  canvasFrame.appendChild(sectionLineEl);
}

function cancelSectionCut() {
  sectionMode = 'idle';
  document.body.classList.remove('mode-section-cut');
  if (sectionLineEl) { sectionLineEl.remove(); sectionLineEl = null; }
  sectionSideEls.forEach(el => el.remove());
  sectionSideEls = [];
  sectionStart = null;
  sectionEnd = null;
}
```

- [ ] **Step 2: Add click handler on canvasFrame for placing points**

```javascript
canvasFrame.addEventListener('click', function sectionClickHandler(e) {
  if (sectionMode === 'idle') return; // handled by existing frame selection
  if (sectionMode !== 'drawing') return;
  e.stopPropagation();

  const rect = canvasFrame.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;

  if (!sectionStart) {
    // First click — start point
    sectionStart = { x, y };
    // Draw start dot
    const dot = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    dot.setAttribute('cx', x);
    dot.setAttribute('cy', y);
    dot.setAttribute('r', '4');
    dot.setAttribute('fill', '#0099FF');
    sectionLineEl.appendChild(dot);
  } else {
    // Second click — end point, draw line + show side choice
    sectionEnd = { x, y };

    // Draw final line + end dot
    sectionLineEl.innerHTML = '';
    const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
    line.setAttribute('x1', sectionStart.x);
    line.setAttribute('y1', sectionStart.y);
    line.setAttribute('x2', sectionEnd.x);
    line.setAttribute('y2', sectionEnd.y);
    sectionLineEl.appendChild(line);

    const d1 = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    d1.setAttribute('cx', sectionStart.x); d1.setAttribute('cy', sectionStart.y);
    sectionLineEl.appendChild(d1);
    const d2 = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    d2.setAttribute('cx', sectionEnd.x); d2.setAttribute('cy', sectionEnd.y);
    sectionLineEl.appendChild(d2);

    document.body.classList.remove('mode-section-cut');
    sectionMode = 'choosing';
    showSectionSides();
  }
});
```

- [ ] **Step 3: Add mousemove handler for preview line**

```javascript
canvasFrame.addEventListener('mousemove', function sectionMoveHandler(e) {
  if (sectionMode !== 'drawing' || !sectionStart) return;

  const rect = canvasFrame.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;

  // Remove previous preview
  const old = sectionLineEl.querySelector('.section-cut-preview');
  if (old) old.remove();

  const preview = document.createElementNS('http://www.w3.org/2000/svg', 'line');
  preview.classList.add('section-cut-preview');
  preview.setAttribute('x1', sectionStart.x);
  preview.setAttribute('y1', sectionStart.y);
  preview.setAttribute('x2', x);
  preview.setAttribute('y2', y);
  sectionLineEl.appendChild(preview);
});
```

- [ ] **Step 4: Verify — click Section button, check console for errors**

---

### Task 4: Section side selection (hover-to-select)

**Files:**
- Modify: `index.html` (JS section, continue after Task 3 code)

- [ ] **Step 1: Implement showSectionSides()**

```javascript
function showSectionSides() {
  // Determine the two half-planes defined by the cut line
  // Simplification: use a bounding box split
  const frameW = canvasFrame.offsetWidth;
  const frameH = canvasFrame.offsetHeight;
  const midX = (sectionStart.x + sectionEnd.x) / 2;
  const midY = (sectionStart.y + sectionEnd.y) / 2;

  // Line direction vector
  const dx = sectionEnd.x - sectionStart.x;
  const dy = sectionEnd.y - sectionStart.y;

  // Normal vector (perpendicular) — determines the two sides
  // Side A: normal points "left" of line direction
  // Side B: normal points "right"

  // Create two clickable half-plane overlays
  // Simplified as two rectangles covering each half of the frame
  const isMoreHorizontal = Math.abs(dx) > Math.abs(dy);

  const sideA = document.createElement('div');
  sideA.className = 'section-side-highlight';
  const sideB = document.createElement('div');
  sideB.className = 'section-side-highlight';

  if (isMoreHorizontal) {
    // Line is mostly horizontal — split top/bottom
    const splitY = midY;
    sideA.style.cssText = `left:0;top:0;width:100%;height:${splitY}px;`;
    sideB.style.cssText = `left:0;top:${splitY}px;width:100%;height:${frameH - splitY}px;`;
  } else {
    // Line is mostly vertical — split left/right
    const splitX = midX;
    sideA.style.cssText = `top:0;left:0;width:${splitX}px;height:100%;`;
    sideB.style.cssText = `top:0;left:${splitX}px;width:${frameW - splitX}px;height:100%;`;
  }

  sideA.addEventListener('click', (e) => { e.stopPropagation(); generateSectionFrame('A'); });
  sideB.addEventListener('click', (e) => { e.stopPropagation(); generateSectionFrame('B'); });

  canvasFrame.appendChild(sideA);
  canvasFrame.appendChild(sideB);
  sectionSideEls = [sideA, sideB];
}
```

- [ ] **Step 2: Test — draw a line, verify two hover zones appear**

---

### Task 5: Generate Section frame

**Files:**
- Modify: `index.html` (JS section, continue after Task 4 code)

- [ ] **Step 1: Implement generateSectionFrame()**

```javascript
function generateSectionFrame(side) {
  sectionCount++;
  const letter = String.fromCharCode(64 + sectionCount); // A, B, C...
  const name = `Section ${letter}-${letter}`;

  // Clean up section UI
  sectionSideEls.forEach(el => el.remove());
  sectionSideEls = [];
  if (sectionLineEl) { sectionLineEl.remove(); sectionLineEl = null; }
  sectionMode = 'idle';

  // Leave a permanent cut mark on the frame
  const mark = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
  mark.classList.add('section-cut-mark');
  mark.style.cssText = 'position:absolute;inset:0;width:100%;height:100%;';
  const markLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
  markLine.setAttribute('x1', sectionStart.x);
  markLine.setAttribute('y1', sectionStart.y);
  markLine.setAttribute('x2', sectionEnd.x);
  markLine.setAttribute('y2', sectionEnd.y);
  mark.appendChild(markLine);
  // Label at start
  const markLabel = document.createElementNS('http://www.w3.org/2000/svg', 'text');
  markLabel.setAttribute('x', sectionStart.x - 14);
  markLabel.setAttribute('y', sectionStart.y + 4);
  markLabel.textContent = letter;
  mark.appendChild(markLabel);
  canvasFrame.appendChild(mark);

  // Create the generated frame
  createGeneratedFrame(name, 'section');
}
```

- [ ] **Step 2: Implement createGeneratedFrame() — shared by Section + Elevation**

```javascript
function createGeneratedFrame(name, type) {
  // Position: right of the multilayer frame, stacking vertically
  const frameRect = canvasFrame.getBoundingClientRect();
  const canvasRect = document.querySelector('.canvas').getBoundingClientRect();
  const baseX = canvasFrame.offsetLeft + canvasFrame.offsetWidth + 40;
  const baseY = canvasFrame.offsetTop;

  // Stack generated frames vertically
  const existingCount = generatedFrames.length;
  const yOffset = existingCount * 300; // 260px height + 40px gap

  const frame = document.createElement('div');
  frame.className = 'generated-frame';
  frame.style.cssText = `left:${baseX}px;top:${baseY + yOffset}px;width:220px;height:260px;`;

  const label = document.createElement('div');
  label.className = 'generated-frame-label';
  label.textContent = name;
  frame.appendChild(label);

  // Content SVG — different for section vs elevation
  const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
  svg.style.cssText = 'position:absolute;inset:12px;width:calc(100% - 24px);height:calc(100% - 24px);';

  const floorCount = FLOORS.length;
  const visibleFloors = FLOORS.slice().sort((a, b) => b.id - a.id).slice(0, 8); // Show max 8

  if (type === 'section') {
    // Section: floor slabs + side walls + labels
    const slabH = 200 / visibleFloors.length;
    visibleFloors.forEach((floor, i) => {
      const y = i * slabH;
      // Slab line
      const slab = document.createElementNS('http://www.w3.org/2000/svg', 'line');
      slab.setAttribute('x1', '0'); slab.setAttribute('y1', y);
      slab.setAttribute('x2', '196'); slab.setAttribute('y2', y);
      slab.setAttribute('stroke', 'rgba(255,255,255,0.2)');
      slab.setAttribute('stroke-width', '2');
      svg.appendChild(slab);
      // Label
      const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      txt.setAttribute('x', '4'); txt.setAttribute('y', y + slabH / 2 + 4);
      txt.setAttribute('fill', 'rgba(255,255,255,0.15)');
      txt.setAttribute('font-size', '9'); txt.setAttribute('font-family', 'Inter');
      txt.textContent = floor.label;
      svg.appendChild(txt);
      // Height
      const ht = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      ht.setAttribute('x', '160'); ht.setAttribute('y', y + slabH / 2 + 4);
      ht.setAttribute('fill', 'rgba(255,255,255,0.1)');
      ht.setAttribute('font-size', '8'); ht.setAttribute('font-family', 'Inter');
      ht.textContent = (FLOOR_HEIGHTS[floor.id] || '2.80') + 'm';
      svg.appendChild(ht);
    });
    // Bottom slab
    const bottom = document.createElementNS('http://www.w3.org/2000/svg', 'line');
    bottom.setAttribute('x1', '0'); bottom.setAttribute('y1', '200');
    bottom.setAttribute('x2', '196'); bottom.setAttribute('y2', '200');
    bottom.setAttribute('stroke', 'rgba(255,255,255,0.2)'); bottom.setAttribute('stroke-width', '2');
    svg.appendChild(bottom);
    // Side walls
    ['0', '196'].forEach(x => {
      const wall = document.createElementNS('http://www.w3.org/2000/svg', 'line');
      wall.setAttribute('x1', x); wall.setAttribute('y1', '0');
      wall.setAttribute('x2', x); wall.setAttribute('y2', '200');
      wall.setAttribute('stroke', 'rgba(255,255,255,0.15)'); wall.setAttribute('stroke-width', '2');
      svg.appendChild(wall);
    });
  } else {
    // Elevation: facade outline + floor lines + roof
    const facadeH = 180;
    const facadeW = 180;
    const oX = 8;
    const oY = 30;
    // Roof triangle
    const roof = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
    roof.setAttribute('points', `${oX},${oY} ${oX + facadeW / 2},${oY - 25} ${oX + facadeW},${oY}`);
    roof.setAttribute('stroke', 'rgba(255,255,255,0.15)'); roof.setAttribute('fill', 'rgba(255,255,255,0.02)');
    roof.setAttribute('stroke-width', '1.5');
    svg.appendChild(roof);
    // Facade rect
    const facade = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
    facade.setAttribute('x', oX); facade.setAttribute('y', oY);
    facade.setAttribute('width', facadeW); facade.setAttribute('height', facadeH);
    facade.setAttribute('stroke', 'rgba(255,255,255,0.2)'); facade.setAttribute('fill', 'none');
    facade.setAttribute('stroke-width', '2');
    svg.appendChild(facade);
    // Floor lines + labels
    const slabH = facadeH / Math.min(visibleFloors.length, 6);
    visibleFloors.slice(0, 6).forEach((floor, i) => {
      const y = oY + i * slabH;
      if (i > 0) {
        const fl = document.createElementNS('http://www.w3.org/2000/svg', 'line');
        fl.setAttribute('x1', oX); fl.setAttribute('y1', y);
        fl.setAttribute('x2', oX + facadeW); fl.setAttribute('y2', y);
        fl.setAttribute('stroke', 'rgba(255,255,255,0.1)'); fl.setAttribute('stroke-width', '1');
        svg.appendChild(fl);
      }
      const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      txt.setAttribute('x', oX + 4); txt.setAttribute('y', y + slabH / 2 + 3);
      txt.setAttribute('fill', 'rgba(255,255,255,0.12)');
      txt.setAttribute('font-size', '8'); txt.setAttribute('font-family', 'Inter');
      txt.textContent = floor.label;
      svg.appendChild(txt);
    });
    // Ground line
    const ground = document.createElementNS('http://www.w3.org/2000/svg', 'line');
    ground.setAttribute('x1', '0'); ground.setAttribute('y1', oY + facadeH);
    ground.setAttribute('x2', '196'); ground.setAttribute('y2', oY + facadeH);
    ground.setAttribute('stroke', 'rgba(255,255,255,0.25)'); ground.setAttribute('stroke-width', '2');
    svg.appendChild(ground);
  }

  frame.appendChild(svg);

  // Make selectable
  frame.addEventListener('click', (e) => {
    e.stopPropagation();
    deselectAll();
    frame.classList.add('selected');
    propertiesPanel.classList.add('visible');
    propTitle.textContent = name;
    propFloorSection.style.display = 'none';
    propOutputSection.style.display = '';
  });

  document.querySelector('.canvas').appendChild(frame);
  generatedFrames.push(frame);

  // Also make document click aware of generated frames
  return frame;
}
```

- [ ] **Step 3: Test — full Section Cut flow end-to-end**

1. Convert frame to Multilayer
2. Click "Section" button
3. Draw a line (two clicks)
4. Hover each side — one highlights
5. Click a side — new frame appears to the right
6. Cut mark stays on original frame
7. New frame is selectable

---

### Task 6: Wire Section Cut button + Escape handler

**Files:**
- Modify: `index.html` (JS section)

- [ ] **Step 1: Wire the button**

```javascript
document.getElementById('secSectionCut').addEventListener('click', (e) => {
  e.stopPropagation();
  if (sectionMode !== 'idle') { cancelSectionCut(); return; }
  startSectionCut();
});
```

- [ ] **Step 2: Add Escape handler for section cut**

Update the existing keydown listener — add before the existing Escape check:

```javascript
// Add to the existing keydown handler, at the top:
if (e.key === 'Escape' && sectionMode !== 'idle') {
  cancelSectionCut();
  return;
}
```

- [ ] **Step 3: Test cancellation**

1. Click Section → crosshair appears
2. Press Escape → crosshair gone, back to normal
3. Click Section → click one point → Escape → line + dot removed

---

### Task 7: Elevation compass overlay (HTML + JS)

**Files:**
- Modify: `index.html` (JS section)

- [ ] **Step 1: Implement elevation compass**

```javascript
// ══════════════════════════════════════════════════════════════════
// ── Elevation Flow ───────────────────────────────────────────────
// ══════════════════════════════════════════════════════════════════

let compassEl = null;
const elevationCounts = { North: 0, South: 0, East: 0, West: 0 };

function showCompass() {
  if (compassEl) compassEl.remove();

  const fw = canvasFrame.offsetWidth;
  const fh = canvasFrame.offsetHeight;
  const ringSize = Math.min(120, fw * 0.3, fh * 0.3);
  const cx = fw / 2;
  const cy = fh / 2;

  compassEl = document.createElement('div');
  compassEl.className = 'elevation-compass';
  compassEl.style.cssText = `left:${cx - ringSize / 2}px;top:${cy - ringSize / 2}px;width:${ringSize}px;height:${ringSize}px;`;

  // Ring
  const ring = document.createElement('div');
  ring.className = 'compass-ring';
  ring.style.cssText = `inset:0;position:absolute;`;
  compassEl.appendChild(ring);

  // Markers: N (top), S (bottom), E (right), W (left)
  const markers = [
    { dir: 'North',  label: 'N', x: '50%', y: '0%',   tx: '-50%', ty: '-50%' },
    { dir: 'South',  label: 'S', x: '50%', y: '100%',  tx: '-50%', ty: '-50%' },
    { dir: 'East',   label: 'E', x: '100%', y: '50%',  tx: '-50%', ty: '-50%' },
    { dir: 'West',   label: 'W', x: '0%',  y: '50%',  tx: '-50%', ty: '-50%' },
  ];

  markers.forEach(m => {
    const marker = document.createElement('div');
    marker.className = 'compass-marker';
    marker.style.cssText = `left:${m.x};top:${m.y};transform:translate(${m.tx},${m.ty});position:absolute;`;
    marker.textContent = m.label;
    marker.addEventListener('click', (e) => {
      e.stopPropagation();
      generateElevation(m.dir);
    });
    compassEl.appendChild(marker);
  });

  // Click on compass background = cancel
  compassEl.addEventListener('click', (e) => {
    if (e.target === compassEl || e.target === ring) {
      e.stopPropagation();
      hideCompass();
    }
  });

  canvasFrame.appendChild(compassEl);
  // Animate in
  requestAnimationFrame(() => compassEl.classList.add('visible'));
}

function hideCompass() {
  if (compassEl) {
    compassEl.classList.remove('visible');
    setTimeout(() => { if (compassEl) { compassEl.remove(); compassEl = null; } }, 200);
  }
}

function generateElevation(direction) {
  hideCompass();

  elevationCounts[direction]++;
  const suffix = elevationCounts[direction] > 1 ? ` ${elevationCounts[direction]}` : '';
  const name = `Elevation ${direction}${suffix}`;

  createGeneratedFrame(name, 'elevation');
}
```

- [ ] **Step 2: Wire the Elevation button**

```javascript
document.getElementById('secElevation').addEventListener('click', (e) => {
  e.stopPropagation();
  if (compassEl) { hideCompass(); return; } // toggle
  showCompass();
});
```

- [ ] **Step 3: Add Escape handler for compass**

Add to the existing keydown handler:

```javascript
if (e.key === 'Escape' && compassEl) {
  hideCompass();
  return;
}
```

- [ ] **Step 4: Test full Elevation flow**

1. Convert to Multilayer
2. Click "Elevation" → compass appears centered
3. Hover markers → they highlight blue
4. Click "N" → compass disappears, "Elevation North" frame appears right
5. Click "Elevation" again → click "E" → "Elevation East" appears below first
6. Escape while compass is open → cancels

---

### Task 8: Update document click handler for generated frames

**Files:**
- Modify: `index.html` (JS section — update existing document click handler)

- [ ] **Step 1: Add generated frames to click-outside detection**

In the existing `document.addEventListener('click', ...)` handler, add before `deselectAll()`:

```javascript
// Check if click is on a generated frame
for (const gf of generatedFrames) {
  if (gf.contains(e.target)) return;
}
```

- [ ] **Step 2: Test — clicking generated frames doesn't deselect, clicking canvas does**

---

### Task 9: End-to-end verification

**Files:**
- No changes — testing only

- [ ] **Step 1: Full Section Cut test**

1. Open `index.html`
2. Click frame → click "Multi-Layer"
3. Click "Section" → cursor becomes crosshair
4. Click two points on the floor plan to draw a line
5. Hover each side — blue highlight appears
6. Click a side → "Section A-A" frame appears to the right with floor slabs
7. Cut line mark stays on the original frame
8. Click "Section" again → draw another line → "Section B-B" appears below A-A
9. Escape during drawing → cancels cleanly

- [ ] **Step 2: Full Elevation test**

1. Click "Elevation" → compass overlay appears
2. Hover N marker → turns blue
3. Click N → "Elevation North" frame appears with facade
4. Click "Elevation" again → click E → "Elevation East" appears below
5. Click "Elevation" again → Escape → compass disappears
6. All generated frames are selectable and show properties panel

- [ ] **Step 3: Interaction between flows**

1. Generate a Section Cut
2. Generate an Elevation
3. Both frames exist independently
4. Explode the multilayer frame → generated frames stay visible
5. Collapse → generated frames still there
6. Select different frames → correct properties panel shows

- [ ] **Step 4: Commit**

```bash
git add "index.html"
git commit -m "feat: add Section Cut and Elevation generation flows (Phase 2)"
```
