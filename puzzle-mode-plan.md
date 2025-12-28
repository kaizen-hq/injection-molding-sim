# Puzzle Mode Design Plan
## "FACTORY RESCUE" Game Mode

### Overview
Add a puzzle mode where players are given pre-placed buildings and must fix broken production lines. Transforms the game from sandbox builder to logic puzzle.

---

## Core Concept

**The Setup:**
- Pre-placed buildings with specific rotations
- Limited money (maybe $200 or $0)
- Clear objective: "Get this factory producing 10 parts" or "Fix the production line"

**The Challenge:**
Players must diagnose what's wrong and fix it within constraints (budget, existing layout, etc.)

---

## Puzzle Types

### 1. "The Missing Link"
**Problem:** All buildings are placed but disconnected
**Solution:** Add 2-3 belts to connect the flow
**Constraint:** Budget forces minimal solution
**Teaches:** Production flow, belt placement

### 2. "Rotation Nightmare"
**Problem:** Everything placed but arms/belts face wrong directions
**Solution:** Rotate buildings to correct orientations
**Mechanic:** Right-click → rotate → place again (50% refund)
**Teaches:** Directionality, flow tracing

### 3. "The Bottleneck"
**Problem:** Factory works but is painfully slow
**Solution:** Identify choke point, upgrade to FAST variant
**Constraint:** Limited budget for upgrades
**Teaches:** Performance optimization, identifying inefficiencies

### 4. "Filter Failure"
**Problem:** Arms exist but have no filters set
**Solution:** Configure arm filters correctly
**Issue:** Plastic and Steel get mixed up without proper filtering
**Teaches:** Arm configuration, inventory management

### 5. "MEGA Mayhem"
**Problem:** MEGA PRESS placed but only gets single molds (should do 4x)
**Solution:** Add CNC_M to cut quad-molds OR fix routing
**Teaches:** MEGA building mechanics, advanced recipes

### 6. "Budget Crisis"
**Problem:** Complex factory needs one expensive upgrade
**Solution:** Scrap other buildings to afford critical piece
**Constraint:** $0 starting money, must salvage for funds
**Teaches:** Cost management, trade-offs

---

## Technical Implementation

### Data Structure

```javascript
const PUZZLES = [
    {
        id: 'tutorial_1',
        name: 'First Production',
        description: 'Connect the factory to produce your first part',
        difficulty: 1,
        startMoney: 300,
        startLevel: 1,
        goal: {
            type: 'parts_sold',
            target: 1,
            message: 'Sell 1 part'
        },
        buildings: [
            { type: 'SRC_P', x: 5, y: 5, rot: 0 },  // Facing right
            { type: 'PRESS', x: 10, y: 5, rot: 0 },
            { type: 'BIN', x: 15, y: 5, rot: 0 }
        ],
        hints: [
            'The press needs plastic AND a mold to produce parts',
            'You\'ll need to add a CNC machine to cut molds from steel',
            'Use belts to connect buildings'
        ]
    },
    {
        id: 'rotation_101',
        name: 'Rotation Nightmare',
        description: 'Everything is backwards! Fix the rotations.',
        difficulty: 2,
        startMoney: 0,
        startLevel: 1,
        goal: {
            type: 'parts_sold',
            target: 5,
            message: 'Sell 5 parts'
        },
        buildings: [
            { type: 'SRC_P', x: 5, y: 5, rot: 2 },  // Facing left (wrong!)
            { type: 'BELT', x: 6, y: 5, rot: 2 },   // Facing left (wrong!)
            { type: 'ARM', x: 7, y: 5, rot: 1 },    // Facing down (wrong!)
            // ... more buildings with wrong rotations
        ],
        hints: [
            'Right-click buildings to remove them (50% refund)',
            'Place them again with correct rotation (R key)',
            'Trace the flow: where should items go?'
        ]
    },
    {
        id: 'mega_mystery',
        name: 'The MEGA Mystery',
        description: 'This MEGA PRESS should produce 4x parts. Why doesn\'t it?',
        difficulty: 4,
        startMoney: 0,
        startLevel: 4,
        goal: {
            type: 'production_rate',
            target: 4,  // 4 parts per cycle
            message: 'Achieve 4x production multiplier'
        },
        buildings: [
            { type: 'SRC_P', x: 5, y: 5, rot: 0 },
            { type: 'SRC_S', x: 5, y: 10, rot: 0 },
            { type: 'CNC', x: 10, y: 10, rot: 0 },      // Regular CNC (problem!)
            { type: 'PRESS_M', x: 15, y: 7, rot: 0 },   // MEGA PRESS
            { type: 'BIN', x: 20, y: 7, rot: 0 }
            // Problem: Regular CNC produces MOLD, but MEGA PRESS needs MOLD_QUAD
        ],
        hints: [
            'MEGA PRESS can produce 4 parts at once',
            'But it needs a QUAD MOLD to do so',
            'Regular CNC only cuts single molds',
            'You might need to scrap something to afford an upgrade'
        ]
    }
];
```

### Goal Types

```javascript
const GOAL_TYPES = {
    parts_sold: (target) => partsSoldTotal >= target,
    production_rate: (target) => /* check parts per minute */,
    money_earned: (target) => lifetimeRevenue >= target,
    time_limit: (target) => elapsedTime <= target,
    efficiency: (target) => /* parts/cost ratio */
};
```

### Code Changes Needed

**Minimal modifications to existing codebase:**

1. **New puzzle mode toggle**
   ```javascript
   let isPuzzleMode = false;
   let currentPuzzle = null;
   ```

2. **Load puzzle function**
   ```javascript
   function loadPuzzle(puzzleId) {
       const puzzle = PUZZLES.find(p => p.id === puzzleId);
       isPuzzleMode = true;
       currentPuzzle = puzzle;

       // Clear existing state
       buildings = [];
       items = [];
       grid = createGrid();

       // Set initial conditions
       money = puzzle.startMoney;
       level = puzzle.startLevel;

       // Place buildings
       puzzle.buildings.forEach(b => {
           const bData = ALL_BUILDINGS[b.type];
           const tile = grid[b.y][b.x];
           tile.building = new Building(bData, b.x, b.y, b.rot);
           buildings.push(tile.building);
       });

       gameRunning = true;
       updateUI();
   }
   ```

3. **Goal checking**
   ```javascript
   function checkPuzzleGoal() {
       if (!isPuzzleMode || !currentPuzzle) return;

       const goal = currentPuzzle.goal;
       let completed = false;

       switch(goal.type) {
           case 'parts_sold':
               completed = partsSoldTotal >= goal.target;
               break;
           case 'money_earned':
               completed = lifetimeRevenue >= goal.target;
               break;
           // ... other goal types
       }

       if (completed) {
           showPuzzleComplete();
       }
   }
   ```

4. **UI additions**
   - New "PUZZLE MODE" button on main menu
   - Puzzle selection screen
   - Goal display during gameplay ("Goal: Sell 10 parts | Current: 3/10")
   - Hint system (press H to show hints)

---

## Player Experience Flow

### Starting a Puzzle

1. Main menu → "PUZZLE MODE"
2. See puzzle list with difficulty stars ⭐⭐⭐
3. Click puzzle → See description + goal
4. "START PUZZLE" → Buildings appear on grid
5. Goal appears in corner: "🎯 GOAL: Sell 10 parts (0/10)"

### Playing

1. Player looks at factory: "Why isn't this working?"
2. Traces flow: Plastic → ??? → Press → ??? → Bin
3. "Oh! The arm is facing backwards!"
4. Rotates arm (or removes + replaces)
5. Factory starts working
6. "Wait, where are the molds?"
7. Realizes no CNC machine exists
8. Adds CNC, connects steel source
9. Success! Parts start flowing
10. Goal updates: "🎯 GOAL: Sell 10 parts (8/10)"
11. **PUZZLE COMPLETE** modal appears

### Completion

**3-Star Rating System:**
- ⭐ Complete the goal
- ⭐⭐ Complete under budget (money remaining > $X)
- ⭐⭐⭐ Complete under time limit (< 2 minutes)

**Replay value:**
- Can replay for better rating
- Leaderboards for speed/efficiency
- Community puzzles

---

## Educational Progression

### Tutorial Puzzles (1-3)
- Basic connections
- Understanding flow
- Rotation basics
- Adding missing buildings

### Intermediate Puzzles (4-6)
- Arm filters
- Multiple production lines
- Resource management
- Bottleneck identification

### Advanced Puzzles (7-10)
- MEGA building mechanics
- Complex routing
- Quad-mold production
- Budget optimization
- Multi-stage production

### Expert Puzzles (11+)
- Minimal budget challenges
- Speed challenges
- Space constraints
- "Can't add buildings, only reconfigure"

---

## Community Features

### Puzzle Sharing (Future)

**Export puzzle as JSON:**
```javascript
function exportPuzzle() {
    const puzzleData = {
        buildings: buildings.map(b => ({
            type: b.type.id,
            x: b.x,
            y: b.y,
            rot: b.rot
        })),
        money: money,
        level: level,
        goal: { type: 'parts_sold', target: 10 }
    };

    const code = btoa(JSON.stringify(puzzleData)); // Base64 encode
    return `PUZZLE-${code}`;
}
```

**Import puzzle:**
```javascript
function importPuzzle(code) {
    const data = JSON.parse(atob(code.replace('PUZZLE-', '')));
    loadPuzzle(data);
}
```

**Community sharing:**
- Players create puzzles in sandbox mode
- Export as code: `PUZZLE-eyJidWlsZGluZ3MiOlt7...`
- Share on Discord/Reddit
- Others import and solve
- "Puzzle of the Day" server

---

## Why This is Brilliant

### Educational Value
- Teaches mechanics without boring tutorials
- Visual debugging (like reading someone's code)
- Natural progression from simple to complex
- Immediate feedback on solutions

### Engagement
- Each puzzle is a mini-challenge (2-5 minutes)
- Perfect for "one more puzzle" addiction
- Satisfying "aha!" moments
- Replayability for better scores

### Accessibility
- Lower barrier than sandbox (no blank canvas anxiety)
- Clear objectives (vs. open-ended building)
- Guided learning path
- Can jump to difficulty level

### Game Design
- Adds structure to sandbox gameplay
- Provides objectives for goal-oriented players
- Showcases advanced mechanics
- Creates shareable content (puzzle codes)

---

## Implementation Priority

### Phase 1: Core Puzzle Mode (MVP)
- [ ] Puzzle data structure
- [ ] Load puzzle function
- [ ] 3 tutorial puzzles
- [ ] Goal checking system
- [ ] Puzzle complete modal
- [ ] Simple puzzle selection menu

### Phase 2: Polish
- [ ] Star rating system
- [ ] Hint system
- [ ] 10 total puzzles (tutorial → advanced)
- [ ] Better UI/UX for puzzle mode
- [ ] Goal progress display

### Phase 3: Community
- [ ] Puzzle export/import
- [ ] Puzzle validation
- [ ] Community puzzle browser
- [ ] Leaderboards
- [ ] "Puzzle of the Day"

---

## Design Notes

### Keep It Simple
- Don't overcomplicate with too many goal types
- Start with 5-10 handcrafted puzzles
- Test each puzzle thoroughly
- Make sure each puzzle teaches ONE concept

### Avoid Frustration
- Always provide hints (H key)
- Allow unlimited retries
- Show goal progress clearly
- No penalties for failure

### Maintain Sandbox Feel
- Puzzles are optional side mode
- Don't lock sandbox behind puzzles
- Keep original game flow intact
- Puzzle mode is "bonus content"

### Balance Difficulty
- Puzzle 1 should be solvable in 30 seconds
- Puzzle 10 should take 5+ minutes for new players
- Expert puzzles can be genuinely hard
- Always playtesting with fresh eyes

---

## Conclusion

Puzzle mode transforms the game from **"build whatever you want"** to **"fix this broken factory"** - adding structure, objectives, and replayability without losing the core sandbox appeal.

It's essentially **visual debugging as a game mechanic**, which is perfect for teaching factory automation concepts in a fun, engaging way.
