# Hyper Arm - Extended Reach Building

## Concept

**An advanced robotic arm with extended reach - picks up from 2 squares behind, delivers 2 squares forward.**

### Core Mechanic

**Regular ARM:**
```
[Source] → [ARM] → [Target]
   -1       0        +1
```

**HYPER ARM:**
```
[Source] ← [Empty] ← [HYPER] → [Empty] → [Target]
   -2        -1        0         +1        +2
```

Reaches over one square, allowing:
- Skipping obstacles
- Longer-range transfers
- More compact layouts
- New routing possibilities

---

## Design Rationale

### Problem It Solves

**Current limitation:** ARMs require direct adjacency

```
[Plastic] [Belt] [Press] [Belt] [Bin]
           ↑              ↑
         Need ARM here and here
```

**With HYPER ARM:**
```
[Plastic] [Belt] [HYPER] [Belt] [Bin]
           ↑       ↑↑↑↑↑   ↓
    Reaches 2 back ↑  ↓ Delivers 2 forward
                   └──┘
```

Can reach over the belt to grab from Plastic and deliver over the other belt to Bin.

### Strategic Depth

**Trade-offs:**
- **Expensive** (Level 5, ~$4,000)
- **Niche use case** (only valuable when you need extended reach)
- **Requires planning** (must position correctly)
- **Slower?** (maybe 75% speed due to longer travel distance)

**When to use:**
- Tight spaces where belts don't fit
- Reaching across production lines
- Grabbing from distant sources
- Bypassing obstacles

**When NOT to use:**
- Regular ARM is adjacent and works fine
- Budget-constrained (regular ARM is $800, HYPER is $4,000)
- Speed is critical (HYPER might be slower)

---

## Visual Design

### Icon Representation

**Regular ARM:** `🦾`
**MEGA ARM:** `🦾M`
**HYPER ARM:** `🦾H` or `🦾⚡` or `🦿` (prosthetic leg = longer reach?)

**Color:** Electric blue (`#00D4FF`) to distinguish from regular ARM orange

### Visual Indicators

**Option A: Extended reach lines**
```
Canvas overlay showing:
- Dashed line 2 squares backward (pickup zone)
- Dashed line 2 squares forward (delivery zone)
- Different color than regular ARM
```

**Option B: Glowing effect**
```
HYPER ARM tile glows/pulses
Pickup target (2 back) glows when item detected
Delivery target (2 forward) glows when delivering
```

**Option C: Double-length arm animation**
```
Arm extends 2 tiles instead of 1
Longer animation path
Visual feedback of extended reach
```

**Recommendation: Combination of A + C**
- Show reach zones when placing (like regular ARM preview)
- Animate arm extending 2 tiles during pickup/delivery

---

## Technical Implementation

### Building Definition

```javascript
ARM_H: {
    id: 'arm_hyper',
    name: 'HYPER ARM',
    cost: 4000,
    color: '#00D4FF',
    icon: '🦾H',
    reqLevel: 5,
    isArm: true,
    canFilter: true,
    canDirectDeliver: false,  // Only delivers forward (2 squares)
    speed: 0.75,  // 75% speed (slower due to longer reach)
    reachDistance: 2,  // NEW PROPERTY
    desc: "Extended-Reach Robotic Arm. Picks up from 2 squares behind, delivers 2 squares forward. Slower due to longer travel distance."
}
```

### Core Logic Changes

**Current ARM pickup logic:**
```javascript
let sourceXY = getNeighbor(this.x, this.y, (this.rot + 2) % 4);  // 1 square back
```

**HYPER ARM pickup logic:**
```javascript
function getNeighborAtDistance(x, y, dir, distance) {
    const dx = [0, 1, 0, -1][dir];
    const dy = [-1, 0, 1, 0][dir];
    return { x: x + dx * distance, y: y + dy * distance };
}

// In ARM update:
const reachDistance = this.type.reachDistance || 1;
let sourceXY = getNeighborAtDistance(this.x, this.y, (this.rot + 2) % 4, reachDistance);
let targetXY = getNeighborAtDistance(this.x, this.y, this.rot, reachDistance);
```

### Animation Changes

**Regular ARM animation path:**
```javascript
// Interpolate from (armX, armY) to (targetX, targetY) over 1 tile distance
const dx = targetX - armX;
const dy = targetY - armY;
const drawX = armX + dx * this.armAnim;
const drawY = armY + dy * this.armAnim;
```

**HYPER ARM animation path (same logic, just 2x distance):**
```javascript
// Already works! Animation automatically extends to 2 tiles
// because we're calculating based on actual target position
// No changes needed - it'll just naturally animate further
```

**Rendering the arm:**
```javascript
function drawArm(building) {
    if (!building.type.isArm) return;

    const reachDistance = building.type.reachDistance || 1;

    if (building.heldItem) {
        // Draw line from building to delivery target (2 squares away)
        const target = getNeighborAtDistance(building.x, building.y, building.rot, reachDistance);
        const targetX = target.x * TILE_SIZE + TILE_SIZE / 2;
        const targetY = target.y * TILE_SIZE + TILE_SIZE / 2;
        const armX = building.x * TILE_SIZE + TILE_SIZE / 2;
        const armY = building.y * TILE_SIZE + TILE_SIZE / 2;

        // Interpolate
        const drawX = armX + (targetX - armX) * building.armAnim;
        const drawY = armY + (targetY - armY) * building.armAnim;

        // Draw extended arm line (thicker for HYPER?)
        ctx.strokeStyle = building.type.color || '#FF6B35';
        ctx.lineWidth = reachDistance * 2;  // Thicker line for longer reach
        ctx.beginPath();
        ctx.moveTo(armX, armY);
        ctx.lineTo(drawX, drawY);
        ctx.stroke();

        // Draw item at end
        drawItem(building.heldItem, drawX, drawY, TILE_SIZE * 0.6);
    }
}
```

---

## Gameplay Examples

### Example 1: Skipping Over a Belt

**Problem:** Want to move items from Source to Press, but there's a belt in the way

**Without HYPER ARM:**
```
[Source] → [Belt] → [ARM] → [Press]
                      ↓
                  Needs to grab from belt, not source
```

**With HYPER ARM:**
```
[Source] → [Belt] [HYPER] → [Empty] → [Press]
    ↑               ↑↑↑         ↓         ↓
    └───────────────┘            └────────┘
      Reaches 2 back           Delivers 2 forward
```

HYPER ARM can reach OVER the belt to grab directly from Source, and deliver directly to Press.

### Example 2: Compact Cross-Line Transfer

**Problem:** Two parallel production lines need to share resources

**Without HYPER ARM (needs 2 belts + 2 ARMs):**
```
[Line A] [Belt] [ARM] [Transfer Zone] [ARM] [Belt] [Line B]
```

**With HYPER ARM (direct transfer):**
```
[Line A] [Empty] [HYPER] [Empty] [Line B]
    ↑              ↑↑↑      ↓        ↓
    └──────────────┘        └────────┘
```

### Example 3: Reaching Into Dense Clusters

**Problem:** Dense machine cluster, hard to route ARMs

**With HYPER ARM:**
```
    [CNC]
     ↑
[HYPER] reaches 2 up to grab from CNC
     ↓
  [Empty]
     ↓
  [Press] receives delivery 2 down
```

Can reach OVER an empty square to connect non-adjacent machines.

---

## Balance Considerations

### Cost Analysis

| ARM Type | Cost | Speed | Reach | Filter | Delivery |
|----------|------|-------|-------|--------|----------|
| ARM | $800 | 100% | 1 | ❌ | Forward only |
| ARM_F | $1,500 | 150% | 1 | ✅ | Forward only |
| ARM_S | $1,800 | 100% | 1 | ✅ | Forward only |
| ARM_M | $2,000 | 100% | 1 | ✅ | 4-directional |
| **ARM_H** | **$4,000** | **75%** | **2** | **✅** | **Forward only** |

**Design philosophy:**
- Most expensive ARM (reflects advanced tech)
- Slower speed (penalty for extended reach)
- Has filter (expected at this tier)
- No directional delivery (trade-off for extended reach)

### Power Level

**Is this too powerful?**

**NO, because:**
1. **Expensive** - $4,000 is 5x cost of basic ARM
2. **Slower** - 75% speed means less throughput
3. **Niche** - Only valuable in specific layouts
4. **Level 5** - Late-game unlock
5. **Forward-only** - Can't change delivery direction like MEGA ARM

**Use case is specialized:**
- Not a replacement for regular ARMs
- Solves specific routing problems
- Enables advanced factory designs
- Rewards creative layouts

### When Regular ARM is Better

**Stick with regular ARM when:**
- Buildings are already adjacent (no need for reach)
- Budget is tight ($800 << $4,000)
- Speed matters (100% vs 75%)
- MEGA ARM directional delivery is needed

**HYPER ARM is situational, not superior in all cases.**

---

## Level 5 Integration (Deferred)

**Note:** User requested Level 5 discussion be separate plan.

**Quick notes for future:**
- HYPER ARM unlocks at Level 5
- Level 5 threshold: $2M revenue? (2x current $1M goal)
- Other Level 5 buildings: HYPER variants of other tools?
  - HYPER BELT (teleports items 2 squares?)
  - HYPER PRESS (instant molding?)
  - HYPER CNC (cuts ultra-molds for 8x output?)

**This plan assumes Level 5 exists and focuses only on HYPER ARM.**

---

## UI/UX Details

### Placement Preview

**When placing HYPER ARM:**

Show ghosted indicators:
```
[ ? ]  ← Pickup zone (2 back)
[   ]  ← Empty
[🦾H] ← HYPER ARM
[   ]  ← Empty
[ ✓ ]  ← Delivery zone (2 forward)
```

Similar to current ARM placement preview, but extended 2 tiles.

### Inspector Panel

**Same as other ARMs:**
- Show inventory (should be empty for arms)
- Show filter dropdown (HYPER ARM has filtering)
- Show rotation button (if in-place rotation is implemented)

**Additional info:**
```
HYPER ARM
───────────────
REACH: 2 TILES
SPEED: 75%
FILTER: [Dropdown]
```

### Toolbar Display

**Slot 2 (ARM slot) progression:**
```
Level 1: 🦾 ARM
Level 2: 🦾F FAST ARM
Level 3: 🦾S SMART ARM
Level 4: 🦾M MEGA ARM
Level 5: 🦾H HYPER ARM
```

Only shows HYPER ARM when Level 5 is unlocked.

---

## Edge Cases & Behaviors

### 1. Pickup Zone Out of Bounds

**If HYPER ARM at (1, 5) facing left:**
- Pickup zone would be at (-1, 5) - OFF GRID!

**Behavior:**
```javascript
function attemptPickup(sourceXY, targetXY) {
    // Check if source is valid position
    if (!isValid(sourceXY)) return null;  // Can't pick up from off-grid

    // ... rest of pickup logic
}
```

ARM simply idles if pickup zone is invalid. No errors, just doesn't work.

### 2. Delivery Zone Out of Bounds

**If HYPER ARM at (GRID_W - 1, 5) facing right:**
- Delivery zone would be at (GRID_W + 1, 5) - OFF GRID!

**Behavior:**
```javascript
function attemptDrop(targetXY, item) {
    if (!isValid(targetXY)) return false;  // Can't deliver off-grid

    // ... rest of delivery logic
}
```

ARM holds item forever if delivery zone is invalid. Player must fix layout.

### 3. Obstacle in Intermediate Square

**Setup:**
```
[Source] [Belt] [HYPER] [Belt] [Target]
```

HYPER ARM reaches OVER the intermediate belt.

**Question: Does the intermediate square matter?**

**Answer: NO - Arm reaches OVER it**

```javascript
// Pickup/delivery logic only checks source and target
// Doesn't care what's in between
// Arm extends over obstacles
```

This is the whole point! Arm can skip over occupied squares.

### 4. Multiple HYPER ARMs Targeting Same Square

**Setup:**
```
    [HYPER A]
       ↓↓
    [Target]
       ↑↑
    [HYPER B]
```

Both ARMs trying to deliver to same square 2 tiles away.

**Behavior: Same as regular ARMs**

```javascript
// attemptDrop already handles this
// Only one item can be dropped per tick
// Other ARM waits until square is clear
```

No new logic needed - existing collision handling works.

### 5. Filter Behavior

**HYPER ARM with filter set to PLASTIC:**

```javascript
// Pickup logic
let item = this.attemptPickup(sourceXY, targetXY);
if (item) {
    // Filter check (existing logic)
    if (this.filter && item.type !== this.filter) {
        return;  // Don't pick up filtered items
    }
    this.heldItem = item;
}
```

Filters work exactly like SMART ARM - no changes needed.

---

## Implementation Checklist

### Phase 1: Core Mechanic
- [ ] Add `ARM_H` building definition with `reachDistance: 2`
- [ ] Add `getNeighborAtDistance(x, y, dir, distance)` helper function
- [ ] Update ARM pickup logic to use `reachDistance`
- [ ] Update ARM delivery logic to use `reachDistance`
- [ ] Test basic pickup/delivery at 2 tiles distance

### Phase 2: Visual Feedback
- [ ] Update placement preview to show 2-tile reach
- [ ] Update arm rendering to draw extended line
- [ ] Make arm line thicker for HYPER ARM (visual clarity)
- [ ] Add color differentiation (electric blue)

### Phase 3: Edge Cases
- [ ] Test pickup/delivery out-of-bounds (grid edges)
- [ ] Test reaching over obstacles (belts, buildings)
- [ ] Test filter behavior with HYPER ARM
- [ ] Test multiple HYPER ARMs targeting same square

### Phase 4: Balance & Polish
- [ ] Playtest cost ($4,000) - too cheap/expensive?
- [ ] Playtest speed (75%) - too slow/fast?
- [ ] Add to manual/help text
- [ ] Add building description tooltip

### Phase 5: Level 5 Integration
- [ ] (Deferred to separate plan)
- [ ] Add Level 5 unlock logic
- [ ] Add HYPER ARM to toolbar at Level 5
- [ ] Set Level 5 requirements

---

## Alternative Designs Considered

### 1. Symmetric Reach (2 back, 2 forward)

**Proposed in this plan** ✅

**Pros:**
- Intuitive (reaches 2 in both directions)
- Balanced (equal pickup/delivery range)
- Simpler to understand

**Cons:**
- None significant

**Verdict: RECOMMENDED**

---

### 2. Asymmetric Reach (1 back, 3 forward)

**"Launcher" variant - throws items far**

**Pros:**
- Different flavor than regular ARM
- Interesting for skip-routing

**Cons:**
- Weird balance (why would pickup be shorter?)
- Less intuitive
- Harder to visualize

**Verdict: Not recommended**

---

### 3. Variable Reach (Choose 1 or 2 tiles)

**Player can configure reach distance**

**Pros:**
- Flexibility
- Can adapt to different layouts

**Cons:**
- Complexity creep
- Hard to configure in UI
- Not clear when to use 1 vs 2

**Verdict: Too complex**

---

### 4. Same Speed as Regular ARM (100%)

**No speed penalty**

**Pros:**
- More powerful
- No throughput loss

**Cons:**
- Too good - always better than regular ARM
- Removes trade-off
- Makes regular ARM obsolete at Level 5

**Verdict: Not balanced**

---

### 5. 50% Speed (Very Slow)

**Heavy speed penalty**

**Pros:**
- Strong trade-off
- Forces strategic choice

**Cons:**
- Too slow - might be unusable
- Frustrating to watch

**Verdict: Too punishing**

**Chosen 75% is sweet spot** - noticeable but not crippling

---

## Thematic Justification

**Why does an arm that reaches further exist?**

**In-universe explanation:**
- Advanced hydraulics and telescoping joints
- Precision servos allow extended reach while maintaining accuracy
- Trade-off: Longer travel distance = slower cycle time
- Represents cutting-edge American manufacturing robotics
- Level 5 = "Future Tech" tier

**Visual theme:**
- Electric blue color (high-tech)
- Glowing effect (advanced systems)
- Sleeker icon (modernized design)

---

## Future Extensions

### 1. HYPER Variants of Other Buildings

**HYPER BELT:**
- Teleports items 2 squares?
- Skips intermediate tile?

**HYPER PRESS:**
- Accepts inputs from 2 squares away?
- No ARM needed to feed it?

**HYPER CNC:**
- Cuts ULTRA MOLDS (8x output)?
- Reaches 2 tiles for input steel?

### 2. Reach Distance as Configurable Property

**If multiple HYPER variants exist:**

```javascript
// Generic "extended reach" system
const REACH_MULTIPLIERS = {
    NORMAL: 1,
    HYPER: 2,
    ULTRA: 3,  // Future tier?
};

building.reachMultiplier = REACH_MULTIPLIERS.HYPER;
```

Allows easy creation of new reach-based buildings.

### 3. Diagonal Reach

**HYPER ARM can reach diagonally?**

```
    [Source]
       ↖
    [HYPER]
       ↘
    [Target]
```

Enables even more creative routing, but might be too complex.

---

## Success Metrics

**How do we know HYPER ARM is well-designed?**

### Playtesting Questions

1. **Is it used?**
   - Do players unlock and place HYPER ARMs?
   - Or do they stick with cheaper alternatives?

2. **Is it situational?**
   - Used in specific layouts, not everywhere?
   - Or does it replace all regular ARMs? (bad)

3. **Is it satisfying?**
   - Does the extended reach feel cool?
   - Do players enjoy the routing possibilities?

4. **Is it balanced?**
   - Does $4,000 feel fair?
   - Is 75% speed acceptable?

### Ideal Outcome

**Players say:**
- "HYPER ARM is perfect for this tight space!"
- "I can finally skip over that belt cluster"
- "Expensive but worth it for this layout"

**NOT:**
- "HYPER ARM is useless, too expensive"
- "HYPER ARM is OP, I only use these now"
- "HYPER ARM is too slow, never worth it"

---

## Conclusion

**HYPER ARM is a specialized, high-tier building that rewards creative factory layouts.**

**Core identity:**
- Extended reach (2 tiles)
- Expensive premium ($4,000)
- Speed trade-off (75%)
- Enables advanced routing strategies

**Fills a niche:**
- Not a direct upgrade to regular ARM
- Solves specific layout problems
- Late-game optimization tool

**Implementation is straightforward:**
- Minimal code changes (add `reachDistance` property)
- Visual updates (extended arm rendering)
- Balance testing (cost and speed)

**Ready to implement once Level 5 system exists.**
