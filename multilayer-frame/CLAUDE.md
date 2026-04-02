# Odin Toolbar – Project Context

## Was ist das?

Prototyp einer floating Bottom-Toolbar für **Odin** – ein Architektur/Grundriss-Designtool (ähnlich wie Figma, aber für Floor Plans). Die gesamte UI ist als **einzelne `index.html`** gebaut (Vanilla JS, inline CSS, inline SVG).

## Projektstruktur

```
odin toolbar/
├── index.html      # Gesamte App – CSS, HTML, JS in einer Datei
└── CLAUDE.md
```

## Toolbar-Komponenten

Die Toolbar besteht aus mehreren **Pills** (abgerundete Kapseln), die nebeneinander am unteren Bildschirmrand zentriert schweben:

| Pill | ID | Beschreibung |
|---|---|---|
| Search-Pill | `.search-pill` | 48×48px Button mit Odin-AI-Icon; expandiert zur Command Bar |
| Tools Pill | `#toolsPill` | Alle Zeichenwerkzeuge nebeneinander (48px hoch) |
| Mode Pill | `#modePill` | Toggle zwischen Draft Mode und Polish Mode |
| Tool Submenu | `#toolSubmenu` | Popup-Menü über Chevron-Tools (glassmorphism) |

## Zustände / Interaktionen

### Search öffnen (Klick auf Odin-Button oder `/` drücken)
- Search-Pill expandiert von 48px auf die Breite der Tools-Pill (smooth width transition)
- `.open` Klasse: `justify-content: flex-start`, `padding: 8px 8px 8px 16px`, `gap: 8px`
- Input wird sichtbar (`flex: 1; opacity: 1`)
- Tools-Pill schrumpft auf 48px (zeigt nur aktives Tool via `translateX` auf `.tools-inner`)
- Mode-Pill bleibt unverändert
- Command-Dropdown zeigt gefilterte Befehle beim Tippen

### Search schließen (Escape oder Klick außerhalb)
- `.closing` Klasse wird gesetzt (behält `flex-start` Position, kollabiert Input sofort)
- Width animiert zurück auf 48px
- Nach 400ms: `.closing` wird entfernt → Icon settelt in `justify-content: center`
- Tools-Pill expandiert zurück auf volle Breite

### Tool wechseln
- Klick auf Tool-Button → `.tool-icon.active` wird auf neues Tool gesetzt
- `activeToolId` wird aktualisiert

### Chevron Submenu
- Klick auf `.tool-chevron` → öffnet `.tool-submenu` Popup über dem Tool
- Menü links-bündig mit dem Chevron-Button positioniert
- Chevron bekommt `.active` State (hellerer Hintergrund)
- Items: Checkmark (✓) für aktives Sub-Tool, Icon, Label, Shortcut
- Hover-State auf Items: `#0099FF` Hintergrund
- Toggle: zweiter Klick schließt, Escape/Klick außerhalb auch

### Mode wechseln
- Klick auf Draft/Polish Mode Button → Glow-Effekt bewegt sich

## Design-System

### Farben
- Background Canvas: `#1a1a1a`
- Pill Background: `rgba(21, 23, 25, 0.7)` mit `backdrop-filter: blur(12px)`
- Pill Border: `1px solid rgba(255, 255, 255, 0.1)`
- Active Tool (blau): `#0099FF`
- Text primär: `#F8FAFC`
- Text sekundär/inaktiv: `#A1A3A5`
- Draft Mode Glow: `#33B5FF`
- Polish Mode Glow: `#FFD700`
- Submenu Hover: `#0099FF`
- Chevron Active: `rgba(255, 255, 255, 0.14)`

### Typography
- Font: `Inter` (Google Fonts), weights 400 + 500
- Tool-Labels / Submenu Labels: 11px, font-weight 500
- Submenu Shortcuts: 10px, font-weight 500, `rgba(255, 255, 255, 0.54)`

### Pills
- Border-radius: `9px`
- Höhe: `48px` (alle Pills)
- Box-shadow: mehrstufig (5 Stufen, 0px–65px, subtil)

### Tool-Buttons
- Icon-Container: 36×36px, border-radius 6px
- Aktiver Zustand: `#0099FF` Hintergrund + `border-radius: 4px`
- Hover: `rgba(255, 255, 255, 0.08)`
- Icons: 16×16px

### Chevron-Buttons
- Höhe: 36px (gleich wie Tool-Icon)
- Padding: `0 4px`
- Border-radius: 6px
- Active State: `rgba(255, 255, 255, 0.14)`

### Tool Submenu
- Border-radius: `11px`, padding: `4px`
- Items: 32px hoch, border-radius 8px, gap 8px
- Check-Slot: 12×12px, Icon-Slot: 16×16px
- Animation: fade-in + slide-up (150ms)

### Animationen
- Haupt-Übergänge: `cubic-bezier(0.4, 0, 0.2, 1)`, 350ms
- Submenu: 150ms `cubic-bezier(0.2, 0, 0, 1)`
- Tool-Hover: 120ms ease

## Tools in der Toolbar (v.l.n.r.)

1. **Cursor** – Pfeil-Cursor (kein Chevron)
2. **Frame** – 4 Corner L-Shapes (kein Chevron)
3. *(Separator)*
4. **Rectangle** – Rechteck (kein Chevron)
5. **Connect/Polyline** – Aktiv by default (blau), mit Chevron → Submenu: Line, Polyline (aktiv), Pencil
6. *(Separator)*
7. **Wall** – Brick-Grid (kein Chevron)
8. **Room** – Cross-Grid (kein Chevron)
9. **Stair** – Kurve, mit Chevron → Submenu: L-Stair, Spiral
10. *(Separator)*
11. **Ruler** – Parallelogramm mit Ticks (kein Chevron)
12. **Pen** – Freehand-Linie (kein Chevron)
13. **Text** – T-Icon, mit Chevron → Submenu: Label, Annotation
14. *(Separator)*
15. **Maximize** – Corner-Arrows, mit Chevron → Submenu: Fit View, Zoom In, Zoom Out

## Konventionen

- Alle Icons sind **inline SVG** (kein externes Icon-Set)
- Kein Framework, kein Build-Step – pure HTML/CSS/JS
- Animations-State wird über `.active`, `.visible`, `.open`, `.closing`-Klassen + inline Style-Overrides gesteuert
- Die Command-Dropdown-Items werden dynamisch aus dem `commands`-Array gerendert
- Submenu-Items werden dynamisch aus dem `SUBMENUS`-Objekt gerendert
- Aktive Sub-Tools werden in `activeSubItem`-Objekt getrackt (z.B. `{ connect: 'polyline', ... }`)
- Search-Pill Icon-Zentrierung: `justify-content: center` + `gap: 0` + Input `width: 0; flex: none` wenn geschlossen
- Close-Animation: `.closing`-Klasse hält Icon-Position stabil während Width-Transition

## Architektur-Gotchas (aus Debugging-Sessions)

### Zwei-Stufen-Close für Search/Odin
- `closePalette()` = Step 1: nur Dropdown verstecken, Bar bleibt expanded
- `closeSearch()` = Step 2: kompletter Collapse (Bar + Dropdown)
- State-Check: `isSearchOpen` = Bar expanded, `commandDropdown.classList.contains('visible')` = Palette sichtbar — diese sind unabhängig!

### Event Propagation — häufigste Bug-Quelle
- Nested click handlers: `searchPill > commandDropdown > .command-item` — Clicks bubblen hoch!
- **Immer** `e.stopPropagation()` in child-Handlern nutzen, sonst triggert der parent-Handler unerwünscht
- Dropdown-Item-Clicks werden per **Event Delegation** auf `commandDropdown` gehandelt, NICHT per-item in `renderPalette()` (vermeidet Memory Leaks bei jedem Re-Render)

### Collapsed Toolbar Width
- 48px ohne Chevron, 58px mit Chevron
- Invariante: `searchPillWidth + toolsPillWidth = konstant` während Animationen
- `toolsPillWidth` wird bei Load gemessen + bei Resize aktualisiert (nur wenn Search geschlossen)

### Context Bar (polyCtxBar) Position
- Wird per `translateX` verschoben wenn Search offen ist
- Muss synchronisiert werden in: `showCtxBar()`, `openSearch()`, `closePalette()`, `closeSearch()`
- `showCtxBar()` prüft `isSearchOpen` und berechnet Offset selbst

### Timer Management
- `openSearch()` nutzt `setTimeout` (350ms overflow, 200ms focus) — IDs in `openSearchTimer`/`openSearchFocusTimer`
- `closeSearch()` muss diese Timer canceln (User kann vor Ablauf schließen)

### Polyline SVG Handler
- SVG deckt gesamten Canvas ab → Click-Handler MUSS `activeToolId === 'connect'` prüfen
- Zweiter Click-Handler stoppt Propagation nur wenn Polyline aktiv + Palette nicht sichtbar
- `plSnapAng` wird in `plDeactivate()` zurückgesetzt

## Offene Punkte / Bekannte TODOs

- Command-Items haben noch keine echte Funktionalität (nur `closeSearch()` beim Klick)
- Kein responsives Verhalten (fixed für Desktop konzipiert)
- Sub-Tool Auswahl ändert noch nicht das Toolbar-Icon
