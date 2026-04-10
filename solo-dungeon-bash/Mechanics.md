# Mechanics Identification & Categorization — Solo Dungeon Bash

## Mechanics Overview

| # | Mechanic | Category | Role |
|---|---|---|---|
| M1 | Dice Rolling (pool, count-sixes) | Dice Rolling | **Primary** |
| M2 | Roll-and-Write Grid Exploration | Programmed Movement / Roll-and-Write hybrid | **Primary** |
| M3 | Resource Management (HP / Treasure / Potions) | Resource Management | **Primary** |
| M4 | Random Encounter Table Lookup | Dice Rolling (stochastic content) | Secondary |
| M5 | Item Shop / Loadout Tableau | Tableau Building | Secondary |
| M6 | Push-Your-Luck (depth vs. risk) | Push Your Luck | Secondary |
| M7 | Hand/Inventory Management (Potions) | Hand Management | Secondary |
| M8 | Character Progression (Permanent upgrades) | Engine Building | Secondary |
| M9 | Hit-Point Combat (with defence cancels) | Dice Rolling (combat subsystem) | Secondary |
| M10 | Self-Blocking Pathfinding Constraint | Programmed Movement | Tertiary |

## Detailed Mechanics

### M1 — Dice Rolling (Primary)
- **Category:** Dice Rolling
- **How it works:** Every stochastic event in the game is resolved by rolling one or more d6 and either consulting a table (room content) or counting 6s (combat hits and defence cancels). The game uses *count successes* rather than sum.
- **Key parameters:**
  - Success threshold: fixed at 6 (1/6 per die).
  - Starting dice pools: Attack = 1d6, Defence = 1d6.
  - Maximum practical Attack pool (all additive items): 1 + 2 (Big Axe) + 2 (Spiky Armour) = **5 AD** → expected ~0.83 hits/round.
  - Maximum practical Defence pool (all additive items): 1 + 2 (Shield) + 5 (Magical Armour) = **8 DD** → expected ~1.33 blocks/round.
  - Boss pool: 9 AD / 9 DD.

### M2 — Roll-and-Write Grid Exploration (Primary)
- **Category:** Programmed Movement with Roll-and-Write content generation.
- **How it works:** The 9×10 dungeon grid is a shared movement space. Each turn the player chooses an adjacent (king-move) unvisited square, draws it, rolls the level table, and resolves the result. The player plots a self-avoiding path from Start to End. Players may move both upward (towards the boss) and back down to farm lower-risk rows.
- **Key parameters:**
  - Grid: 9 × 10 = **90 rooms** + 2 special (Start, End) = **92 squares**.
  - Neighbourhood: 8-way king (adjacent + diagonals).
  - Revisit: forbidden.
  - Minimum path length Start → End: 11 moves (straight line through middle column).
  - Maximum path length: bounded by grid size (self-avoiding Hamiltonian-style walks can theoretically visit nearly every square).

### M3 — Resource Management (Primary)
- **Category:** Resource Management
- **How it works:** The player juggles three interlocking resources:
  - **Health (17 max)** — life total. Lost to unblocked monster hits. Restored by Potions. Hard-capped at starting value.
  - **Treasure** — currency earned from Treasure rooms. Spent at the end-of-turn shop on items (permanent) or potions (consumable).
  - **Potions** — consumable heals (+1 HP each). Can stockpile, convert to HP only during step 6 of a turn.
- **Key parameters:** Starting HP 17, HP cap 17, Potion→HP ratio 1:1, Treasure costs 1–6 per item.

### M4 — Random Encounter Table Lookup (Secondary)
- **Category:** Dice Rolling (content)
- **How it works:** Each of 10 rows has its own d6 encounter table. As the player ascends, each table "slides" upward in monster strength (Level 1 peaks at 1-AD Orcs, Level 10 peaks at 10-AD Demons). Treasure & Potion probabilities vary by row.
- **Key parameters:**
  - Monster probability per level (proportion of 1–6 rolls yielding a monster): L1 60% · L2 33% · L3 50% · L4 67% · L5 67% · L6 67% · L7 67% · L8 67% · L9 67% · L10 67%.
  - Treasure probability: L1–L5 17%, L6 17%, L7–L10 17% (flat, except L2 16.7% = 1/6).
  - Potion probability: L1 17%, L3 17%, L6 17%, L9 17%, elsewhere 0%.
  - Empty probability: varies 0–50% (L2 highest at 50%, L1 & L4/5/7/8/10 at 17%, L3/L6/L9 at 17%).

### M5 — Item Shop / Loadout Tableau (Secondary)
- **Category:** Tableau Building / Market Purchase
- **How it works:** At the end of each turn, the player may spend accumulated Treasure at a fixed, always-available shop. Items are permanent stat modifiers that change the dice pools for all future combats. Several items form mutually-exclusive "slots" (weapon: Sword/Axe, small-shield: Buckler/Shield, armour: Spiky/Magical).
- **Key parameters:**
  - Item cost range: 1–6 Treasure.
  - Three exclusive slots → best possible equipped loadout = (Big Axe) + (Shield or Magical Armour) + (Spiky Armour) ... **with armour exclusivity the top combination is ambiguous — see `AmbiguousRules.md` A-3**.
  - Dominant strategy suggestion: Big Axe + Magical Armour (+2 AD, +5 DD) OR Big Axe + Shield + Spiky Armour (+4 AD, +3 DD) depending on offensive vs. defensive bias.

### M6 — Push-Your-Luck (Secondary)
- **Category:** Push Your Luck
- **How it works:** The player decides when to advance deeper (higher monster tables, higher reward density, higher risk) vs. when to detour through already-explored levels to farm treasure or find potions. Going deeper early risks fighting high-tier monsters under-equipped; staying shallow risks running out of legal moves to reach End.
- **Key parameters:** Driven by the level table slide and the no-revisit constraint (M2). The player has no explicit "press/stop" button; pressing is implicit in choosing the next square.

### M7 — Hand Management (Potions) (Secondary)
- **Category:** Hand Management
- **How it works:** Potions are a stockable consumable resource with a "timing" constraint: freshly purchased Potions can only be used on the *next* turn, never the same turn they were bought. Found Potions (from rolled rooms) appear to be available the same turn they were collected, at step 6.
- **Key parameters:** Arrival delay on bought potions = 1 turn. 1 Potion = 1 HP. HP cap blocks over-healing.

### M8 — Character Progression / Engine Building (Secondary)
- **Category:** Engine Building
- **How it works:** Every item purchase permanently boosts the player's combat engine (more Attack or Defence Dice), compounding the player's advantage. Early Treasure is worth more than late Treasure because early upgrades apply to more future combats. The ladder of items gives a rough XP-style progression curve without an XP score.
- **Key parameters:**
  - Max raw AD with all stackable weapons/armour: **1 + 2 + 2 = 5** (Big Axe + Spiky Armour).
  - Max raw DD: **1 + 2 + 5 = 8** (Shield + Magical Armour) — BUT exclusivity A-3 may restrict this.
  - Time-value of Treasure: steep early, flat late.

### M9 — Hit-Point Combat Subsystem (Secondary)
- **Category:** Dice Rolling (combat)
- **How it works:** Alternating-round combat with initiative fixed: monster always attacks first. Each round both sides roll Attack → opponent rolls Defence → unblocked hits apply. Monsters have 1 HP so combat almost always ends on the player's first successful turn — unless the player is one-shotted first.
- **Key parameters:**
  - Monster HP: always 1 (except Dracular, also 1 but with 9 DD).
  - Player HP: up to 17.
  - Initiative: fixed to monster.

### M10 — Self-Blocking Pathfinding Constraint (Tertiary)
- **Category:** Programmed Movement
- **How it works:** Because revisit is forbidden, the player can paint themselves into a corner and become unable to reach End. This turns movement into a soft puzzle layered on top of the combat game — you have to plan routes so that every legal branch still has a legal continuation to End.
- **Key parameters:** The only fail-state here is a Hamiltonian-style trap; practically rare unless the player deliberately spirals.

## Mechanic Interaction Summary
- **M2 feeds M4** (grid position determines table used).
- **M4 feeds M3 and M9** (room content yields resources or combat).
- **M3 feeds M5** (Treasure buys items; items feed back into M9).
- **M5 feeds M9** (stat upgrades change combat outcomes).
- **M9 feeds M3** (lost Health, consumed Potions).
- **M6 sits across M2 and M8** (meta-decision about when to push deeper).
- **M10 is a constraint on M2** (movement options get structurally pruned, not just tactically).

## Notable Absences
- **No deck of cards** → no deck-builder / draft / set-collection mechanics.
- **No auction / negotiation** (purely solo).
- **No hidden information** (the level tables are public and deterministic once rolled).
- **No tempo or time limit** — runs can be as long as the player wants within grid limits.
- **No narrative branches** — the game has no story, only emergent mini-stories.
