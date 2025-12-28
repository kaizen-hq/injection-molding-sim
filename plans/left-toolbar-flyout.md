# Left Sidebar Toolbar with Flyout Selection

## Design Vision

**Move toolbar from bottom to left sidebar with flyout menus for variant selection.**

### Core Principles

1. **Slot = Building Type** (not variant)
   - Slot 2 is always "ARM" (shows all arm variants)
   - Slot 5 is always "CNC" (shows all CNC variants)
   - etc.

2. **Click to Open Flyout**
   - Click slot → Flyout shows all variants
   - Press number → Select variant, close flyout

3. **Shift+Number Cycles Variants**
   - Shift+2 → Next ARM variant (no flyout)
   - Quick switching without menu

4. **Currently Selected Shows on Toolbar**
   - Main button displays active variant icon/name
   - Visual feedback of current selection

5. **Level/Lock Status in Flyout**
   - Locked variants grayed out
   - Show "Unlock at Level X"

---

## Layout Mockup

### Left Sidebar Structure

```
┌──────────────────┐
│                  │
│  TOOLS           │  ← Header
│  ──────────      │
│                  │
│  [🚗] BELT    1  │  ← Click or press 1
│  [🦾S] ARM    2  │  ← Currently: SMART ARM
│  [🔴M] PLASTIC 3 │  ← Currently: MEGA PLASTIC
│  [⬜] STEEL    4  │
│  [⚙️] CNC      5  │
│  [🏭] PRESS    6  │
│  [💲] SELL     7  │
│                  │
└──────────────────┘
    ~150px wide
```

### Flyout When Clicking ARM (Slot 2)

```
┌──────────────────┐         ┌─────────────────────┐
│  [🚗] BELT    1  │         │ ARM VARIANTS        │
│  [🦾S] ARM*   2  │────────→│                     │
│  [🔴M] PLASTIC 3 │         │ [1] 🦾 ARM     $800 │
│  [⬜] STEEL    4  │         │     Level 1         │
│  [⚙️] CNC      5  │         │                     │
│  [🏭] PRESS    6  │         │ [2] 🦾+ FAST $1,500│
│  [💲] SELL     7  │         │     Level 2         │
│                  │         │                     │
└──────────────────┘         │ [3] 🦾S SMART $1,800│ ⭐
                             │     Level 3         │
                             │                     │
                             │ [4] 🦾M MEGA $2,000 │
                             │     Level 4 🔒      │
                             │                     │
                             └─────────────────────┘
                                      ↑
                              Press 1-4 to select
                              ⭐ = Currently selected
                              🔒 = Locked
```

---

## Keyboard Controls

### Primary Controls

| Key | Action |
|-----|--------|
| `1-7` | Select building type slot |
| `1-4` (in flyout) | Select variant from flyout |
| `Shift+1-7` | Cycle to next variant for that slot |
| `R` | Rotate building |
| `Esc` | Close flyout |

### Example Workflows

**Workflow 1: Select SMART ARM (with flyout)**
1. Press `2` → ARM flyout opens
2. Press `3` → SMART ARM selected, flyout closes
3. Now placing SMART ARM

**Workflow 2: Cycle through ARMs (no flyout)**
1. Press `Shift+2` → Next ARM variant
2. Press `Shift+2` again → Next variant
3. Cycles: ARM → FAST → SMART → MEGA → ARM...

**Workflow 3: Quick switch**
1. Currently: SMART ARM selected
2. Press `2` → Flyout opens (SMART ARM highlighted)
3. Press `1` → Basic ARM selected
4. Saves $1,000 for this placement

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

### Left Sidebar HTML

```html
<div id="left-toolbar" style="position: fixed; left: 0; top: 0; height: 100vh; width: 150px; background: var(--panel-bg); border-right: 4px solid var(--usa-blue); display: flex; flex-direction: column; padding: 20px 10px; gap: 10px; z-index: 100;">

    <div style="text-align: center; color: var(--gold); font-weight: bold; margin-bottom: 10px; font-size: 14px;">
        TOOLS
    </div>

    <!-- Tool Slots -->
    <div id="tool-slot-1" class="tool-slot" data-slot="1" onclick="openFlyout(1)">
        <div class="tool-icon">🚗</div>
        <div class="tool-name">BELT</div>
        <div class="tool-hotkey">1</div>
    </div>

    <div id="tool-slot-2" class="tool-slot" data-slot="2" onclick="openFlyout(2)">
        <div class="tool-icon">🦾</div>
        <div class="tool-name">ARM</div>
        <div class="tool-hotkey">2</div>
    </div>

    <!-- ... slots 3-7 ... -->

</div>

<!-- Flyout Container (positioned absolutely, appears next to toolbar) -->
<div id="flyout-menu" style="display: none; position: fixed; left: 160px; top: 100px; background: var(--panel-bg); border: 3px solid var(--usa-red); border-radius: 8px; padding: 15px; z-index: 200; min-width: 250px;">
    <div id="flyout-header" style="color: var(--gold); font-weight: bold; margin-bottom: 10px; border-bottom: 2px solid var(--usa-blue); padding-bottom: 5px;">
        ARM VARIANTS
    </div>
    <div id="flyout-content">
        <!-- Variants populated dynamically -->
    </div>
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

### Flyout Variant CSS

```css
.flyout-variant {
    padding: 12px;
    margin-bottom: 8px;
    background: rgba(255, 255, 255, 0.05);
    border: 2px solid transparent;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s;
}

.flyout-variant:hover {
    background: rgba(255, 255, 255, 0.1);
    border-color: var(--usa-blue);
}

.flyout-variant.locked {
    opacity: 0.4;
    cursor: not-allowed;
}

.flyout-variant.locked:hover {
    background: rgba(255, 255, 255, 0.05);
    border-color: transparent;
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
                <span class="variant-hotkey">[${index + 1}]</span>
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

### Close Flyout

```javascript
function closeFlyout() {
    openFlyout = null;
    document.getElementById('flyout-menu').style.display = 'none';
}

// Close on Esc key
window.addEventListener('keydown', (e) => {
    if (e.key === 'Escape' && openFlyout !== null) {
        closeFlyout();
    }
});

// Close when clicking outside
document.addEventListener('click', (e) => {
    if (openFlyout !== null) {
        const flyout = document.getElementById('flyout-menu');
        const toolbar = document.getElementById('left-toolbar');

        if (!flyout.contains(e.target) && !toolbar.contains(e.target)) {
            closeFlyout();
        }
    }
});
```

### Select Variant

```javascript
function selectVariant(slotNumber, variantId) {
    const variant = ALL_BUILDINGS[variantId];

    // Check if unlocked
    if (variant.reqLevel > level) {
        showNotification(`UNLOCK AT LEVEL ${variant.reqLevel}`);
        return;
    }

    // Update selection
    selectedVariants[slotNumber] = variantId;
    selectedTool = variantId;

    // Update toolbar display
    updateToolbarDisplay();

    // Close flyout
    closeFlyout();

    // Save to localStorage
    saveToolbarSelection();
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
    BUILDING_SLOTS.forEach(slot => {
        const slotElement = document.getElementById(`tool-slot-${slot.slot}`);
        const selectedVariantId = selectedVariants[slot.slot];
        const variant = ALL_BUILDINGS[selectedVariantId];

        // Update icon
        slotElement.querySelector('.tool-icon').textContent = variant.icon;

        // Update name (show variant name, not type name)
        slotElement.querySelector('.tool-name').textContent = variant.name;

        // Highlight if selected for placement
        if (selectedTool === selectedVariantId) {
            slotElement.classList.add('selected');
        } else {
            slotElement.classList.remove('selected');
        }
    });
}
```

### Keyboard Handling

```javascript
function handleKey(e) {
    // Existing keys (I, V, R, etc.) ...

    // Number keys 1-7 (without Shift)
    if (!e.shiftKey && e.key >= '1' && e.key <= '7') {
        const slotNumber = parseInt(e.key);

        if (openFlyout === slotNumber) {
            // Flyout already open for this slot, close it
            closeFlyout();
        } else if (openFlyout !== null) {
            // Different flyout open, switch to this one
            openFlyout(slotNumber);
        } else {
            // No flyout open, open this one
            openFlyout(slotNumber);
        }
        return;
    }

    // Shift+Number: Cycle variant
    if (e.shiftKey && e.key >= '1' && e.key <= '7') {
        const slotNumber = parseInt(e.key);
        cycleVariant(slotNumber);
        return;
    }

    // Number keys 1-4 inside flyout (select variant)
    if (openFlyout !== null && e.key >= '1' && e.key <= '4') {
        const variantIndex = parseInt(e.key) - 1;
        const slot = BUILDING_SLOTS.find(s => s.slot === openFlyout);

        if (slot && slot.variants[variantIndex]) {
            selectVariant(openFlyout, slot.variants[variantIndex]);
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
