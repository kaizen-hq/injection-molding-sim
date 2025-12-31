# Unified Inspection Panel System

## Goal
Replace the current arm-config modal and canvas-based inspector with a unified panel system in the flyout area. Panels should be compact (280px), visually consistent, and contextually relevant.

## Current State
- **Inspector**: Positioned on canvas near clicked building, shows stats/buffers
- **Arm Config Modal**: Fixed center modal (500px), shows filter + direct delivery options
- **Tool-info-card**: Shows info about selected tool for placement
- **Variant list**: Shows variants of selected tool

## Design Decisions

### Two Flyout Modes
1. **Build Mode** (default)
   - Header: "[TOOL] VARIANTS"
   - Content: Variant list + tool-info-card
   - Active when: No building inspected, or user clicks X on inspection
   - **Clicking empty tile**: Places selected tool
   - **Clicking building**: Switches to Inspect Mode

2. **Inspect Mode** (when building clicked)
   - Header: "INSPECTING: [BUILDING NAME]" with X button
   - Content: Stack of applicable inspection panels
   - Active when: User clicks any placed building
   - **Clicking empty tile**: Does nothing (no placement)
   - **Clicking another building**: Switches inspection to that building
   - **Exiting**: Click X button, press ESC, or press a tool hotkey (1-7)

This separation prevents accidental placement while inspecting and makes the two modes feel distinct.

### Panel Types (independent, stackable)

#### 1. Status Panel (all buildings)
- Building icon + name + variant
- Status indicator (OK/FULL/ERR)
- Key stats (cycles, items processed, efficiency)
- Compact: ~80px height

#### 2. Buffer Panel (buildings with input/output)
- Input buffer: visual slots showing contents
- Output buffer: visual slots showing contents
- Compact: ~60px height per buffer

#### 3. Filter Panel (arms with canFilter)
- "ITEM FILTER" header
- 2x3 grid of filter buttons (compact)
- Currently selected filter highlighted
- Compact: ~100px height

#### 4. Delivery Panel (arms with canDirectDeliver)
- "DIRECT DELIVERY" header
- Toggle: ON/OFF
- Target selector (when ON): dropdown or mini-grid
- Compact: ~80px height

### Panel Stacking Example
Clicking a **Mega Arm** would show:
```
┌─────────────────────────┐
│ INSPECTING: MEGA ARM  X │
├─────────────────────────┤
│ [Status Panel]          │
│ 🦾 MEGA ARM             │
│ ● OK  |  Cycles: 847    │
├─────────────────────────┤
│ [Filter Panel]          │
│ ITEM FILTER             │
│ [⚡ALL] [🔴] [⬜]       │
│ [🟫  ] [🟫4] [⭐]       │
├─────────────────────────┤
│ [Delivery Panel]        │
│ DIRECT DELIVERY: [ON]   │
│ Target: SELLER #3       │
└─────────────────────────┘
```

Clicking a **Press** would show:
```
┌─────────────────────────┐
│ INSPECTING: PRESS     X │
├─────────────────────────┤
│ [Status Panel]          │
│ 🏭 INJECTION PRESS      │
│ ● OK  |  Parts: 234     │
├─────────────────────────┤
│ [Buffer Panel]          │
│ IN: [🔴][🟫][ ][ ]      │
│ OUT: [⭐][ ][ ][ ]      │
└─────────────────────────┘
```

## Implementation Steps

### Phase 1: Create Panel Infrastructure
1. Add `#inspect-mode` container in flyout-panel (hidden by default)
2. Add CSS for `.inspect-panel` base styles
3. Add `enterInspectMode(building)` and `exitInspectMode()` functions
4. Wire up: clicking building calls `enterInspectMode()`, X button calls `exitInspectMode()`

### Phase 2: Status Panel
1. Create `renderStatusPanel(building)` function
2. Show building icon, name, variant, status dot
3. Show relevant stats based on building type

### Phase 3: Buffer Panel
1. Create `renderBufferPanel(building)` function
2. Show input/output buffers as compact slot grids
3. Update in real-time as items move

### Phase 4: Filter Panel (migrate from arm-config)
1. Create `renderFilterPanel(building)` function
2. Compact 2x3 button grid
3. Reuse existing filter logic

### Phase 5: Delivery Panel (migrate from arm-config)
1. Create `renderDeliveryPanel(building)` function
2. Compact toggle + target selector
3. Reuse existing direct delivery logic

### Phase 6: Cleanup
1. Remove old `#inspector` from canvas
2. Remove old `#arm-config` modal
3. Remove associated CSS
4. Update click handlers

## Visual Style (compact, 280px width)

```css
.inspect-panel {
    background: rgba(26, 38, 52, 0.95);
    border: 2px solid var(--usa-blue);
    border-radius: 6px;
    padding: 10px;
    margin-bottom: 8px;
}

.inspect-panel-header {
    color: var(--gold);
    font-size: 11px;
    font-weight: bold;
    margin-bottom: 6px;
    text-transform: uppercase;
}

.inspect-panel .stat-row {
    display: flex;
    justify-content: space-between;
    font-size: 11px;
    padding: 2px 0;
}
```

## Open Questions

1. **Real-time updates**: Should panels auto-refresh while open, or only on re-click?
   - Recommendation: Auto-refresh every 500ms while in inspect mode

2. **Multiple selection**: Should clicking another building switch inspection, or allow multi-select?
   - Recommendation: Switch to new building (single selection)

3. **Keyboard shortcut**: Should ESC exit inspect mode?
   - Recommendation: Yes, ESC returns to build mode

4. **Canvas highlight**: Keep yellow dashed outline on inspected building?
   - Recommendation: Yes, helps identify what's being inspected

## Files to Modify
- `public/index.html` (HTML structure, CSS, JavaScript)

## Dependencies
- None (self-contained refactor)

## Estimated Complexity
Medium - mostly reorganizing existing functionality into new layout
