# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

American Manufacturing 2026 - An injection molding factory simulation game celebrating USA's 250th Anniversary. Built as a single-file vanilla JavaScript game with no build system or dependencies.

## Development Commands

```bash
# Start local dev server (any of these work)
dotnet serve -p 3000 -d public
python -m http.server 3000 -d public
npx http-server public -p 3000

# Then visit http://localhost:3000
```

No build, lint, or test commands - pure static HTML/JS/CSS.

## Debug Mode

Press backtick (`` ` ``) in-game to toggle debug panel for testing:
- Adjust money, parts, revenue
- Change level (1-5)
- Test different game states

## Architecture

### Single-File Structure

The entire game is in `public/index.html` (~3000 lines):
- **Lines 1-1320**: CSS styles and HTML structure
- **Lines 1322-3095**: JavaScript game logic

### Core Game Systems

**Grid System**: 35x32 tile grid (`GRID_W`, `GRID_H`) with 40px tiles (`TILE_SIZE`)

**Classes**:
- `Tile` - Grid cell that can hold a building
- `Building` - Placed machines (sources, conveyors, processors, sellers)
- `Item` - Moving resources on conveyor belts

**Building Types** (defined in `ALL_BUILDINGS`):
- Sources: `SRC_P` (plastic), `SRC_S` (steel) and faster variants
- Transport: `BELT`, `BELT_F` (fast)
- Arms: `ARM`, `ARM_F` (filter), `ARM_S` (smart), `ARM_M` (mega)
- Processing: `CNC` variants (mill steel→molds), `PRESS` variants (molds+plastic→parts)
- Output: `BIN` (sell items)

**Tool Slots** (1-7): Each slot represents a building type with unlockable variants. Player level (1-4) gates which variants are available.

**Game Loop**:
- `update()` - Tick at 60fps, updates buildings and items
- `draw()` - Renders grid, buildings, items, and ghost preview
- `updateMetrics()` - Records production rates every second for graphs

### Key State Variables

```javascript
let money = 600;           // Current funds
let level = 1;             // Player level (unlocks variants)
let partsSoldTotal = 0;    // Progress toward level-ups
let selectedSlot = 1;      // Current tool slot (1-7)
let selectedVariants = {}; // Active variant per slot
let rotation = 0;          // Building placement rotation (0-3)
let grid = [];             // 2D array of Tiles
let buildings = [];        // All placed buildings
let items = [];            // All moving items
```

### Save/Load

Game auto-saves to `localStorage` key `injectionMoldingSave` on page unload and loads on startup.

### UI Components

- **Left Toolbar**: Tool slots with flyout variant selector
- **Right Sidebar**: Stats, production graphs, operator controls
- **Game Canvas**: Main grid where buildings are placed

### Keyboard Controls

- `1-7`: Select tool slot
- `Alt+1-4`: Select variant within current slot
- `Shift+1-7`: Cycle variants for slot
- `R`: Rotate building
- `L-Click`: Place building
- `R-Click`: Delete building
- `` ` ``: Toggle debug panel

## File Structure

```
public/           <- Deployable folder
├── index.html    <- All game code
├── images/       <- Flags, favicons
└── changes/      <- Changelog page

plans/            <- Design docs (not deployed)
```

## Deployment

Cloudflare Pages with build output directory set to `public`. No build command needed.

## DOM Manipulation Patterns

**CRITICAL**: This project follows a strict pattern for building interactive UI components to avoid DOM destruction anti-patterns.

### The Anti-Pattern (NEVER DO THIS)

```javascript
// ❌ WRONG - Destroys and recreates DOM every update
function updatePanel() {
    container.innerHTML = '';  // Destroys all elements including buttons
    container.appendChild(createButton());  // Recreates everything
}
setInterval(updatePanel, 500);  // Buttons get destroyed before clicks register
```

**Problems:**
- Interactive elements (buttons, inputs) are destroyed before user interactions can complete
- Event listeners must be constantly re-attached
- Causes flickering and poor performance
- Makes UI feel unresponsive and broken

### The Correct Pattern (ALWAYS DO THIS)

**Build structure with HTML/CSS, update values with JavaScript:**

```javascript
// ✅ CORRECT - Structure created once in HTML
<div id="status-panel">
    <span id="status-value">0</span>
    <button id="status-btn">Click Me</button>
</div>

// ✅ CORRECT - JavaScript only updates text/values
function updatePanel(value) {
    document.getElementById('status-value').textContent = value;
    // Button persists - no destruction, clicks work reliably
}

// ✅ CORRECT - Event listeners attached once during init
document.getElementById('status-btn').addEventListener('click', handler);
```

### Implementation Rules

1. **Structure in HTML**: Create the full HTML structure with all containers, buttons, and interactive elements
2. **Style in CSS**: Use CSS classes for all styling, states, and visibility
3. **JavaScript for values only**: Use JavaScript to:
   - Update `textContent`, `classList`, `style` properties
   - Show/hide elements with `display: none/block` or classes
   - Enable/disable buttons with `disabled` property
   - NEVER use `innerHTML = ''` or `appendChild()` on elements that update frequently

4. **Event listeners once**: Attach all event listeners during initialization, never in update functions

### Example: Dynamic List Pattern

When you need a dynamic list (like inventory items), use this pattern:

```javascript
// ✅ CORRECT - Pre-create max slots in HTML
<div id="inventory">
    <div class="inv-slot" id="slot-0" style="display: none;">
        <span class="inv-name"></span>
        <span class="inv-count"></span>
        <button class="inv-delete">DEL</button>
    </div>
    <div class="inv-slot" id="slot-1" style="display: none;">...</div>
    <!-- Create enough slots for max items -->
</div>

// ✅ CORRECT - Show/hide and update slots
function updateInventory(items) {
    items.forEach((item, i) => {
        const slot = document.getElementById(`slot-${i}`);
        slot.style.display = 'block';
        slot.querySelector('.inv-name').textContent = item.name;
        slot.querySelector('.inv-count').textContent = item.count;
        // Button persists, event listener stays attached
    });
    // Hide unused slots
    for (let i = items.length; i < MAX_SLOTS; i++) {
        document.getElementById(`slot-${i}`).style.display = 'none';
    }
}
```

### When This Pattern Applies

Use the persistent DOM pattern for:
- ✅ Panels that update frequently (every 200-500ms)
- ✅ Interactive elements (buttons, inputs, checkboxes)
- ✅ Status displays that change in real-time
- ✅ Any UI that responds to user clicks

You can use dynamic DOM creation for:
- ✅ One-time initialization
- ✅ Modals/dialogs that are created once and destroyed on close
- ✅ Elements that don't update frequently (< once per 5 seconds)

### Debugging DOM Issues

If buttons aren't clickable or UI feels unresponsive:
1. Check for `innerHTML = ''` in any update functions
2. Check for `appendChild()` in setInterval or frequent update loops
3. Verify event listeners are attached during init, not in update functions
4. Use browser DevTools to watch elements - they shouldn't disappear/reappear
