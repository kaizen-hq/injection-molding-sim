# In-Place Building Rotation

## Problem Statement

Currently, rotating a building requires:
1. Right-click to destroy (50% refund)
2. Rebuild in correct orientation (100% cost)
3. **Net cost: 50% of building price**

This is expensive and punishes experimentation. Players need a cheaper way to fix rotation mistakes.

---

## Proposed Solution

**Allow in-place rotation for 10% of building cost per 90° rotation**

### Benefits
- **Cheaper mistake correction:** 10% vs 50% cost
- **Rewards planning:** Getting it right first time costs $0
- **Quality of life:** Faster iteration, less friction
- **Strategic depth:** Cost-benefit analysis (is fixing this worth $80?)

### Trade-offs
- Still has a cost (not free)
- 4 rotations = 40% (cheaper than destroy+rebuild at 50%)
- Encourages thinking before placing

---

## User Experience

### How to Rotate

**Option A: Inspector Button (Recommended)**
1. Click building to open inspector
2. See "↻ ROTATE ($X)" button
3. Click to rotate 90° clockwise
4. Building rotates, money deducted
5. Can rotate multiple times

**Option B: Keyboard Shortcut**
1. Click building (inspector opens)
2. Press `R` key
3. Building rotates 90° clockwise
4. Money deducted

**Option C: Middle-Click**
1. Middle-click building
2. Rotates 90° clockwise
3. Money deducted

**Recommendation: Option A (Inspector Button)**
- Most discoverable
- Shows cost before rotating
- Consistent with existing UI patterns
- Can add confirmation if needed

---

## Technical Design

### Data Changes

No new state needed - buildings already have `rot` property (0-3).

### Functions

```javascript
function rotateBuilding(building) {
    const cost = Math.floor(building.type.cost * 0.10);

    if (money < cost) {
        showNotification("INSUFFICIENT FUNDS TO ROTATE");
        return false;
    }

    // Deduct cost
    money -= cost;

    // Rotate 90° clockwise
    building.rot = (building.rot + 1) % 4;

    // Update UI
    updateUI();
    updateInspectorUI();

    showNotification(`ROTATED ${building.type.name} (-$${cost})`);
    return true;
}
```

### UI Changes

**Inspector Panel:**

Add rotate button after inventory display:

```html
<!-- Existing inspector content -->
<div id="insp-inventory">...</div>

<!-- New rotation button -->
<div style="margin-top: 10px; padding-top: 10px; border-top: 1px solid #555;">
    <button
        id="btn-rotate"
        onclick="rotateInspectedBuilding()"
        style="width: 100%; padding: 8px; background: var(--usa-blue); color: #fff; border: none; border-radius: 4px; cursor: pointer; font-weight: bold;">
        ↻ ROTATE ($<span id="rotate-cost">0</span>)
    </button>
</div>
```

**Update inspector when opened:**
```javascript
function openInspector(b) {
    // ... existing code ...

    // Show rotation cost
    const rotateCost = Math.floor(b.type.cost * 0.10);
    document.getElementById('rotate-cost').textContent = rotateCost;
}

function rotateInspectedBuilding() {
    if (inspectedBuilding) {
        rotateBuilding(inspectedBuilding);
    }
}
```

---

## Edge Cases & Behaviors

### 1. Inventory Handling

**Recommendation: Preserve inventory when rotating**

- ARMs: Keep held items and filters
- Machines: Keep input inventory and output buffers
- Sources: Keep production timers
- Rational: Rotation is a minor adjustment, not a rebuild

**Alternative: Clear inventory (more conservative)**
- Prevents weird states
- Simpler to implement
- More punishing (not recommended)

### 2. Items on Belts

**Belt rotation affects item direction:**

```javascript
function rotateBuilding(building) {
    // ... existing rotation code ...

    // If rotating a belt, update items on it
    if (building.type.id.includes('belt')) {
        items.forEach(item => {
            if (item.x === building.x && item.y === building.y) {
                // Rotate item direction to match new belt direction
                item.dir = building.rot;
            }
        });
    }
}
```

Items on the belt should rotate with it to maintain flow direction.

### 3. ARM Configuration

**MEGA ARM has delivery direction separate from rotation:**

- `building.rot` = where it picks up from (backwards)
- `building.deliverDir` = where it delivers to (configurable)

**Behavior when rotating ARM:**
- Rotate both `rot` and `deliverDir` together
- Maintains relative pickup/delivery relationship
- User can reconfigure delivery after rotating if needed

```javascript
if (building.type.canDirectDeliver) {
    const oldRot = building.rot;
    building.rot = (building.rot + 1) % 4;

    // Rotate delivery direction by same amount
    const rotDelta = (building.rot - oldRot + 4) % 4;
    building.deliverDir = (building.deliverDir + rotDelta) % 4;
}
```

### 4. Insufficient Funds

Show notification, don't rotate:
```
"INSUFFICIENT FUNDS TO ROTATE"
```

### 5. Visual Feedback

**Option A: Instant rotation** (simple)
- Building immediately faces new direction
- No animation needed

**Option B: Rotation animation** (polished)
- Rotate building over 200ms
- CSS transform or canvas rotation interpolation
- More satisfying, but complex

**Recommendation: Option A for MVP, Option B for polish**

---

## Cost Analysis

### Example: Rotating an ARM ($800)

| Action | Cost | Net Spent |
|--------|------|-----------|
| Place ARM wrong direction | $800 | $800 |
| **OLD:** Destroy + Rebuild | -$400 + $800 | $1,200 |
| **NEW:** Rotate once | $80 | $880 |
| **Savings:** | | **$320** |

### Example: Rotating 4 times (full circle)

| Building | Cost | 1 Rotate | 4 Rotates | Destroy+Rebuild |
|----------|------|----------|-----------|-----------------|
| BELT | $20 | $2 | $8 | $30 (50% loss) |
| ARM | $800 | $80 | $320 | $1,200 |
| PRESS | $800 | $80 | $320 | $1,200 |
| MEGA ARM | $2,000 | $200 | $800 | $3,000 |
| CNC_M | $3,000 | $300 | $1,200 | $4,500 |

**Key insight:** Even 4 full rotations (360°) is cheaper than destroy+rebuild.

---

## User Messaging

### Tutorial / Help Text

Add to manual (press I):

```
ROTATION:
- Click a building, then press ↻ ROTATE
- Costs 10% of building price per 90° turn
- Much cheaper than destroying and rebuilding!
- TIP: Plan ahead to save money
```

### Notifications

```javascript
// Success
"ROTATED ARM (-$80)"

// Insufficient funds
"INSUFFICIENT FUNDS TO ROTATE"

// Multiple rotations
"ROTATED BELT 2x (-$4)"  // If we add multi-rotate feature
```

---

## Future Enhancements

### 1. Multi-Rotate (Shift+Click)

Hold Shift while clicking rotate to rotate 180° (costs 20%):
```javascript
function rotateBuilding(building, rotations = 1) {
    const cost = Math.floor(building.type.cost * 0.10 * rotations);
    // ...
}
```

### 2. Free Rotation Window

Allow free rotation in first 5 seconds after placement:
```javascript
class Building {
    constructor(...) {
        // ...
        this.placedAt = Date.now();
    }
}

function rotateBuilding(building) {
    const age = Date.now() - building.placedAt;
    const cost = age < 5000 ? 0 : Math.floor(building.type.cost * 0.10);
    // ...
}
```

### 3. Rotation Preview

Show ghosted preview of rotated building before committing:
- Click "Preview Rotate"
- See building in new orientation (semi-transparent overlay)
- Click "Confirm" or "Cancel"

### 4. Bulk Rotation

Select multiple buildings (drag box) and rotate all at once.

---

## Implementation Checklist

### Phase 1: Core Functionality (MVP)
- [ ] Add `rotateBuilding(building)` function
- [ ] Add inspector button with cost display
- [ ] Deduct money and update `building.rot`
- [ ] Update UI and inspector after rotation
- [ ] Show notification on rotate

### Phase 2: Edge Cases
- [ ] Handle items on belts (rotate item direction)
- [ ] Handle ARM delivery direction rotation
- [ ] Preserve inventory/timers when rotating
- [ ] Handle insufficient funds gracefully

### Phase 3: Polish
- [ ] Add to manual/help text
- [ ] Add keyboard shortcut (R key in inspector?)
- [ ] Visual feedback (highlight building on rotate?)
- [ ] Sound effect? (optional)

### Phase 4: Testing
- [ ] Test rotating each building type
- [ ] Test rotating buildings with items
- [ ] Test rotating ARMs with filters set
- [ ] Test rotating MEGA ARMs with custom delivery direction
- [ ] Test insufficient funds case
- [ ] Test rotating belts with items on them

---

## Alternative Approaches Considered

### 1. Free Rotation (0% cost)

**Pros:**
- Maximum convenience
- No punishment for experimentation

**Cons:**
- Removes strategic planning element
- Makes initial placement decisions meaningless
- No reason to think before placing

**Verdict:** Too permissive, removes game depth

### 2. Same Cost as Destroy (50%)

**Pros:**
- Consistent with existing scrap mechanic

**Cons:**
- Still expensive
- Doesn't solve the problem
- No benefit over destroy+rebuild

**Verdict:** Doesn't improve current experience enough

### 3. Flat Cost ($50 for all buildings)

**Pros:**
- Simple to understand
- Predictable

**Cons:**
- Weird balancing (same to rotate $20 belt vs $3000 CNC?)
- Either too cheap for expensive buildings or too expensive for cheap ones

**Verdict:** Percentage-based is fairer

### 4. Increasing Cost (10%, 20%, 30%, 40% for rotations 1-4)

**Pros:**
- Penalizes repeated rotation
- Encourages planning even more

**Cons:**
- Complex
- Punishing
- Hard to communicate

**Verdict:** Flat 10% is simpler and sufficient

---

## Recommendation

**Implement Phase 1 (MVP) first:**
1. Inspector button to rotate for 10% cost
2. Preserve inventory when rotating
3. Rotate items on belts to match
4. Basic notifications

**Then evaluate:**
- Do players use it?
- Is 10% the right cost?
- Is inspector button the right UX?

**Can add polish/enhancements based on feedback.**

---

## Rationale for 10% Cost

**Why not free?**
- Maintains strategic value of planning
- Preserves reward for getting it right
- Prevents mindless trial-and-error

**Why not higher?**
- Should be accessible for fixing mistakes
- Still want experimentation to be affordable
- 10% is "ouch but okay" not "might as well rebuild"

**Why 10% specifically?**
- Round number (easy to calculate)
- Meaningful but not punishing
- 4 rotations = 40% (still cheaper than 50% destroy cost)
- Sweet spot between free and expensive

**Balancing philosophy:**
Making a mistake should have a cost, but not be devastating. 10% says "try to get it right, but we won't punish you hard if you don't."
