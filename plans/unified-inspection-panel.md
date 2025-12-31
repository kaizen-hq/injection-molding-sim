# Unified Inspection Panel System

## Goal
Replace the current arm-config modal and canvas-based inspector with a unified panel system in the flyout area. Panels should be compact (280px), visually consistent, and contextually relevant.

## Current State (Main Branch)
- **Inspector**: Positioned on canvas near clicked building, shows stats/buffers with delete buttons
- **Arm Config Modal**: Fixed center modal (500px), shows filter + direction picker (↑↓←→)
- **Tool-info-card**: Shows info about selected tool for placement
- **Variant list**: Shows variants of selected tool
- **Insufficient Funds**: Red flash notification on canvas when placement fails

## Design Decisions

### Flyout Independence
**CRITICAL**: The variants flyout and inspector panel are **completely independent and separate**.
- They occupy the same physical space but are distinct UI modes
- Inspector panel does NOT replace or modify the variants flyout
- When entering inspect mode, the variants flyout is **dimmed** (reduced opacity), not hidden
- The inspector panel appears as an overlay in the same area
- When exiting inspect mode, the dimmed variants flyout returns to full opacity

### Two Flyout States
1. **Build Mode** (default)
   - Variants flyout is **active and visible** (full opacity)
   - Shows: Tool slot variants + tool-info-card
   - Active when: No building inspected
   - **Clicking empty tile**: Places selected tool (or shows "INSUFFICIENT FUNDS" notification)
   - **Clicking building**: Enters Inspect Mode

2. **Inspect Mode** (when building clicked)
   - Variants flyout is **dimmed** (reduced opacity, 30-40%)
   - Inspector panel appears **on top** of dimmed flyout
   - Shows: Stack of inspection panels for selected building
   - Active when: User clicks any placed building
   - **Clicking empty tile**: Does nothing (no placement)
   - **Clicking another building**: Switches inspection to that building
   - **Exiting**: Click X button, press ESC, or press a tool hotkey (1-7)
   - On exit: Dimmed variants flyout returns to full opacity

This design ensures:
- Variants and inspector are completely separate systems
- No risk of state conflicts between build/inspect modes
- Clear visual distinction between modes (dimming)
- Smooth transitions without rebuilding DOM

### Panel Types (independent, stackable)

#### 1. Status Panel (all buildings)
- Building icon + name + variant
- Status indicator (OK/FULL/ERR)
- Key stats (cycles, items processed, efficiency)
- Compact: ~80px height

#### 2. Buffer Panel (buildings with input/output)
- **Input buffer**: visual slots showing contents with DELETE buttons per item type
- **Output buffer**: visual slots showing contents with DELETE button
- Uses `clearInv(type)` and `clearOutput()` functions from main branch
- Compact: ~60px height per buffer
- **IMPORTANT**: Buttons must persist across updates, not be destroyed/recreated

#### 3. Filter Panel (arms with canFilter)
- "ITEM FILTER" header
- 2x3 grid of filter buttons (compact)
- Currently selected filter highlighted
- Buttons: ⚡ ALL, 🔴 PLASTIC, ⬜ STEEL, 🟫 MOLD, 🟫⁴ QUAD MOLD, ⭐ PART
- Compact: ~100px height
- **IMPORTANT**: Buttons must persist across updates, not be destroyed/recreated

#### 4. Direction Panel (arms with canDirectDeliver)
- **"DELIVERY DIRECTION"** header (not "Direct Delivery")
- **3x3 compass grid** with ↑↓←→ arrow buttons
- Center shows 🦾 icon
- Selected direction highlighted
- Pickup direction button disabled (can't deliver back where you're picking from)
- Uses `deliverDir` property (0=RIGHT, 1=DOWN, 2=LEFT, 3=UP)
- Calls `setArmDirection(dir)` which updates `building.deliverDir` and shows notification
- Compact: ~120px height
- **IMPORTANT**: Buttons must persist across updates, not be destroyed/recreated
- **Reference**: See main branch lines 1376-1390 and 2662-2669 for exact implementation

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
│ [Direction Panel]       │
│ DELIVERY DIRECTION      │
│         ↑               │
│    ←   🦾   →           │
│         ↓               │
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
│ INPUT                   │
│ [🔴][🟫][ ][ ] [DEL]    │
│ OUTPUT                  │
│ [⭐][ ][ ][ ] [DEL]     │
└─────────────────────────┘
```

## Implementation Architecture

### Critical Issue: DOM Replacement Strategy

**PROBLEM (Current Implementation)**:
- `setInterval(updateInspectPanels, 500)` calls `renderInspectPanels()` every 500ms
- `renderInspectPanels()` does `container.innerHTML = ''` destroying all panels
- All panels are recreated from scratch every 500ms
- Interactive elements (buttons) get destroyed before clicks can register
- Event listeners must be re-attached constantly

**SOLUTION (Required Fix)**:
1. **Create HTML structure once** when entering inspect mode
2. **Only update text/values** during periodic refreshes
3. **Keep buttons persistent** in the DOM with stable event listeners
4. **Show/hide panels** based on building type rather than destroying/recreating

### Implementation Approach

#### Phase 1: Create Persistent HTML Structure
```javascript
function createInspectPanelStructure(building) {
    // Create panel divs with IDs for each section
    // Add all buttons, labels, containers
    // Attach event listeners ONCE
    // Return container with all panels
}
```

#### Phase 2: Update Values Only
```javascript
function updateInspectPanelValues(building) {
    // Use getElementById/querySelector to find existing elements
    // Update textContent, classList, style properties
    // DO NOT use innerHTML = '' or appendChild
    // DO NOT destroy/recreate buttons
}
```

#### Phase 3: Show/Hide Relevant Panels
```javascript
function showRelevantPanels(building) {
    // Show/hide panels based on building.type properties
    // if (building.type.hasInventory) show buffer panel
    // if (building.type.canFilter) show filter panel
    // if (building.type.canDirectDeliver) show direction panel
}
```

## Implementation Steps

### Phase 0: Add Dimming Effect (Independent Flyouts) ✅ COMPLETED
- [x] Add CSS for `.dimmed` class on `#build-mode` (opacity: 0.35)
- [x] Make `#inspect-mode` position absolute to overlay on top
- [x] Add `position: relative` to `#flyout-panel` for absolute positioning context
- [x] Update `enterInspectMode()` to add .dimmed class instead of display:none
- [x] Update `exitInspectMode()` to remove .dimmed class instead of display:flex

### Phase 1: Fix DOM Replacement Issue ⏳ IN PROGRESS
**Goal**: Stop destroying/recreating panels every 500ms. Create structure once, update values only.

**Current Problem**:
- Line 2014: `setInterval(updateInspectPanels, 500)` → calls renderInspectPanels
- Line 2660: `container.innerHTML = ''` → destroys all panels
- Lines 2661-2677: All panels recreated from scratch
- Lines 2789, 2833, 2852: Button clicks call renderInspectPanels → destroys panels

**Implementation Checklist**:
- [x] Create `createInspectPanelStructure(building)` function (lines 2897-2997)
  - [x] Create status panel HTML with IDs for dynamic values
  - [x] Create buffer panel HTML with IDs for buffer slots
  - [x] Create filter panel HTML with persistent buttons
  - [x] Create direction panel HTML with persistent buttons (placeholder for Phase 4)
  - [x] Show/hide panels based on building.type properties
  - [x] Attach event listeners to buttons ONCE
  - [x] Return nothing (updates DOM directly)

- [x] Create `updateInspectPanelValues(building)` function (lines 2999-3058)
  - [x] Update status panel text (icon, name, status, stats)
  - [x] Update buffer panel slots (add/remove filled class, update icons)
  - [x] Update filter panel button active states
  - [x] Update direction panel button active states (placeholder)
  - [x] Use getElementById/querySelector, NOT innerHTML
  - [x] Only modify textContent, classList, style properties

- [x] Update `enterInspectMode(building)` function (lines 2637-2641)
  - [x] Call `createInspectPanelStructure(building)` instead of renderInspectPanels
  - [x] Initial value update with `updateInspectPanelValues(building)`

- [x] Update button click handlers
  - [x] Filter buttons: Update building.filter, call updateInspectPanelValues (line 2989-2991)
  - [x] Direction buttons: Implemented in Phase 4 (lines 2158-2174)
  - [x] clearInv/clearOutput now call updateInspectPanelValues (lines 2879, 2887)

- [x] Update setInterval on line 2027
  - [x] Change from `updateInspectPanels` to custom function
  - [x] Custom function: `if (isInspectMode && inspectedBuilding) updateInspectPanelValues(inspectedBuilding)`

- [x] Clean up old functions
  - [x] Removed `updateInspectPanels()` function
  - [x] Removed `renderInspectPanels()` function
  - [x] Removed all old render functions (renderStatusPanel, renderBufferPanel, renderFilterPanel, renderDeliveryPanel, getItemIcon)
  - [x] Kept `getBuildingStatus()` function (still used by updateInspectPanelValues)

### Phase 2: Restore Insufficient Funds Notification ✅ COMPLETED
- [x] Verified `#notification` element exists in HTML (line 1446)
- [x] Verified CSS `.notification` styles are present (lines 945-950)
- [x] Checked z-index: notification is z-index 50
- [x] Verified `showNotification("INSUFFICIENT FUNDS")` fires on placement failure (line 2718)
- [x] Function implementation correct (lines 3019-3023)

### Phase 3: Add Delete Buttons to Buffer Panel ✅ COMPLETED
- [x] Restored `clearInv(type)` function (lines 2830-2835)
- [x] Restored `clearOutput()` function (lines 2836-2842)
- [x] Added delete buttons next to each input item type in updateInspectPanelValues (lines 2913-2926)
- [x] Added delete button for output buffer (lines 2928-2960)
- [x] Wired up buttons to call clearInv/clearOutput with event listeners

### Phase 4: Replace Delivery Panel with Direction Panel ✅ COMPLETED
- [x] **REMOVED** `renderDeliveryPanel()` function
- [x] **REMOVED** all references to `directDeliver` and `deliverTarget` properties
- [x] **ADDED** Direction panel HTML structure (lines 1372-1386)
- [x] Added 3x3 grid with ↑↓←→ buttons using `.direction-grid` and `.direction-btn` classes
- [x] Center cell shows 🦾 icon
- [x] Restored `setArmDirection(dir)` function (lines 2844-2852)
- [x] Added CSS for direction panel (lines 546-578)
- [x] Attached direction button event listeners once during init (lines 2158-2174)
- [x] Implemented logic to disable pickup direction button (lines 2999-3001)
- [x] Implemented logic to highlight current `building.deliverDir` selection (lines 3004-3007)
- [x] Shows notification when direction changes (line 2848)

### Phase 5: Cleanup ✅ COMPLETED
- [x] Removed old `#inspector` from canvas (already done)
- [x] Removed old `#arm-config` modal (already done)
- [x] Removed renderDeliveryPanel and all directDeliver/deliverTarget code
- [x] All panels now use persistent DOM pattern (no innerHTML destruction)

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

/* Direction picker buttons (from main) */
.direction-grid {
    display: grid;
    grid-template-columns: repeat(3, 60px);
    gap: 8px;
    margin: 0 auto;
    width: fit-content;
}

.direction-btn {
    padding: 15px;
    font-size: 20px;
    background: var(--panel-bg);
    border: 1px solid #444;
    color: #fff;
    cursor: pointer;
}

.direction-btn.active {
    border-color: var(--gold);
    box-shadow: 0 0 8px var(--gold);
}

.direction-btn:disabled {
    opacity: 0.3;
    cursor: not-allowed;
}
```

## Reference Implementation (Main Branch)

### Buffer Panel with Delete Buttons
- **Location**: Lines 2492-2530 in main/public/index.html
- **Functions**: `clearInv(type)` (2478-2483), `clearOutput()` (2484-2490)
- **Event handling**: Lines 1880-1886 (click delegation)

### Direction Picker
- **HTML**: Lines 1376-1390 in main/public/index.html
- **Logic**: Lines 2616-2649 (openArmConfig direction section)
- **Function**: `setArmDirection(dir)` (2662-2669)
- **Property**: `building.deliverDir` (0=RIGHT, 1=DOWN, 2=LEFT, 3=UP)

### Insufficient Funds Notification
- **HTML**: Line ~800 in main/public/index.html (`<div id="notification">`)
- **CSS**: Lines 727-732 (.notification styles)
- **Function**: `showNotification(msg)` (2565-2569)
- **Usage**: Line 2417 (placeBuilding function)

## Known Issues to Fix

### Issue #1: DOM Destruction Pattern
- **Problem**: Panels recreated every 500ms, destroying interactive elements
- **Location**: Line 2014, 2644, 2667 in unified-inspector
- **Fix**: Create structure once, update values only

### Issue #2: Insufficient Funds Not Showing
- **Problem**: Notification might be hidden by z-index or missing element
- **Location**: Notification element and CSS
- **Fix**: Verify element exists, check z-index conflicts

### Issue #3: Missing Delete Buttons in Buffers
- **Problem**: Buffer panel doesn't show delete buttons for items
- **Location**: `renderBufferPanel()` in unified-inspector (lines 2713-2768)
- **Fix**: Add delete buttons, wire up clearInv/clearOutput functions

### Issue #4: Wrong Delivery Feature
- **Problem**: "Direct delivery to seller" implemented instead of "Direction picker"
- **Location**: `renderDeliveryPanel()` in unified-inspector (lines 2810-2857)
- **Fix**: Replace with direction picker grid (↑↓←→)

## Answered Design Questions

1. **Real-time updates**: Panels auto-refresh every 500ms while in inspect mode
   - **Update**: Only VALUES refresh, structure persists

2. **Multiple selection**: Clicking another building switches inspection (single selection)
   - **Status**: ✓ Implemented correctly

3. **Keyboard shortcut**: ESC exits inspect mode and returns to build mode
   - **Status**: ✓ Should be implemented

4. **Canvas highlight**: Yellow dashed outline on inspected building
   - **Status**: ✓ Should be implemented

## Files to Modify
- `public/index.html` (HTML structure, CSS, JavaScript)

## Dependencies
- None (self-contained refactor)

## Estimated Complexity
Medium-High - Requires:
- Refactoring DOM update pattern from destructive to persistent
- Migrating direction picker from modal to panel
- Restoring delete button functionality
- Fixing notification display issue
