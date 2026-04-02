# Properties Panel Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the current Properties Panel with a contextual panel that changes content based on selection type (Frame, Floor, Section Cut Line) and includes a stackable Export preset system.

**Architecture:** All changes in single `index.html`. Replace the existing panel HTML (lines ~1747-1831) with new markup. Update CSS to add new styles. Update all JS `selectMainFrame`/`selectPdfFrame`/`selectExplodedGroup` functions to populate the panel contextually. Section Cut lines become selectable objects with their own panel mode.

**Tech Stack:** Vanilla HTML/CSS/JS, inline SVG, no dependencies.

**Key file:** `index.html` (~4700 lines). CSS before `</style>`, HTML body ~1680-1850, JS ~2050-4700.

---

### Task 1: Replace Panel HTML with new structure

**Files:**
- Modify: `index.html` — replace the `<!-- Properties Panel -->` block (lines ~1747-1831)

- [ ] **Step 1: Read the current panel HTML**

Read lines 1747-1831 of `index.html` to find the exact `<div class="properties-panel"` opening and its closing `</div>`.

- [ ] **Step 2: Replace the entire panel HTML with new structure**

Replace the entire `<div class="properties-panel" id="propertiesPanel">...</div>` block with:

```html
<!-- Properties Panel (right side) -->
<div class="properties-panel" id="propertiesPanel">

  <!-- Header -->
  <div class="prop-header">
    <div style="display:flex;gap:0;">
      <div style="width:24px;height:24px;border-radius:50%;background:#444;border:2px solid rgba(21,23,25,0.85);"></div>
      <div style="width:24px;height:24px;border-radius:50%;background:#666;border:2px solid rgba(21,23,25,0.85);margin-left:-6px;"></div>
      <div style="width:24px;height:24px;border-radius:50%;background:#555;border:2px solid rgba(21,23,25,0.85);margin-left:-6px;"></div>
    </div>
    <div style="display:flex;align-items:center;gap:6px;">
      <div class="prop-close" style="color:rgba(255,255,255,0.3);font-size:13px;">↑</div>
      <div class="prop-close" style="color:rgba(255,255,255,0.3);font-size:13px;">💬</div>
      <div style="padding:5px 12px;background:#0099FF;border-radius:6px;font-size:11px;font-weight:600;color:#fff;cursor:pointer;">Share</div>
    </div>
  </div>

  <!-- Dynamic content container — populated by JS -->
  <div id="panelContent"></div>

</div>
```

- [ ] **Step 3: Verify the page loads without errors**

Open `index.html` in browser. Panel should be empty except for the header when a frame is selected.

---

### Task 2: Add new CSS for panel components

**Files:**
- Modify: `index.html` — CSS section, before `</style>`

- [ ] **Step 1: Add CSS for new panel elements**

Insert before `</style>`:

```css
/* ─── Properties Panel: Redesigned ─── */
.prop-section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}
.prop-section-name {
  font-size: 12px;
  font-weight: 600;
  color: #F8FAFC;
}
.prop-section-name .floor-highlight {
  color: #0099FF;
}
.prop-section-actions {
  display: flex;
  gap: 4px;
}
.prop-section-action-btn {
  width: 24px;
  height: 24px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: rgba(255,255,255,0.3);
  cursor: pointer;
  transition: background 0.12s, color 0.12s;
  font-size: 12px;
}
.prop-section-action-btn:hover {
  background: rgba(255,255,255,0.06);
  color: rgba(255,255,255,0.5);
}

/* Format dropdown (full width) */
.prop-format-dropdown {
  height: 28px;
  padding: 0 8px;
  background: rgba(255,255,255,0.05);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 11px;
  color: #F8FAFC;
  font-family: 'Inter', sans-serif;
  cursor: pointer;
}
.prop-format-icon {
  width: 14px;
  height: 18px;
  border: 1px solid rgba(255,255,255,0.2);
  border-radius: 1px;
  margin-right: 6px;
  flex-shrink: 0;
}
.prop-format-dropdown .chevron {
  font-size: 8px;
  color: rgba(255,255,255,0.2);
}

/* Size row with swap button */
.prop-size-row {
  display: flex;
  gap: 4px;
  align-items: flex-end;
}
.prop-size-row .prop-field { flex: 1; }
.prop-swap-btn {
  width: 20px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: rgba(255,255,255,0.15);
  cursor: pointer;
  font-size: 12px;
  flex-shrink: 0;
  transition: color 0.12s;
}
.prop-swap-btn:hover { color: rgba(255,255,255,0.4); }

/* Scale compound input */
.prop-scale-compound {
  display: flex;
  align-items: center;
}
.prop-scale-compound .prop-field-input {
  border-radius: 5px 0 0 5px;
  border-right: none;
}
.prop-scale-suffix {
  height: 28px;
  padding: 0 6px;
  background: rgba(255,255,255,0.03);
  border: 1px solid rgba(255,255,255,0.08);
  border-left: none;
  border-radius: 0 5px 5px 0;
  display: flex;
  align-items: center;
  font-size: 10px;
  color: rgba(255,255,255,0.3);
  font-family: 'Inter', sans-serif;
}

/* Info icon (for story height tooltip) */
.prop-info-icon {
  width: 13px;
  height: 13px;
  border-radius: 50%;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 8px;
  color: rgba(255,255,255,0.2);
  cursor: help;
  margin-left: 4px;
  position: relative;
}
.prop-info-icon:hover { background: rgba(255,255,255,0.08); color: rgba(255,255,255,0.4); }
.prop-info-tooltip {
  display: none;
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(21,23,25,0.95);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 6px;
  padding: 6px 10px;
  font-size: 10px;
  color: rgba(255,255,255,0.6);
  white-space: nowrap;
  z-index: 10;
  pointer-events: none;
}
.prop-info-icon:hover .prop-info-tooltip { display: block; }

/* Floor action buttons */
.prop-floor-actions {
  display: flex;
  gap: 6px;
  margin-top: 2px;
}
.prop-floor-action {
  flex: 1;
  height: 28px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 10px;
  font-family: 'Inter', sans-serif;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.12s;
}
.prop-floor-action.duplicate {
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.06);
  color: rgba(255,255,255,0.35);
}
.prop-floor-action.duplicate:hover { background: rgba(255,255,255,0.07); color: rgba(255,255,255,0.5); }
.prop-floor-action.delete {
  background: rgba(255,80,80,0.04);
  border: 1px solid rgba(255,80,80,0.08);
  color: rgba(255,80,80,0.4);
}
.prop-floor-action.delete:hover { background: rgba(255,80,80,0.08); color: rgba(255,80,80,0.6); }

/* Line style: color swatch */
.prop-color-swatch {
  width: 24px;
  height: 24px;
  border-radius: 4px;
  border: 1px solid rgba(255,255,255,0.1);
  flex-shrink: 0;
  cursor: pointer;
}

/* Solid/Dash toggle */
.prop-toggle-group {
  display: flex;
  gap: 0;
}
.prop-toggle-btn {
  flex: 1;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  font-family: 'Inter', sans-serif;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.12s, color 0.12s;
  color: rgba(255,255,255,0.3);
  background: rgba(255,255,255,0.03);
  border: 1px solid rgba(255,255,255,0.08);
}
.prop-toggle-btn:first-child { border-radius: 5px 0 0 5px; }
.prop-toggle-btn:last-child { border-radius: 0 5px 5px 0; border-left: none; }
.prop-toggle-btn.active {
  background: rgba(255,255,255,0.08);
  color: #F8FAFC;
  border-color: rgba(255,255,255,0.1);
}

/* Export presets */
.prop-export-card {
  padding: 10px;
  background: rgba(255,255,255,0.025);
  border: 1px solid rgba(255,255,255,0.06);
  border-radius: 7px;
  margin-bottom: 8px;
}
.prop-export-row {
  display: flex;
  gap: 6px;
  align-items: center;
  margin-bottom: 8px;
}
.prop-export-row:last-child { margin-bottom: 0; }
.prop-export-dropdown {
  flex: 1;
  height: 26px;
  padding: 0 7px;
  background: rgba(255,255,255,0.05);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 10px;
  color: #F8FAFC;
  font-family: 'Inter', sans-serif;
  cursor: pointer;
}
.prop-export-remove {
  width: 18px;
  height: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: rgba(255,255,255,0.12);
  cursor: pointer;
  font-size: 10px;
  flex-shrink: 0;
}
.prop-export-remove:hover { color: rgba(255,255,255,0.3); }
.prop-export-toggle {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}
.prop-export-toggle-label {
  font-size: 10px;
  color: rgba(255,255,255,0.35);
}
.prop-export-toggle-switch {
  width: 30px;
  height: 16px;
  border-radius: 8px;
  background: rgba(255,255,255,0.1);
  position: relative;
  cursor: pointer;
  transition: background 0.15s;
}
.prop-export-toggle-switch.on { background: #0099FF; }
.prop-export-toggle-switch::after {
  content: '';
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #fff;
  position: absolute;
  top: 2px;
  left: 2px;
  transition: left 0.15s;
}
.prop-export-toggle-switch.on::after { left: 16px; }
.prop-export-size-selector {
  display: flex;
  gap: 2px;
  background: rgba(255,255,255,0.03);
  border: 1px solid rgba(255,255,255,0.06);
  border-radius: 5px;
  padding: 2px;
  flex: 1;
}
.prop-export-size-option {
  flex: 1;
  height: 20px;
  border-radius: 3px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 9px;
  color: rgba(255,255,255,0.25);
  cursor: pointer;
  font-family: 'Inter', sans-serif;
  transition: background 0.12s, color 0.12s;
}
.prop-export-size-option.active {
  background: rgba(255,255,255,0.08);
  color: #F8FAFC;
  font-weight: 500;
}
.prop-export-btn {
  height: 28px;
  background: #0099FF;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  font-weight: 600;
  color: #fff;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
  transition: background 0.12s;
}
.prop-export-btn:hover { background: #1aa3ff; }
.prop-add-export {
  width: 22px;
  height: 22px;
  border-radius: 5px;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.06);
  display: flex;
  align-items: center;
  justify-content: center;
  color: rgba(255,255,255,0.3);
  font-size: 14px;
  cursor: pointer;
  transition: background 0.12s, color 0.12s;
}
.prop-add-export:hover { background: rgba(255,255,255,0.08); color: rgba(255,255,255,0.5); }
```

- [ ] **Step 2: Verify in browser — no CSS errors**

---

### Task 3: JS — Panel content renderer functions

**Files:**
- Modify: `index.html` — JS section, before `</script>`

- [ ] **Step 1: Add panel rendering functions**

Insert before `</script>`. These functions generate HTML and insert it into `#panelContent`:

```javascript
// ══════════════════════════════════════════════════════════════════
// ── Properties Panel Renderer ────────────────────────────────────
// ══════════════════════════════════════════════════════════════════

const panelContent = document.getElementById('panelContent');

function renderFrameSection(name) {
  return `
    <div class="prop-section">
      <div class="prop-section-header">
        <span class="prop-section-name">${name}</span>
        <div class="prop-section-actions">
          <div class="prop-section-action-btn" title="Lock">🔒</div>
          <div class="prop-section-action-btn" title="Group">⬚</div>
          <div class="prop-section-action-btn" title="Copy">⊞</div>
        </div>
      </div>
      <div class="prop-row" style="margin-bottom:8px;">
        <div class="prop-field">
          <div class="prop-field-label">Format</div>
          <div class="prop-format-dropdown">
            <div style="display:flex;align-items:center;">
              <div class="prop-format-icon"></div>
              A3
            </div>
            <span class="chevron">▾</span>
          </div>
        </div>
      </div>
      <div class="prop-row">
        <div class="prop-size-row">
          <div class="prop-field">
            <div class="prop-field-label">Size (cm)</div>
            <input class="prop-field-input" type="text" value="1536">
          </div>
          <div class="prop-swap-btn">⇄</div>
          <div class="prop-field">
            <div class="prop-field-label">&nbsp;</div>
            <input class="prop-field-input" type="text" value="1024">
          </div>
          <div class="prop-field" style="width:72px;flex:none;">
            <div class="prop-field-label">Scale</div>
            <div class="prop-scale-compound">
              <input class="prop-field-input" type="text" value="100" style="width:40px;">
              <div class="prop-scale-suffix">:1</div>
            </div>
          </div>
        </div>
      </div>
    </div>`;
}

function renderFloorSection(floor) {
  const floorName = floor ? floor.name : 'Ground Floor';
  const floorHeight = floor ? (FLOOR_HEIGHTS[floor.id] || '2.80') : '2.80';
  return `
    <div class="prop-section" id="propFloorSectionNew">
      <div class="prop-section-header">
        <span class="prop-section-name" style="font-size:11px;color:rgba(255,255,255,0.5);">Floor — <span class="floor-highlight">${floorName}</span></span>
      </div>
      <div class="prop-row" style="margin-bottom:8px;">
        <div class="prop-field">
          <div class="prop-field-label">Name</div>
          <input class="prop-field-input" type="text" value="${floorName}" id="propFloorName">
        </div>
      </div>
      <div class="prop-row" style="margin-bottom:8px;">
        <div class="prop-field prop-field-suffix" data-suffix="m">
          <div class="prop-field-label">Story Height <span class="prop-info-icon">i<div class="prop-info-tooltip">Changing story height affects staircase geometry on this floor</div></span></div>
          <input class="prop-field-input" type="text" value="${floorHeight}" id="propFloorHeight">
        </div>
      </div>
      <div class="prop-row" style="margin-bottom:8px;">
        <div class="prop-field prop-field-suffix" data-suffix="%">
          <div class="prop-field-label">Ghost Opacity</div>
          <input class="prop-field-input" type="text" value="20" id="propGhostOpacity">
        </div>
      </div>
      <div class="prop-floor-actions">
        <div class="prop-floor-action duplicate" id="propDuplicateFloor">⊕ Duplicate</div>
        <div class="prop-floor-action delete" id="propDeleteFloor">✕ Delete</div>
      </div>
    </div>`;
}

function renderLineSectionCut(name) {
  return `
    <div class="prop-section">
      <div class="prop-section-header">
        <span class="prop-section-name">${name}</span>
      </div>
      <div class="prop-row" style="margin-bottom:6px;">
        <div class="prop-field">
          <div class="prop-field-label">Stroke</div>
          <div style="display:flex;gap:6px;align-items:center;">
            <div class="prop-color-swatch" style="background:#0099FF;"></div>
            <input class="prop-field-input" type="text" value="0099FF" style="flex:1;">
            <div class="prop-field-suffix" data-suffix="%" style="width:56px;">
              <input class="prop-field-input" type="text" value="100" style="padding-right:24px;">
            </div>
          </div>
        </div>
      </div>
      <div class="prop-row" style="margin-bottom:8px;">
        <div class="prop-field prop-field-suffix" data-suffix="m" style="width:56px;flex:none;">
          <div class="prop-field-label">&nbsp;</div>
          <input class="prop-field-input" type="text" value="0.8">
        </div>
        <div class="prop-field" style="flex:1;">
          <div class="prop-field-label">&nbsp;</div>
          <div class="prop-toggle-group">
            <div class="prop-toggle-btn active">Solid</div>
            <div class="prop-toggle-btn">Dash</div>
          </div>
        </div>
      </div>
      <div class="prop-row">
        <div class="prop-field">
          <div class="prop-field-label">Start</div>
          <div class="prop-format-dropdown">
            <div style="display:flex;align-items:center;gap:5px;">
              <div style="width:6px;height:6px;border-radius:50%;background:#0099FF;"></div>
              Dot
            </div>
            <span class="chevron">▾</span>
          </div>
        </div>
        <div class="prop-field">
          <div class="prop-field-label">End</div>
          <div class="prop-format-dropdown">
            <div style="display:flex;align-items:center;gap:5px;">
              <div style="width:0;height:0;border-left:5px solid #0099FF;border-top:3px solid transparent;border-bottom:3px solid transparent;"></div>
              Arrow
            </div>
            <span class="chevron">▾</span>
          </div>
        </div>
      </div>
    </div>`;
}

function renderExportSection() {
  return `
    <div class="prop-section" id="propExportSection">
      <div class="prop-section-header">
        <span class="prop-section-name" style="font-size:11px;color:rgba(255,255,255,0.5);">Export</span>
        <div class="prop-add-export" id="propAddExport">+</div>
      </div>
      <div id="propExportPresets"></div>
      <div style="font-size:9px;color:rgba(255,255,255,0.1);text-align:center;padding:4px 0;" id="propExportHint">Click + to add an export</div>
    </div>`;
}

function addExportPreset() {
  const container = document.getElementById('propExportPresets');
  const hint = document.getElementById('propExportHint');
  if (hint) hint.style.display = 'none';

  const card = document.createElement('div');
  card.className = 'prop-export-card';
  card.innerHTML = `
    <div class="prop-export-row">
      <div class="prop-export-dropdown">All Floors <span class="chevron" style="font-size:7px;color:rgba(255,255,255,0.2);">▾</span></div>
      <div class="prop-export-dropdown">PDF <span class="chevron" style="font-size:7px;color:rgba(255,255,255,0.2);">▾</span></div>
      <div class="prop-export-remove">✕</div>
    </div>
    <div class="prop-export-toggle">
      <span class="prop-export-toggle-label">Merge into single PDF</span>
      <div class="prop-export-toggle-switch on"></div>
    </div>
    <div class="prop-export-btn">Export ${FLOORS.length} floors as merged PDF</div>
  `;

  // Toggle click
  const toggle = card.querySelector('.prop-export-toggle-switch');
  toggle.addEventListener('click', (e) => {
    e.stopPropagation();
    toggle.classList.toggle('on');
    const btn = card.querySelector('.prop-export-btn');
    btn.textContent = toggle.classList.contains('on')
      ? `Export ${FLOORS.length} floors as merged PDF`
      : `Export ${FLOORS.length} floors as separate PDFs`;
  });

  // Remove click
  card.querySelector('.prop-export-remove').addEventListener('click', (e) => {
    e.stopPropagation();
    card.remove();
    const remaining = container.querySelectorAll('.prop-export-card');
    if (remaining.length === 0 && hint) hint.style.display = '';
  });

  container.appendChild(card);
}
```

- [ ] **Step 2: Verify functions exist — no syntax errors in console**

---

### Task 4: JS — Wire panel rendering into selection functions

**Files:**
- Modify: `index.html` — update existing `selectMainFrame()`, `selectPdfFrame()`, `selectExplodedGroup()` functions

- [ ] **Step 1: Update selectMainFrame()**

Find the existing `selectMainFrame()` function and replace its body with:

```javascript
function selectMainFrame() {
  clearSelection();
  focusedGroup = null;
  selectedFrame = 'main';
  canvasFrame.classList.add('selected');
  propertiesPanel.classList.add('visible');
  syncToolbars();

  // Render panel content
  const activeFloorObj = FLOORS.find(f => f.id === activeFloor);
  if (isMultilayer) {
    panelContent.innerHTML = renderFrameSection('Residence Block A')
      + renderFloorSection(activeFloorObj)
      + renderExportSection();
    wireFloorActions();
    wireExportActions();
  } else {
    panelContent.innerHTML = renderFrameSection('Frame');
  }
}
```

- [ ] **Step 2: Update selectPdfFrame()**

Replace body:

```javascript
function selectPdfFrame() {
  clearSelection();
  focusedGroup = null;
  selectedFrame = 'pdf';
  pdfFrame.classList.add('selected');
  propertiesPanel.classList.add('visible');
  syncToolbars();

  panelContent.innerHTML = renderFrameSection('Section A-A')
    + renderExportSection();
  wireExportActions();
}
```

- [ ] **Step 3: Update selectExplodedGroup()**

Replace body:

```javascript
function selectExplodedGroup(type) {
  clearSelection();
  focusedGroup = type;
  const label = type === 'ml' ? 'Residence Block A — Exploded' : 'Section A-A — Exploded';
  propertiesPanel.classList.add('visible');
  syncToolbars();

  panelContent.innerHTML = renderFrameSection(label)
    + renderExportSection();
  wireExportActions();
}
```

- [ ] **Step 4: Update setActiveFloor() to re-render floor section**

Find `function setActiveFloor(floorId)` and add at the end (after the existing `applyFloorNav()` call):

```javascript
  // Re-render panel if frame is selected
  if (selectedFrame === 'main' && isMultilayer) {
    selectMainFrame(); // re-renders the whole panel with updated floor
  }
```

- [ ] **Step 5: Add wireFloorActions() and wireExportActions() helpers**

```javascript
function wireFloorActions() {
  const dupBtn = document.getElementById('propDuplicateFloor');
  const delBtn = document.getElementById('propDeleteFloor');
  if (dupBtn) {
    dupBtn.addEventListener('click', (e) => {
      e.stopPropagation();
      // Duplicate: copy current floor, insert above
      const current = FLOORS.find(f => f.id === activeFloor);
      if (!current) return;
      const maxId = Math.max(...FLOORS.map(f => f.id));
      const newFloor = { id: maxId + 1, label: `F${maxId + 1}`, name: current.name + ' Copy' };
      FLOORS.push(newFloor);
      buildFloorDom();
      setActiveFloor(newFloor.id);
    });
  }
  if (delBtn) {
    delBtn.addEventListener('click', (e) => {
      e.stopPropagation();
      if (FLOORS.length <= 1) return;
      const idx = FLOORS.findIndex(f => f.id === activeFloor);
      if (idx === -1) return;
      FLOORS.splice(idx, 1);
      buildFloorDom();
      const nearest = FLOORS[Math.min(idx, FLOORS.length - 1)];
      setActiveFloor(nearest.id);
    });
  }

  // Ghost opacity coupling
  const ghostInput = document.getElementById('propGhostOpacity');
  if (ghostInput) {
    ghostInput.addEventListener('input', () => {
      const val = parseFloat(ghostInput.value);
      if (isNaN(val)) return;
      if (activeFloor !== 0) {
        frameGhost.style.opacity = (Math.max(0, Math.min(100, val)) / 100).toString();
      }
    });
  }

  // Floor name editing
  const nameInput = document.getElementById('propFloorName');
  if (nameInput) {
    nameInput.addEventListener('change', () => {
      const floor = FLOORS.find(f => f.id === activeFloor);
      if (floor) {
        floor.name = nameInput.value;
        frameName.textContent = floor.name;
        buildFloorDom();
      }
    });
  }
}

function wireExportActions() {
  const addBtn = document.getElementById('propAddExport');
  if (addBtn) {
    addBtn.addEventListener('click', (e) => {
      e.stopPropagation();
      addExportPreset();
    });
  }
}
```

- [ ] **Step 6: Verify — select frame, panel shows new layout. Switch floors, panel updates.**

---

### Task 5: JS — Section Cut line selection with Line panel

**Files:**
- Modify: `index.html` — JS section

- [ ] **Step 1: Make section cut marks selectable**

In the `generateSectionFrame()` function, find where the `mark` SVG is created and appended. After `canvasFrame.appendChild(mark);`, add:

```javascript
// Make the cut mark selectable
mark.style.pointerEvents = 'auto';
mark.style.cursor = 'default';
mark.addEventListener('click', (e) => {
  e.stopPropagation();
  deselectAll();
  mark.style.opacity = '1';
  propertiesPanel.classList.add('visible');
  panelContent.innerHTML = renderLineSectionCut(name);
  // Store reference for deselection
  mark._selected = true;
});
```

- [ ] **Step 2: Update deselectAll to reset section cut mark styles**

In the `deselectAll()` function, add:

```javascript
// Deselect any section cut marks
canvasFrame.querySelectorAll('.section-cut-mark').forEach(m => {
  m.style.opacity = '';
  m._selected = false;
});
```

- [ ] **Step 3: Update document click handler to handle cut mark clicks**

In the existing `document.addEventListener('click', ...)`, add before `deselectAll()`:

```javascript
// Check if click is on a section cut mark
if (e.target.closest('.section-cut-mark')) return;
```

- [ ] **Step 4: Also update generated frame click handler to show panel**

In `createGeneratedFrame()`, update the frame click handler to render the panel:

Find the existing `frame.addEventListener('click', ...)` inside `createGeneratedFrame` and replace the body:

```javascript
frame.addEventListener('click', (e) => {
  e.stopPropagation();
  deselectAll();
  frame.classList.add('selected');
  propertiesPanel.classList.add('visible');
  panelContent.innerHTML = renderFrameSection(name)
    + renderExportSection();
  wireExportActions();
});
```

- [ ] **Step 5: Verify — click a section cut mark line → line panel appears with stroke, caps, etc.**

---

### Task 6: Clean up old panel references

**Files:**
- Modify: `index.html` — JS section

- [ ] **Step 1: Remove old panel element references**

Find and remove these lines (they reference elements that no longer exist):

```javascript
const propFloorSection = document.getElementById('propFloorSection');
const propOutputSection = document.getElementById('propOutputSection');
```

- [ ] **Step 2: Remove all references to propFloorSection and propOutputSection**

Search for `propFloorSection` and `propOutputSection` in the JS. Remove or comment out any lines that reference them (e.g., `propFloorSection.style.display = ...`). These are now handled by the renderer functions.

Specific lines to remove/update:
- In `selectExplodedGroup`: remove `propFloorSection.style.display` and `propOutputSection.style.display` lines
- In `convertToMultilayer`: remove `propFloorSection.style.display` and `propOutputSection.style.display` lines
- Any other function that sets display on these elements

- [ ] **Step 3: Remove old propGhostOpacity reference**

Find `const propGhostOpacity = document.getElementById('propGhostOpacity');` near the top of the multilayer section and remove it (the ghost opacity input is now created dynamically by `renderFloorSection`).

- [ ] **Step 4: Remove old ghost opacity event listener**

Find the standalone `propGhostOpacity.addEventListener('input', ...)` block and remove it (now wired inside `wireFloorActions()`).

- [ ] **Step 5: Verify — full test**

1. Open index.html
2. Click frame → panel shows Frame section only
3. Click "Multi-Layer" → panel shows Frame + Floor + Export
4. Switch floors via nav → Floor section updates (name, height)
5. Click "+" in Export → preset card appears
6. Toggle merge switch → button text updates
7. Click Section → draw line → choose side → cut mark appears
8. Click the cut mark → Line panel appears (stroke, caps)
9. Click canvas → deselects, panel hides
10. Escape works throughout

---

### Task 7: End-to-end verification

**Files:**
- No changes — testing only

- [ ] **Step 1: Multilayer Frame panel**

1. Open `index.html`
2. Click frame → "Frame" panel with Format, Size, Scale
3. Click Multi-Layer → "Residence Block A" panel with Floor section below
4. Floor section shows "Floor — Ground Floor" in blue
5. Story Height has ⓘ icon, hover shows tooltip
6. Ghost Opacity field present
7. Duplicate + Delete buttons present
8. Click Duplicate → new floor created, panel updates
9. Click Delete → floor removed, panel updates
10. Edit floor name → nav and header update

- [ ] **Step 2: Export section**

1. Click "+" → export preset appears
2. Shows "All Floors" + "PDF" + merge toggle
3. Toggle merge → button text changes
4. Click "+" again → second preset
5. Click ✕ on a preset → removed
6. Remove all → hint text reappears

- [ ] **Step 3: Section Cut Line panel**

1. Click Section → draw line → choose side → frame generated
2. Click the cut mark line on the frame
3. Panel shows: Section name, Stroke (color + hex + opacity), Weight, Solid/Dash, Start/End caps
4. Click elsewhere → deselects

- [ ] **Step 4: PDF Frame**

1. Click PDF frame → panel shows Frame section + Export
2. No Floor section (correct — PDF frames don't have floors)

- [ ] **Step 5: Exploded group**

1. Explode multilayer frame
2. Click group background → panel shows Frame section + Export
3. Click individual frame → panel shows Frame section + Floor section
