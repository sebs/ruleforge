# Balance Parameter Sheet — Solo Dungeon Bash

See `BalanceSheet.csv` for the full parameter list. This document covers the narrative analysis and sensitivity assessment.

## Parameter Categories
| Category | Count |
|---|---|
| Player stats | 4 (HP, HP cap, base AD, base DD) |
| Combat mechanics | 8 (success threshold, monster HP, boss stats, flee rule, pool caps) |
| Dungeon layout | 6 (grid, adjacency, revisit, path length) |
| Monster tables | 10 (one per level) |
| Economy (rooms) | 5 (treasure/potion probabilities + conversion ratio) |
| Shop items | 14 (costs, bonuses, exclusivities) |
| **Total** | **~48 tunable values** |

## Expected Damage Model (for reference)
With a "count 6s on d6" system, expected hits per die = **1/6 ≈ 0.1667**.
- **Player 1 AD:** expected 0.167 hits/round → vs 1-HP monster, ~1/6 chance per round → expected ~6 rounds to kill, each round taking monster attacks.
- **Player 3 AD:** expected 0.5 hits/round → ~2 rounds expected to kill.
- **Player 5 AD:** expected 0.833 hits/round → ~1.2 rounds expected to kill.

Monster attack vs. player defence (expected unblocked hits per round):
- 1 AD vs 1 DD: E[hits] ≈ 0.167, E[blocks] ≈ 0.167 → net ≈ 0.028 HP/round.
- 5 AD vs 1 DD: E[hits] ≈ 0.833, E[blocks] ≈ 0.167 → net ≈ 0.694 HP/round.
- 10 AD vs 1 DD: E[hits] ≈ 1.667, E[blocks] ≈ 0.167 → net ≈ 1.5 HP/round.
- 10 AD vs 8 DD: E[hits] ≈ 1.667, E[blocks] ≈ 1.333 → net ≈ 0.334 HP/round.

These figures are approximations (independent dice mean `E[max(0, hits-blocks)]` is slightly less than `E[hits]-E[blocks]`) but they show the scale.

## Top 5 Most Sensitive Parameters
Ordered by impact on overall win-rate for a small change.

### 1. `die_success_threshold` (currently 6 on d6)
**Why sensitive:** Changing from "only 6" to "5-6" **doubles** the effective hit rate on every single roll in the game — combat, defence, and even content would be affected. A one-pip shift collapses or explodes difficulty. **Do not touch without full rebalance.**

### 2. `starting_health` (currently 17)
**Why sensitive:** HP directly sets the combat survival budget. +2/-2 HP changes the expected "rounds before death" by 12% in either direction. Combined with the HP cap, it also determines how much potion stockpiling matters. Recommended tuning range: 14–20.

### 3. `monster_l10_max_attack_dice` (currently 10 — Demon)
**Why sensitive:** On Level 10 a 10 AD roll can one-shot any under-geared player. Reducing to 8 or 9 substantially increases survival odds of bad rolls. A +1 change at this level swings win-rate noticeably.

### 4. `dracular_defence_dice` (currently 9)
**Why sensitive:** The final combat pivots entirely on this. With 9 DD Dracular blocks ~1.5 hits/round, meaning a 5 AD player expects to land a hit roughly every ~3 rounds while taking ~1.5 hits/round themselves — a coin-flip fight. Dropping to 6 DD makes the boss a pushover; raising to 12 DD makes the game nearly impossible.

### 5. `grid_width` × `grid_height` (currently 9 × 10)
**Why sensitive:** More squares means more upgrades available before committing to the End room. A 7×8 grid would starve the player of treasure; a 12×12 grid would let the player over-equip and trivialize the boss. The ratio of total rooms to required path length is what controls "farming headroom."

## Tier 2 Sensitivity (notable but not as sharp)
- **`potion_to_hp_ratio`** (1:1) — changing to 1:2 or 2:1 would make potions dramatically more or less valuable as a strategic resource.
- **`max_attack_dice_pool` cap** (5 via Big Axe + Spiky Armour) — determines ceiling of player offence. A +1 bump fundamentally changes the boss math.
- **`potion_probability` on L1/3/6/9** — the vertical spacing of potion-yielding levels determines how long the player goes without healing options. Closing gaps (add potions on L5/L8) would ease the mid-game.
- **`shield_cost` vs `big_sword_cost`** — the shield is dramatically under-costed compared to the sword (2T for +2 DD vs 3T for +1 AD). Defensive builds are clearly favoured.

## Pacing / Tempo Observations
- **Expected Treasure accrual:** In 30 turns at 1/6 treasure probability, expected treasure = 5. That's barely enough for Big Axe + 1 Potion. The player must push *deep* early to see meaningful gear.
- **Expected Potion accrual:** In 30 turns with 4 rows offering potions at 1/6 each, expected potions found ≈ 2. Extra potions must come from the shop (2T/3 potions = efficient).
- **"Dominant path":** Upwards-only through the middle column is 11 turns — way too short to farm enough gear. The player *must* snake through adjacent rows.

## Balance Red Flags
| Flag | Parameter(s) | Severity |
|---|---|---|
| Shield strictly dominates Buckler (same exclusion slot, 2× better for 2× cost) | `shield_cost`, `buckler_defence_bonus` | Low — the Buckler is still early-accessible |
| Big Sword strictly dominated by Big Axe in T/AD ratio (3T for 1AD vs 4T for 2AD) | `big_sword_cost`, `big_axe_cost` | Medium — Big Sword is essentially a "saving up" stepping stone |
| Magical Armour (6T, +5 DD) strictly dominates Shield (2T, +2 DD) per-die cost BUT they stack if A-3 literal | `magical_armour_cost`, `shield_cost` | Low — different tiers of commitment |
| Spiky Armour's hybrid bonus (2 AD + 1 DD for 5T) is superior to Big Axe + Buckler (2 AD + 1 DD for 5T) only if one of them stacks with other items better | `spiky_armour_*` | Low — interesting design choice |
| Dracular's 9/9 line means even max-gear players win only ~55-60% | `dracular_attack_dice`, `dracular_defence_dice` | **Intentional tension — this is the point of the game** |

## Digital Annotations
The CSV includes a `digital_recommended` column. For Solo Dungeon Bash, recommended = source for *every* parameter — the game doesn't need rebalancing for digital. The real value of the sheet is to let a tuner quickly adjust any value in a data file for difficulty modes.

## Proposed Difficulty Mode Tuning (optional, not in source rules)
| Mode | Delta | Expected effect |
|---|---|---|
| Easy | Starting HP 20, HP cap 20, +1 starting DD | ~60% → ~75% win rate |
| Normal | Source values | Baseline |
| Hard | Starting HP 14, HP cap 14, potion→HP ratio 1:1 (unchanged), max 4 potions carried | ~60% → ~45% win rate |
| Nightmare | Hard + Dracular gets 10 AD / 10 DD | ~60% → ~30% win rate |

These are presented as tunables, not recommendations. The source game is already a balanced single-mode experience.
