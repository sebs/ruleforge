# Component Interaction Model — Solo Dungeon Bash

See `InteractionModel.mmd` for the visual diagram.

## Component Inventory

### Physical components
| Component | Quantity | Variants | Digital type |
|---|---|---|---|
| Dungeon grid | 1 | 9×10 + Start/End | `Grid` class |
| Grid cell | 90 normal + 2 special | state: unvisited/visited/current | `Cell` enum/record |
| Level table | 10 + 1 boss | one per row | `LevelTable[]` data asset |
| d6 die | variable pool | none | `Die` or just `random.randint(1,6)` |
| Player tracker | 1 | — | `PlayerState` struct |
| Pen/marker | 1 | — | cursor / tap handler |

### Abstract game entities
| Entity | Count | Variants |
|---|---|---|
| Monster | 11 types | Orc, Wolf, Skeleton, Evil Warrior, Devil Bat, Cyclops, Dark Elf, Skeleton Lord, Wizard, Demon, Dracular |
| Item | 8 types | Buckler, Shield, Big Sword, Big Axe, Spiky Armour, Magical Armour, 1P pack, 3P pack, 6P pack |
| Resource | 3 types | Health, Treasure, Potions |

## Interaction Types

### Triggers (A → B fires automatically)
| Source | Target | When | Notes |
|---|---|---|---|
| Cell enter | Level Table roll | Step 2 of every turn | Except Start & End |
| Level Table → Monster | Combat loop | Any monster result | Immediate, cannot defer |
| Grid Cell = End | Boss Fight | When player enters End | Forced; no abort |
| Hit unblocked | HP reduction | Every combat round | Damage is immediate |
| HP ≤ 0 | Game Over | Combat, at damage check | Terminal |
| Dracular dies | Game Win | Combat, at monster-dead check | Terminal |
| Blocked path | Game Over (see A-1) | At step 1, before pick | Terminal |

### Modifies (A changes state on B)
| Source | Target | Direction |
|---|---|---|
| Treasure Room | Treasure | +1 |
| Potion Room | Potions | +1 |
| Potion used | Health | +1 (capped 17), Potions -1 |
| Shop purchase | Treasure | −cost |
| Shop purchase: weapon | AD pool | +1 or +2 |
| Shop purchase: shield | DD pool | +1 or +2 |
| Shop purchase: armour | AD and/or DD pool | +(1..5) |
| Shop purchase: potion pack | Potions | +(1/3/6) |
| Unblocked monster hit | Health | −(unblocked hit count) |

### Requires (A is gated by B)
| Requirement | Gated action |
|---|---|
| Treasure ≥ cost | Shop purchase of item |
| Potions ≥ 1 | Step-6 potion use |
| Target cell is unvisited, king-adjacent, in grid | Legal next-square pick |
| End reachable via BFS from current | Continuing the run (A-1 strict default) |
| Health > 0 | Continuing at all |

### Blocks (A forbids B)
| Blocker | Blocked |
|---|---|
| Weapon slot contains Big Sword | Buying Big Axe |
| Weapon slot contains Big Axe | Buying Big Sword |
| Shield slot contains Buckler | Buying Shield |
| Shield slot contains Shield | Buying Buckler |
| Armour slot contains Spiky Armour | Buying Magical Armour |
| Armour slot contains Magical Armour | Buying Spiky Armour |
| Visited cell | Moving into that cell |
| Currently in combat | Step 1 of next turn |
| Health ≥ 17 | Gaining more HP from potions (wasted) |
| Purchased this turn | Using newly-purchased potions this turn |

### Transforms (A consumed to become B)
| Consumed | Produced |
|---|---|
| 1 Potion | +1 Health (1:1) |
| N Treasure | 1 Item (permanent stat boost) |
| N Treasure | Potion packs (1/3/6 potions for 1/2/3 Treasure) |
| Unvisited cell (king-adjacent) | Visited cell with content |

## Emergent Interaction Chains

These are combinations that emerge from stacking basic interactions:

### Chain 1: Farming Spiral (the primary strategic loop)
```
Move to adjacent unvisited cell → Roll (67% chance monster on L4+) → Combat (likely hit)
→ Health drops slightly → Continue across row for more treasure → Shop at turn end
→ Stat upgrade → Combats become easier → Spiral outward
```
This is the central gameplay: trade HP for Treasure, convert Treasure to permanent stats, push deeper.

### Chain 2: Potion Stockpile Run
```
On L1 (50% safe) or L2 (empty/easy) → Grab any potions → Don't waste them immediately
→ At step 7 buy 3-Potion packs (2T each) → Enter a deep combat → Use potions to survive
```
This is the "defensive build" counter-strategy — spend early Treasure on potions, not gear.

### Chain 3: Glass Cannon Rush
```
Don't buy defensive items → Save Treasure → Buy Big Axe (+2 AD) ASAP → Kill monsters in 1-2 rounds
→ Take less damage due to faster kills → Boss fight with 5 AD minimum
```
High-skill ceiling: relies on good roll luck to avoid getting one-shot.

### Chain 4: Blocker's Curse
```
Player paths carelessly through centre → Visits every cell in row 5 → End reachability fails via BFS
→ Forced Loss-Blocked
```
An emergent failure mode from greedy pathing. Only an experienced player's intuition avoids this.

### Chain 5: The "1-HP cliff"
```
Player in deep combat → Takes a 2-hit unblocked round → HP drops from 3 to 1
→ Next round monster rolls 6 → Dead before step 6 potion phase
```
This is where A-2 (mid-combat potion use) would most change the game. In strict interpretation, hoarded potions can't save you here.

### Chain 6: Buying power mid-turn
```
Kill monster → Step 6: drink potions to heal → Step 7: buy Shield → Next turn: combat with +2 DD
```
This chain shows the ordering importance: recovery before upgrade, both before next risk.

## Digital Implementation Notes

1. **Slot-based inventory** — represent slots as typed nullables:
   ```
   struct Inventory {
     weapon: Option<Weapon>,
     shield: Option<Shield>,
     armour: Option<Armour>,
   }
   ```
   Buying an item checks that the slot is empty; if filled, the UI greys it out.

2. **Derived stats** — `attack_dice` and `defence_dice` should be computed as `base + inventory_modifiers`, never stored directly. Prevents drift.

3. **Level tables as data** — store as a JSON/YAML asset so balance can be tuned without recompiling.

4. **Potion buffer** — separate `potions_available` (usable this turn) from `potions_pending` (usable next turn). Merge the pending into available at turn start.

5. **Combat engine** — a simple async FSM; unit-test all branches with fixed dice. Dice injection enables deterministic tests.

6. **Reachability check** — plain BFS/flood-fill on the grid, called at every move commit. O(90) nodes, trivially fast.
