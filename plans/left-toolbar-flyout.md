# Left Sidebar Toolbar with Flyout Selection

## Design Vision

**Move toolbar from bottom to left sidebar with flyout menus for variant selection.**

### Core Principles

1. **Slot = Building Type** (not variant)
   - Slot 2 is always "ARM" (shows all arm variants)
   - Slot 5 is always "CNC" (shows all CNC variants)
   - etc.

2. **Flyout Always Visible**
   - Flyout permanently shows variants for currently selected slot
   - When switching slots, flyout animates to show new variants
   - No open/close behavior - always present

3. **Click to Switch Slots**
   - Click slot → Flyout updates to show that slot's variants
   - Smooth animation between variant lists
   - Currently selected slot highlighted

4. **Shift+Number Cycles Variants**
   - Shift+2 → Next ARM variant
   - Flyout updates to show new selection
   - Quick switching without clicking

5. **Currently Selected Shows on Toolbar**
   - Main button displays active variant icon/name
   - Visual feedback of current selection

6. **Level/Lock Status in Flyout**
   - Locked variants grayed out
   - Show "Unlock at Level X"

---

## Layout Mockup

### Left Sidebar Structure (Always-Visible Flyout)

```
┌──────────────────┐  ┌─────────────────────┐
│                  │  │ ARM VARIANTS        │ ← Always visible
│  TOOLS           │  │                     │    Shows variants
│  ──────────      │  │ [1] 🦾 ARM     $800 │    for currently
│                  │  │     Level 1         │    selected slot
│  [🚗] BELT    1  │  │                     │
│  [🦾S] ARM*   2  │──│ [2] 🦾+ FAST $1,500│ ← Slot 2 selected
│  [🔴M] PLASTIC 3 │  │     Level 2         │    (highlighted)
│  [⬜] STEEL    4  │  │                     │
│  [⚙️] CNC      5  │  │ [3] 🦾S SMART $1,800│ ⭐
│  [🏭] PRESS    6  │  │     Level 3         │
│  [💲] SELL     7  │  │                     │
│                  │  │ [4] 🦾M MEGA $2,000 │
└──────────────────┘  │     Level 4 🔒      │
    ~150px wide       │                     │
                      └─────────────────────┘
                          ~250px wide

                      Press 1-4 to select variant
                      ⭐ = Currently selected
                      🔒 = Locked
```

### When Switching to PLASTIC (Slot 3)

```
┌──────────────────┐  ┌─────────────────────┐
│  TOOLS           │  │ PLASTIC VARIANTS    │ ← Animates
│  ──────────      │  │                     │    from ARM
│                  │  │ [1] 🔴 SOURCE  $150 │    to PLASTIC
│  [🚗] BELT    1  │  │     Level 1         │    variants
│  [🦾S] ARM    2  │  │                     │
│  [🔴M] PLASTIC*3 │──│ [2] 🔴+ FAST   $300│ ← Slot 3 selected
│  [⬜] STEEL    4  │  │     Level 2         │
│  [⚙️] CNC      5  │  │                     │
│  [🏭] PRESS    6  │  │ [3] 🔴M MEGA   $800│ ⭐
│  [💲] SELL     7  │  │     Level 4         │
│                  │  │                     │
└──────────────────┘  └─────────────────────┘

Flyout content cross-fades to new variants (0.2s animation)
```

---

## Flyout Animation Behavior

### Always Visible
- Flyout is **permanently visible** (not a modal/popup)
- Positioned to the right of the toolbar (~160px from left edge)
- Part of the left sidebar UI, not an overlay

### Switching Slots (Animated Transition)

When user switches slots (press `2` → `3` or click different slot):

**Animation sequence:**
1. **Fade out** current variants (0.1s)
2. **Update** flyout header and content
3. **Fade in** new variants (0.1s)
4. **Total:** 0.2s smooth cross-fade

**CSS:**
```css
#flyout-content {
    transition: opacity 0.1s ease-in-out;
}

/* During transition */
#flyout-content.fading {
    opacity: 0;
}
```

**JavaScript:**
```javascript
function switchToSlot(slotNumber) {
    activeSlot = slotNumber;

    const content = document.getElementById('flyout-content');

    // Fade out
    content.classList.add('fading');

    // Wait for fade out, then update content
    setTimeout(() => {
        updateFlyoutContent(slotNumber);

        // Fade in
        content.classList.remove('fading');
    }, 100);

    // Update toolbar highlights
    updateToolbarDisplay();
}
```

### Selecting Variant (No Animation)

When selecting a variant (Alt+2, clicking, or Shift+cycling):
- **No flyout animation** (content stays visible)
- Only updates: toolbar button icon/name, selected indicator (⭐)
- Instant feedback

---

### SELL Slot (Single Variant)

```
┌──────────────────┐  ┌─────────────────────┐
│  TOOLS           │  │ SELL VARIANTS       │
│  ──────────      │  │                     │
│                  │  │ [1] 💲 BIN     $100 │ ⭐
│  [🚗] BELT    1  │  │     Level 1         │
│  [🦾S] ARM    2  │  │                     │
│  [🔴M] PLASTIC 3 │  │ Market export.      │
│  [⬜] STEEL    4  │  │ Sells items.        │
│  [⚙️] CNC      5  │  │                     │
│  [🏭] PRESS    6  │  │ (Future variants?)  │
│  [💲] SELL*    7 │──│                     │
│                  │  │                     │
└──────────────────┘  └─────────────────────┘

Even single-variant slots show in flyout for UI consistency
```

---

## Keyboard Controls

### Primary Controls

| Key | Action |
|-----|--------|
| `1-7` | Switch to building type slot (updates flyout) |
| `Alt+1-4` | Select variant 1-4 from current flyout (Alt = Alternative) |
| `Shift+1-7` | Quick cycle to next variant for that slot |
| `R` | Rotate building |

### Example Workflows

**Workflow 1: Select SMART ARM**
1. Press `2` → Switch to ARM slot (flyout updates to show ARM variants)
2. Press `Alt+3` → SMART ARM selected (3rd variant)
3. Now placing SMART ARM

**Workflow 2: Quick cycle through ARMs**
1. Press `Shift+2` → Next ARM variant
2. Press `Shift+2` again → Next variant
3. Cycles: ARM → FAST → SMART → MEGA → ARM...

**Workflow 3: Quick switch to save money**
1. Currently: SMART ARM selected
2. Press `2` → ARM slot active (flyout shows ARM variants, SMART highlighted)
3. Press `Alt+1` → Basic ARM selected
4. Saves $1,000 for this placement

**Workflow 4: One-handed operation**
1. Press `5` → CNC slot
2. Press `Alt+2` → FAST CNC
3. Place building
4. All with left hand only!

---

## Data Structure

### Building Type Slots

```javascript
const BUILDING_SLOTS = [
    {
        slot: 1,
        type: 'BELT',
        variants: ['BELT', 'BELT_F'],
        icon: '🚗',
        name: 'BELT'
    },
    {
        slot: 2,
        type: 'ARM',
        variants: ['ARM', 'ARM_F', 'ARM_S', 'ARM_M'],
        icon: '🦾',
        name: 'ARM'
    },
    {
        slot: 3,
        type: 'PLASTIC',
        variants: ['SRC_P', 'SRC_P_F', 'SRC_P_M'],
        icon: '🔴',
        name: 'PLASTIC'
    },
    {
        slot: 4,
        type: 'STEEL',
        variants: ['SRC_S', 'SRC_S_F'],
        icon: '⬜',
        name: 'STEEL'
    },
    {
        slot: 5,
        type: 'CNC',
        variants: ['CNC', 'CNC_F', 'CNC_M'],
        icon: '⚙️',
        name: 'CNC'
    },
    {
        slot: 6,
        type: 'PRESS',
        variants: ['PRESS', 'PRESS_M'],
        icon: '🏭',
        name: 'PRESS'
    },
    {
        slot: 7,
        type: 'SELL',
        variants: ['BIN'],
        icon: '💲',
        name: 'SELL'
    }
];
```

### Selection State

```javascript
// Track currently selected variant for each slot
let selectedVariants = {
    1: 'BELT',      // Slot 1: Basic belt
    2: 'ARM',       // Slot 2: Basic arm
    3: 'SRC_P',     // Slot 3: Basic plastic source
    4: 'SRC_S',     // Slot 4: Basic steel source
    5: 'CNC',       // Slot 5: Basic CNC
    6: 'PRESS',     // Slot 6: Basic press
    7: 'BIN'        // Slot 7: Sell bin
};

// Which flyout is currently open (null if closed)
let openFlyout = null;  // 1-7 or null
```

---

## UI Implementation

### Left Sidebar HTML (Declarative - No JS String Building)

```html
<!-- Left Toolbar -->
<div id="left-toolbar">
    <div class="toolbar-header">TOOLS</div>

    <!-- Tool Slots -->
    <div class="tool-slot" data-slot="1" data-type="BELT">
        <div class="tool-icon">🚗</div>
        <div class="tool-name">BELT</div>
        <div class="tool-hotkey">1</div>
    </div>

    <div class="tool-slot" data-slot="2" data-type="ARM">
        <div class="tool-icon">🦾</div>
        <div class="tool-name">ARM</div>
        <div class="tool-hotkey">2</div>
    </div>

    <!-- ... slots 3-7 ... -->
</div>

<!-- Flyout (Always Visible, Next to Toolbar) -->
<div id="flyout-panel">
    <div id="flyout-header">ARM VARIANTS</div>

    <!-- All variants pre-rendered in HTML -->
    <!-- JavaScript toggles .hidden class based on active slot -->

    <!-- BELT Variants -->
    <div class="flyout-variant" data-slot="1" data-building="BELT">
        <div class="variant-icon">🚗</div>
        <div class="variant-name">BELT</div>
        <div class="variant-cost">$20</div>
        <div class="variant-level">Level 1</div>
        <div class="variant-hotkey">Alt+1</div>
    </div>

    <div class="flyout-variant" data-slot="1" data-building="BELT_F">
        <div class="variant-icon">🚗+</div>
        <div class="variant-name">FAST BELT</div>
        <div class="variant-cost">$40</div>
        <div class="variant-level">Level 2</div>
        <div class="variant-hotkey">Alt+2</div>
    </div>

    <!-- ARM Variants -->
    <div class="flyout-variant hidden" data-slot="2" data-building="ARM">
        <div class="variant-icon">🦾</div>
        <div class="variant-name">ARM</div>
        <div class="variant-cost">$800</div>
        <div class="variant-level">Level 1</div>
        <div class="variant-hotkey">Alt+1</div>
    </div>

    <div class="flyout-variant hidden" data-slot="2" data-building="ARM_F">
        <div class="variant-icon">🦾+</div>
        <div class="variant-name">FAST ARM</div>
        <div class="variant-cost">$1,500</div>
        <div class="variant-level">Level 2</div>
        <div class="variant-hotkey">Alt+2</div>
    </div>

    <!-- ... all other variants ... -->

    <!-- JavaScript adds/removes these classes: -->
    <!-- .hidden - Hide variant (wrong slot) -->
    <!-- .locked - Grayed out (not unlocked) -->
    <!-- .selected - Highlight (currently selected) -->
</div>
```

### Tool Slot CSS

```css
.tool-slot {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 10px;
    background: rgba(255, 255, 255, 0.05);
    border: 2px solid transparent;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s;
    position: relative;
}

.tool-slot:hover {
    background: rgba(255, 255, 255, 0.1);
    border-color: var(--usa-blue);
}

.tool-slot.selected {
    border-color: var(--industrial-orange);
    background: rgba(255, 102, 0, 0.2);
}

.tool-icon {
    font-size: 24px;
    flex-shrink: 0;
}

.tool-name {
    flex: 1;
    font-size: 12px;
    font-weight: bold;
    color: #eee;
}

.tool-hotkey {
    font-size: 10px;
    color: #888;
    background: rgba(0,0,0,0.3);
    padding: 2px 6px;
    border-radius: 3px;
}
```

### Flyout Variant Item

```html
<div class="flyout-variant" data-variant-id="ARM_S" onclick="selectVariant(2, 'ARM_S')">
    <div class="variant-header">
        <span class="variant-icon">🦾S</span>
        <span class="variant-name">SMART ARM</span>
        <span class="variant-hotkey">[3]</span>
    </div>
    <div class="variant-info">
        <span class="variant-cost">$1,800</span>
        <span class="variant-level">Level 3</span>
    </div>
    <div class="variant-desc">Item filtering</div>
    <div class="variant-selected">⭐ SELECTED</div> <!-- Only show if selected -->
</div>

<!-- Locked variant example -->
<div class="flyout-variant locked" data-variant-id="ARM_M">
    <div class="variant-header">
        <span class="variant-icon">🦾M</span>
        <span class="variant-name">MEGA ARM</span>
        <span class="variant-hotkey">[4]</span>
    </div>
    <div class="variant-info">
        <span class="variant-cost">$2,000</span>
        <span class="variant-level">🔒 Level 4</span>
    </div>
    <div class="variant-desc">Unlock at Level 4</div>
</div>
```

### Flyout Variant CSS (Class-Based Display Control)

```css
.flyout-variant {
    padding: 12px;
    margin-bottom: 8px;
    background: rgba(255, 255, 255, 0.05);
    border: 2px solid transparent;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s;
    opacity: 1;
}

/* Hide variants not in active slot */
.flyout-variant.hidden {
    display: none;
}

/* Locked variants (not unlocked yet) */
.flyout-variant.locked {
    opacity: 0.4;
    cursor: not-allowed;
}

.flyout-variant.locked:hover {
    background: rgba(255, 255, 255, 0.05);
    border-color: transparent;
}

/* Selected variant (currently active) */
.flyout-variant.selected {
    border-color: var(--industrial-orange);
    background: rgba(255, 102, 0, 0.2);
}

/* Hover (unlocked only) */
.flyout-variant:not(.locked):hover {
    background: rgba(255, 255, 255, 0.1);
    border-color: var(--usa-blue);
}

/* Fade animation when switching slots */
#flyout-panel {
    transition: opacity 0.1s ease-in-out;
}

#flyout-panel.fading {
    opacity: 0;
}

.variant-header {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-bottom: 6px;
}

.variant-icon {
    font-size: 20px;
}

.variant-name {
    flex: 1;
    font-weight: bold;
    color: #eee;
    font-size: 13px;
}

.variant-hotkey {
    font-size: 11px;
    color: #888;
    background: rgba(0,0,0,0.3);
    padding: 2px 6px;
    border-radius: 3px;
}

.variant-info {
    display: flex;
    gap: 10px;
    margin-bottom: 4px;
    font-size: 11px;
}

.variant-cost {
    color: var(--gold);
    font-weight: bold;
}

.variant-level {
    color: #888;
}

.variant-desc {
    font-size: 10px;
    color: #aaa;
    font-style: italic;
}

.variant-selected {
    margin-top: 6px;
    color: var(--industrial-orange);
    font-size: 11px;
    font-weight: bold;
}
```

---

## JavaScript Functions

### Open Flyout

```javascript
function openFlyout(slotNumber) {
    const slot = BUILDING_SLOTS.find(s => s.slot === slotNumber);
    if (!slot) return;

    openFlyout = slotNumber;

    // Position flyout
    const slotElement = document.getElementById(`tool-slot-${slotNumber}`);
    const rect = slotElement.getBoundingClientRect();
    const flyout = document.getElementById('flyout-menu');

    flyout.style.left = (rect.right + 10) + 'px';
    flyout.style.top = rect.top + 'px';
    flyout.style.display = 'block';

    // Populate flyout
    const header = document.getElementById('flyout-header');
    header.textContent = `${slot.name} VARIANTS`;

    const content = document.getElementById('flyout-content');
    content.innerHTML = '';

    slot.variants.forEach((variantId, index) => {
        const variant = ALL_BUILDINGS[variantId];
        const isLocked = variant.reqLevel > level;
        const isSelected = selectedVariants[slotNumber] === variantId;

        const variantEl = document.createElement('div');
        variantEl.className = 'flyout-variant' + (isLocked ? ' locked' : '');
        variantEl.dataset.variantId = variantId;

        if (!isLocked) {
            variantEl.onclick = () => selectVariant(slotNumber, variantId);
        }

        variantEl.innerHTML = `
            <div class="variant-header">
                <span class="variant-icon">${variant.icon}</span>
                <span class="variant-name">${variant.name}</span>
                <span class="variant-hotkey">Alt+${index + 1}</span>
            </div>
            <div class="variant-info">
                <span class="variant-cost">$${variant.cost.toLocaleString()}</span>
                <span class="variant-level">${isLocked ? '🔒 ' : ''}Level ${variant.reqLevel}</span>
            </div>
            <div class="variant-desc">${variant.desc || ''}</div>
            ${isSelected ? '<div class="variant-selected">⭐ SELECTED</div>' : ''}
        `;

        content.appendChild(variantEl);
    });
}
```

### Switch to Slot (Pure Class Manipulation)

```javascript
function switchToSlot(slotNumber) {
    activeSlot = slotNumber;
    const flyout = document.getElementById('flyout-panel');

    // Fade out animation
    flyout.classList.add('fading');

    // Wait for fade, then update
    setTimeout(() => {
        updateFlyoutDisplay(slotNumber);
        flyout.classList.remove('fading');  // Fade in
    }, 100);

    updateToolbarDisplay();
}

function updateFlyoutDisplay(slotNumber) {
    // Update header
    const slotName = document.querySelector(`[data-slot="${slotNumber}"]`).dataset.type;
    document.getElementById('flyout-header').textContent = `${slotName} VARIANTS`;

    // Show/hide variants based on slot
    document.querySelectorAll('.flyout-variant').forEach(variant => {
        if (parseInt(variant.dataset.slot) === slotNumber) {
            variant.classList.remove('hidden');
        } else {
            variant.classList.add('hidden');
        }
    });

    // Update lock status based on current level
    updateVariantLockStatus();

    // Update selection indicators
    updateVariantSelection();
}

function updateVariantLockStatus() {
    document.querySelectorAll('.flyout-variant').forEach(variant => {
        const buildingId = variant.dataset.building;
        const building = ALL_BUILDINGS[buildingId];

        if (building.reqLevel > level) {
            variant.classList.add('locked');
        } else {
            variant.classList.remove('locked');
        }
    });
}

function updateVariantSelection() {
    document.querySelectorAll('.flyout-variant').forEach(variant => {
        const slotNum = parseInt(variant.dataset.slot);
        const buildingId = variant.dataset.building;

        if (selectedVariants[slotNum] === buildingId) {
            variant.classList.add('selected');
        } else {
            variant.classList.remove('selected');
        }
    });
}
```

### Select Variant

```javascript
function selectVariant(slotNumber, buildingId) {
    const building = ALL_BUILDINGS[buildingId];

    // Check if unlocked
    if (building.reqLevel > level) {
        showNotification(`UNLOCK AT LEVEL ${building.reqLevel}`);
        return;
    }

    // Update selection
    selectedVariants[slotNumber] = buildingId;
    selectedTool = buildingId;

    // Update displays
    updateToolbarDisplay();
    updateVariantSelection();

    // Save to localStorage
    saveToolbarSelection();

    // Show feedback
    showNotification(`${building.name} SELECTED`);
}
```

### Cycle Variant (Shift+Number)

```javascript
function cycleVariant(slotNumber) {
    const slot = BUILDING_SLOTS.find(s => s.slot === slotNumber);
    if (!slot) return;

    // Get unlocked variants only
    const unlockedVariants = slot.variants.filter(vId => {
        const variant = ALL_BUILDINGS[vId];
        return variant.reqLevel <= level;
    });

    if (unlockedVariants.length === 0) return;

    // Find current index
    const currentVariant = selectedVariants[slotNumber];
    const currentIndex = unlockedVariants.indexOf(currentVariant);

    // Cycle to next
    const nextIndex = (currentIndex + 1) % unlockedVariants.length;
    const nextVariant = unlockedVariants[nextIndex];

    // Select it
    selectVariant(slotNumber, nextVariant);

    // Show notification
    const variant = ALL_BUILDINGS[nextVariant];
    showNotification(`${variant.name} SELECTED`);
}
```

### Update Toolbar Display

```javascript
function updateToolbarDisplay() {
    document.querySelectorAll('.tool-slot').forEach(slotEl => {
        const slotNum = parseInt(slotEl.dataset.slot);
        const selectedBuildingId = selectedVariants[slotNum];
        const building = ALL_BUILDINGS[selectedBuildingId];

        // Update icon and name
        slotEl.querySelector('.tool-icon').textContent = building.icon;
        slotEl.querySelector('.tool-name').textContent = building.name;

        // Highlight active slot
        if (slotNum === activeSlot) {
            slotEl.classList.add('active');
        } else {
            slotEl.classList.remove('active');
        }
    });
}
```

### Event Listeners Setup (Declarative)

```javascript
function initToolbar() {
    // Tool slot clicks
    document.querySelectorAll('.tool-slot').forEach(slotEl => {
        slotEl.addEventListener('click', () => {
            const slotNum = parseInt(slotEl.dataset.slot);
            switchToSlot(slotNum);
        });
    });

    // Variant clicks
    document.querySelectorAll('.flyout-variant').forEach(variantEl => {
        variantEl.addEventListener('click', () => {
            if (variantEl.classList.contains('locked')) return;

            const slotNum = parseInt(variantEl.dataset.slot);
            const buildingId = variantEl.dataset.building;
            selectVariant(slotNum, buildingId);
        });
    });

    // Initialize display
    switchToSlot(1);  // Start with BELT slot
    updateVariantLockStatus();
}

// Call on page load
document.addEventListener('DOMContentLoaded', initToolbar);
```

### Keyboard Handling

```javascript
function handleKey(e) {
    // Existing keys (I, V, R, etc.) ...

    // Number keys 1-7: Switch slot (updates flyout)
    if (!e.shiftKey && !e.altKey && e.key >= '1' && e.key <= '7') {
        const slotNumber = parseInt(e.key);
        switchToSlot(slotNumber);  // Updates flyout to show this slot's variants
        return;
    }

    // Shift+Number: Quick cycle variant
    if (e.shiftKey && !e.altKey && e.key >= '1' && e.key <= '7') {
        const slotNumber = parseInt(e.key);
        cycleVariant(slotNumber);
        return;
    }

    // Alt+Number: Select variant from current flyout (Alt = Alternative)
    if (e.altKey && e.key >= '1' && e.key <= '4') {
        e.preventDefault();  // Prevent browser Alt shortcuts

        const variantIndex = parseInt(e.key) - 1;
        const currentSlot = BUILDING_SLOTS.find(s => s.slot === activeSlot);

        if (currentSlot && currentSlot.variants[variantIndex]) {
            selectVariant(activeSlot, currentSlot.variants[variantIndex]);
        } else {
            showNotification("VARIANT NOT AVAILABLE");
        }
        return;
    }
}
```

---

## Game Layout Changes

### Current Layout
```
┌─────────────────────────────────────┐
│           Game Canvas                │
│                                      │
│                                      │  Stats Sidebar
│                                      │  (right side)
└─────────────────────────────────────┘
            Toolbar (bottom)
```

### New Layout
```
Toolbar     ┌─────────────────────────┐
(left)      │     Game Canvas         │  Stats Sidebar
            │                         │  (right side)
            │                         │
            │                         │
            └─────────────────────────┘
```

**CSS Changes:**

```css
#game-wrapper {
    margin-left: 150px;  /* Space for left toolbar */
    /* Remove bottom padding for old toolbar */
}

#left-toolbar {
    position: fixed;
    left: 0;
    top: 0;
    height: 100vh;
    width: 150px;
    /* ... styling ... */
}

/* Remove old bottom toolbar */
#toolbar {
    display: none;  /* or remove entirely */
}
```

---

## Save/Load Integration

### Save Selected Variants

```javascript
function saveGameState() {
    const state = {
        // ... existing save data ...
        selectedVariants: selectedVariants,  // NEW: Save variant selections
    };
    localStorage.setItem('gameState', JSON.stringify(state));
}
```

### Load Selected Variants

```javascript
function loadGameState() {
    const saved = localStorage.getItem('gameState');
    if (!saved) return false;

    const state = JSON.parse(saved);

    // ... existing load logic ...

    // Restore variant selections
    if (state.selectedVariants) {
        selectedVariants = state.selectedVariants;
    } else {
        // Initialize with defaults (basic variants)
        selectedVariants = {
            1: 'BELT',
            2: 'ARM',
            3: 'SRC_P',
            4: 'SRC_S',
            5: 'CNC',
            6: 'PRESS',
            7: 'BIN'
        };
    }

    updateToolbarDisplay();
    return true;
}
```

---

## Strategic Gameplay Impact

### Example Scenarios

**Scenario 1: Early Game Budget**
- Player has $5,000
- Needs 3 ARMs for production line
- Options:
  - 3× Basic ARM ($800) = $2,400 ✅
  - 1× MEGA ARM ($2,000) + 2× Basic ARM = $3,600
  - 3× MEGA ARM = Can't afford ❌

**Choice matters!** Budget optimization is now a skill.

---

**Scenario 2: Specialized Routing**
- Main production line: MEGA ARM (4-way delivery)
- Material sorting: SMART ARM (item filters)
- Simple transfers: Basic ARM (cheap)

**Strategic tool selection** creates optimized factories.

---

**Scenario 3: Speedrun Optimization**
- Identify bottlenecks
- Use FAST variants only where speed matters
- Save money on non-critical paths
- Min-max for peak efficiency

**Skill ceiling increases** - knowing when to upgrade is mastery.

---

## Implementation Checklist

### Phase 1: Left Toolbar Structure
- [ ] Create left sidebar HTML/CSS
- [ ] Remove bottom toolbar
- [ ] Adjust game canvas margin
- [ ] Style tool slots with icons
- [ ] Test responsive layout

### Phase 2: Flyout System
- [ ] Create flyout container HTML/CSS
- [ ] Implement `openFlyout(slotNumber)` function
- [ ] Implement `closeFlyout()` function
- [ ] Position flyout dynamically next to slot
- [ ] Populate flyout with variant list
- [ ] Show lock status for unavailable variants

### Phase 3: Variant Selection
- [ ] Implement `selectVariant(slot, variantId)` function
- [ ] Update `selectedVariants` state
- [ ] Update toolbar display with selected variant
- [ ] Highlight selected variant in flyout
- [ ] Update `selectedTool` for placement

### Phase 4: Keyboard Controls
- [ ] Number keys 1-7 open flyouts
- [ ] Number keys 1-4 select variants in flyout
- [ ] Shift+Number cycles variants
- [ ] ESC closes flyout
- [ ] Click outside closes flyout

### Phase 5: Save/Load
- [ ] Save `selectedVariants` to localStorage
- [ ] Restore selections on game load
- [ ] Initialize with defaults for new games

### Phase 6: Polish
- [ ] Hover effects on slots and variants
- [ ] Transition animations for flyout
- [ ] Cost comparison ("$700 cheaper than MEGA")
- [ ] Show variant benefits in flyout
- [ ] Visual feedback when cycling variants
- [ ] Notification when selecting locked variant

### Phase 7: Testing
- [ ] Test all keyboard shortcuts
- [ ] Test flyout positioning at different screen sizes
- [ ] Test save/load of selections
- [ ] Test locked variant handling
- [ ] Test variant cycling (Shift+Number)
- [ ] Test clicking outside to close

---

## Future Enhancements

### 1. Quick Compare Mode
Hold `Alt` while hovering over variant → Show stats comparison

```
SMART ARM vs BASIC ARM
─────────────────────
Cost:   $1,800 (+$1,000)
Speed:  100% (same)
Filter: ✅ YES (vs ❌ NO)
```

### 2. Variant Recommendations
Show which variant is "best" for current situation:

```
💡 RECOMMENDED: MEGA ARM
   Reason: You have $10K+ available
```

### 3. Cost-to-Benefit Indicator
Show efficiency rating per variant:

```
ARM        ⭐⭐⭐⭐⭐  Best value
FAST ARM   ⭐⭐⭐⭐   Good speed
SMART ARM  ⭐⭐⭐    Specialized
MEGA ARM   ⭐⭐     Premium cost
```

### 4. Recently Used
Track most-used variants, show at top of flyout:

```
RECENTLY USED
─────────────
SMART ARM    ⭐
BASIC ARM
```

---

## Why This Design Works

### Strategic Depth
- ✅ Tool choice matters (not just "use latest")
- ✅ Budget optimization becomes a skill
- ✅ Factory design has more variety
- ✅ Encourages experimentation

### Discoverability
- ✅ Flyout shows all options at once
- ✅ Lock indicators teach progression
- ✅ Cost comparison helps decision-making
- ✅ Hotkeys visible in flyout

### Usability
- ✅ Fast keyboard shortcuts (Shift+Number)
- ✅ Visual feedback (selected variant shows on toolbar)
- ✅ Undo-friendly (easy to switch back)
- ✅ Saves selections across sessions

### Visual Balance
- ✅ Left toolbar mirrors right stats sidebar
- ✅ More vertical space for game grid
- ✅ Cleaner, more professional layout
- ✅ Symmetrical composition

---

## Conclusion

**This redesign:**
1. **Adds strategic depth** - tool selection matters
2. **Improves layout** - symmetrical, more space
3. **Maintains simplicity** - intuitive controls
4. **Enables mastery** - skill ceiling increases

**No level tabs needed** - variants handle progression naturally.

**Ready to implement** - clear spec, straightforward execution.
