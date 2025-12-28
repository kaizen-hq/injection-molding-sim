# Operating Costs (Electricity/Power System)

## The Question

**Should buildings have ongoing operating costs (electricity/power) instead of just one-time purchase costs?**

---

## Current Economic Model

### How It Works Now

**One-time costs:**
- Pay $800 once to place ARM
- ARM works forever for free
- Revenue from selling parts
- Only costs: initial building placement

**Player strategy:**
- Build as many buildings as budget allows
- More buildings = more production = more money
- No penalty for idle/inefficient buildings
- Economic growth is exponential once set up

**Victory condition:**
- Reach $1M lifetime revenue
- Pure accumulation game

---

## Proposed: Operating Costs

### Basic Concept

**Buildings consume electricity/power:**
- Pay $/second or $/minute to keep factory running
- Different buildings have different power costs
- Creates ongoing drain on money

**Example costs:**
```javascript
BELT: $1/min
ARM: $5/min
PRESS: $10/min
MEGA PRESS: $25/min
```

**Net profit calculation:**
```
Revenue = Parts sold × $45
Costs = Building power × minutes
Profit = Revenue - Costs
```

---

## Approach 1: Simple Drain (Flat Cost Per Building)

### How It Works

**Each building costs power per minute:**
```javascript
const POWER_COSTS = {
    BELT: 1,      // $1/min
    ARM: 5,       // $5/min
    PRESS: 10,    // $10/min
    CNC: 8,       // $8/min
    BIN: 2,       // $2/min
};

// Every second
money -= calculateTotalPowerCost() / 60;
```

**UI shows:**
- Current power usage: "$120/min"
- Net profit: "+$80/min" (or "-$40/min" if losing money)
- Power budget bar

### Pros
✅ Simple to understand
✅ Rewards efficiency (idle buildings = wasted money)
✅ Creates tension (can I afford this?)
✅ Adds failure state (bankruptcy)
✅ Makes optimization meaningful

### Cons
❌ Can feel punishing ("my money is going down!")
❌ Might discourage experimentation
❌ Could create death spiral (can't afford to fix layout)
❌ Removes "set and forget" satisfaction
❌ Adds mental overhead

### Impact on Gameplay

**Early game:**
- Very tight budget (can barely afford basic factory)
- Must be profitable quickly or go broke
- Stressful, high-stakes

**Mid game:**
- Balance building expansion vs power costs
- Delete inefficient buildings to save power
- Optimization becomes critical

**Late game:**
- Power costs become insignificant (making $500/min, paying $150/min)
- Less impactful

---

## Approach 2: Tiered Power (Building Type Matters)

### How It Works

**Different tiers consume different power:**

```javascript
const POWER_TIERS = {
    // Basic buildings (cheap power)
    BELT: 0.5,
    BELT_F: 1,

    // Production buildings (moderate power)
    ARM: 3,
    ARM_F: 5,
    ARM_S: 4,
    ARM_M: 6,

    // Heavy machinery (expensive power)
    PRESS: 15,
    PRESS_M: 30,
    CNC: 12,
    CNC_M: 25,

    // Utilities (minimal power)
    BIN: 1,
    SRC_P: 2,
    SRC_S: 2,
};
```

**Strategic implications:**
- MEGA buildings are powerful but expensive to run
- Must calculate: "Is 4x output worth 2x power cost?"
- Encourages balanced factories (not just MEGA everything)

### Pros
✅ Adds strategic depth (cost/benefit per building type)
✅ Makes tier choices meaningful (FAST vs regular)
✅ Rewards thoughtful layouts
✅ Creates interesting trade-offs

### Cons
❌ More complex than flat cost
❌ Requires balancing all building costs
❌ Could make expensive buildings feel bad (high upfront + high ongoing)

---

## Approach 3: Power Capacity (Infrastructure System)

### How It Works

**You have a power capacity limit:**

```javascript
let powerCapacity = 100;  // Start with 100 MW
let powerUsed = calculateTotalPower();  // Sum of all buildings

if (powerUsed > powerCapacity) {
    // Brownout! Buildings work at reduced efficiency
    const efficiencyPenalty = powerUsed / powerCapacity;  // e.g., 1.5x = 50% speed
}
```

**Buy generators to increase capacity:**
```javascript
GENERATOR: {
    cost: $5000,
    powerProvided: +50 MW,
    desc: "Increases power capacity by 50 MW"
}
```

**No ongoing cost, just capacity limits**

### Pros
✅ No money drain (less punishing)
✅ Creates planning constraint (must expand power)
✅ Visual/concrete resource (power bar fills up)
✅ Allows "brownout" mechanic (overload = slowdown)
✅ Infrastructure feels meaningful

### Cons
❌ Doesn't create ongoing economic pressure
❌ Generators are one-time purchases (back to current model)
❌ Could feel arbitrary (why can't I just build more?)
❌ Doesn't reward efficiency (unused capacity is wasted)

---

## Approach 4: Hybrid (Capacity + Ongoing Cost)

### How It Works

**Combine capacity limits with operating costs:**

1. **Power capacity** - limits how many buildings you can run
2. **Power cost** - pay for electricity consumed

```javascript
// Infrastructure
powerCapacity = 100 + (generators * 50);
powerUsed = calculateTotalPower();

// Can't exceed capacity
if (powerUsed > powerCapacity) {
    showNotification("INSUFFICIENT POWER - BUILD GENERATOR");
    // Prevent building placement
}

// Pay for what you use
const powerCostPerMW = 0.5;  // $0.50 per MW per minute
money -= (powerUsed * powerCostPerMW) / 60;  // Per second
```

**Generator also costs money to run:**
```javascript
GENERATOR: {
    cost: $5000,           // Upfront
    powerProvided: +50,
    powerConsumed: 5,      // Costs $2.50/min to run
    desc: "Diesel generator. Provides +50 MW, consumes 5 MW (fuel cost)."
}
```

### Pros
✅ Maximum strategic depth
✅ Infrastructure feels important
✅ Economic pressure + planning constraint
✅ Multiple optimization axes

### Cons
❌ Most complex system
❌ Might be overwhelming
❌ Could feel "too realistic" for arcade game
❌ Lots of numbers to track

---

## Alternative: No Operating Costs, But...

### Option A: Maintenance Events

**Buildings degrade over time:**
- After X minutes, building needs maintenance ($)
- Click building → "REPAIR ($500)" button
- Degraded buildings work slower

**Less punishing than constant drain:**
- Only pay when prompted
- Can defer maintenance if needed
- Still creates ongoing costs

### Option B: Property Tax

**Pay tax on total building value:**
- Every X minutes, pay 5% of total factory value
- Tax bill: $2000 (factory value: $40,000)
- Encourages selling unused buildings

**Simpler than per-building power:**
- One number (total value)
- One payment (periodic tax)
- Still creates cost pressure

### Option C: Efficiency Score

**No costs, but efficiency rating:**
- Game tracks: profit/building, idle time, throughput
- Shows grade: A, B, C, D, F
- No mechanical penalty, just bragging rights

**Purely informational:**
- Rewards optimization without punishment
- Safe for casual players
- Encourages mastery

---

## Impact Analysis

### On Core Game Loop

**Current loop:**
1. Build buildings (one-time cost)
2. Wait for money to accumulate
3. Build more buildings
4. Repeat until $1M

**With operating costs:**
1. Build buildings (upfront cost)
2. Monitor profit/loss
3. Optimize to stay profitable
4. Expand carefully
5. Repeat until $1M

**Key difference:** Active management vs passive growth

### On Player Psychology

**Current feeling:**
- "My factory is growing!"
- Positive feedback loop
- Satisfying accumulation

**With operating costs:**
- "Am I profitable?"
- Constant vigilance required
- Anxiety about costs

**Is this fun?** Depends on target audience:
- Optimization fans: YES
- Casual players: MAYBE NOT

### On Victory Condition

**Current ($1M revenue):**
- Pure accumulation
- Always achievable with patience

**With operating costs:**
- Must be profitable to reach $1M
- Bad layouts might never win
- Could create unwinnable states

**Risk:** Frustration if player gets stuck in loss spiral

---

## Case Studies (Other Games)

### Factorio
- No operating costs
- Only pollution (environmental cost)
- Focus on expansion, not efficiency
- **Takeaway:** Unlimited growth is satisfying

### Satisfactory
- No power costs for buildings
- Power is capacity-limited
- Must build more power plants
- **Takeaway:** Infrastructure is fun without ongoing costs

### Cities: Skylines
- Buildings cost money per week
- Balancing budget is core mechanic
- Can go into debt (loans)
- **Takeaway:** Operating costs work for city builders

### Oxygen Not Included
- Resources (oxygen, power) are capacity-limited
- No "money drain"
- Focus on efficiency to survive
- **Takeaway:** Constraints without costs

### This Game's Genre
- **Closer to Factorio than Cities: Skylines**
- Factory builder, not city manager
- Current design rewards expansion
- Operating costs might conflict with core appeal

---

## Recommendation

### Don't Add Operating Costs (Yet)

**Reasons:**
1. **Current model works** - exponential growth is satisfying
2. **Adds complexity** - operating costs require constant attention
3. **Could frustrate** - death spiral / bankruptcy feels bad
4. **Not genre-standard** - Factorio/Satisfactory don't have it
5. **Victory condition mismatch** - $1M revenue assumes positive growth

**Current game is about:**
- Building and optimizing factories
- Watching production scale up
- Creative problem-solving

**Operating costs would make it about:**
- Balancing budgets
- Fear of building "wrong"
- Constant financial monitoring

**That's a different game.**

---

## If You Still Want Economic Pressure...

### Better Alternatives

**Option 1: Milestone-Based Unlocks**
```
Reach $10K revenue → Unlock FAST buildings
Reach $50K revenue → Unlock SMART ARM
Reach $250K revenue → Unlock MEGA buildings
Reach $1M revenue → WIN!
```

Creates goals without ongoing costs.

**Option 2: Efficiency Challenges (Puzzle Mode)**
```
Build a factory that:
- Produces 10 parts
- Uses < $5000 budget
- Occupies < 20 tiles
```

Optimization pressure in optional mode, not core game.

**Option 3: Prestige/New Game+**
```
Win at $1M → Reset with:
- Keep unlocked buildings
- Start at $1000 (instead of $600)
- New goal: $5M revenue
```

Replayability without changing core loop.

**Option 4: Campaign Missions**
```
Mission 1: Produce 5 parts with $1000 budget
Mission 2: Reach $100/min profit
Mission 3: Build factory in 10×10 grid
```

Structured challenges with constraints.

---

## If You Absolutely Must Add Costs...

### Least Disruptive Approach

**Recommendation: Approach 3 (Power Capacity)**

**Why:**
- No ongoing money drain (less punishing)
- Creates planning puzzle (power budget)
- Visible resource (power bar)
- Generators feel like infrastructure investment
- Doesn't change victory condition

**How:**
```javascript
powerCapacity = 100;
powerUsed = buildings.reduce((sum, b) => sum + b.type.power, 0);

// Show in UI
"POWER: 85/100 MW"

// If exceeded
if (powerUsed > powerCapacity) {
    canPlaceBuilding = false;
    showNotification("INSUFFICIENT POWER - BUILD GENERATOR");
}

// Add generator building
GENERATOR: {
    cost: 5000,
    powerProvided: 50,
    icon: '⚡',
    desc: "Diesel generator. +50 MW capacity."
}
```

**Keeps current economic model, adds infrastructure layer.**

---

## Design Philosophy Question

### What Makes This Game Fun?

**Current appeal:**
1. Creative freedom (build any layout)
2. Satisfying growth (watch money go up)
3. Optimization puzzle (make factory efficient)
4. Low stakes (can't lose)
5. Incremental progress (always moving forward)

**Operating costs would add:**
1. ✅ Strategic depth (cost/benefit decisions)
2. ✅ Tension (can I afford this?)
3. ❌ Failure states (bankruptcy)
4. ❌ Stress (constant monitoring)
5. ❌ Punishment (money going down)

**Ask yourself:**
- Do you want players to feel **creative** or **constrained**?
- Should the game be **relaxing** or **challenging**?
- Is failure fun, or frustrating?

**My take:** Current model suits the game's tone. Adding costs changes it significantly.

---

## Alternatives That Add Depth WITHOUT Costs

### 1. Building Limits
```
Level 1: Max 5 arms
Level 2: Max 10 arms
Level 3: Max 20 arms
```

Forces creativity, not just spam.

### 2. Space Constraints
```
Mission: Build factory in 15×15 grid
```

Smaller grid = must optimize layout.

### 3. Speed Challenges
```
Produce 100 parts in 5 minutes
```

Rewards fast, efficient designs.

### 4. Resource Limits
```
Source buildings produce finite resources
Must manage inventory carefully
```

Creates scarcity without money drain.

### 5. Pollution/Heat
```
Buildings generate heat
Too much heat = slowdown
Must space buildings out
```

Environmental constraint, not economic.

---

## Final Recommendation

### Don't Add Operating Costs to Core Game

**Why:**
- Changes fundamental game feel
- Adds stress/complexity
- Could alienate casual players
- Not necessary for depth

**Instead:**
- Keep current one-time cost model
- Add depth through:
  - Puzzle mode (budget/space constraints)
  - Optional challenges (efficiency goals)
  - Advanced buildings (HYPER ARM, etc.)
  - Prestige system (replay with harder goals)

**If you want infrastructure:**
- Add power capacity (Approach 3)
- No ongoing costs
- Generators as expansion requirement
- Keeps current economic model

---

## Playtest Questions (If You Implement It Anyway)

### Evaluation Criteria

**After implementing operating costs, ask:**

1. **Is it fun?**
   - Do players enjoy managing power budget?
   - Or does it feel like a chore?

2. **Does it add depth?**
   - Are strategic decisions more interesting?
   - Or just more complicated?

3. **Is it frustrating?**
   - Do players get stuck in unwinnable states?
   - Can they recover from mistakes?

4. **Does it fit the game?**
   - Does it enhance factory building?
   - Or distract from it?

**If answers are mostly negative → Remove feature**

---

## Conclusion

**Operating costs are a major design decision that changes the game's identity.**

**Current game:** Creative factory builder with satisfying growth
**With op costs:** Economic management game with optimization pressure

**Both are valid**, but they're **different games**.

**My recommendation:** Stick with current model, add depth through:
- Optional puzzle mode
- Campaign challenges
- Prestige system
- Advanced buildings

**If you want infrastructure:** Add power capacity limits (no ongoing costs)

**Only add ongoing costs if:**
- You want a more stressful, challenging game
- You're okay alienating casual players
- You're willing to rebalance everything
- You think budget management is the core fun

**Otherwise, the game is already fun. Don't fix what isn't broken.**
