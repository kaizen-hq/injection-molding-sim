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
