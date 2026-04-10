# Rules Extraction & Summary — Solo Dungeon Bash

> Source: `projects/solodungeonbash/SoloDungeonBash.pdf` (3 pages, Felbrigg Herriot, 2007, BookRanger.co.uk)

## Game Objective
Navigate from the **Start** square through 10 levels of the dungeon, reach the **End** square, and defeat **Dracular!** (the Big Bad Boss Monster — 9 Attack Dice + 9 Defence Dice). Die at any point and you lose; get blocked from reaching End and you also lose.

## Player Count & Play Time
- **Players:** 1 (pure solo; no multiplayer rules)
- **Play time:** Not stated in the rulebook. Estimated 20–40 minutes per run based on 10-level grid and combat density.

## Component Inventory (player-supplied)
| Component | Quantity | Role |
|---|---|---|
| Graph paper (or printed player sheet from p3) | 1 sheet per run | Dungeon grid + stats tracker |
| Pen / pencil | 1 | Mark visited squares, update counters |
| Six-sided dice (d6) | Several (at least ~10, scaling with monsters) | Attack / Defence / Room content rolls |

The printed sheet includes: the 9×10 dungeon grid with Start/End protrusions in the middle column, and tracker boxes for **Attack Dice, Defence Dice, Health, Treasure, Potions**.

## Dungeon Layout
- Grid: **9 columns × 10 rows** = 90 normal rooms.
- **Start** is a single square attached below the middle column of row 1.
- **End** is a single square attached above the middle column of row 10.
- Each row is a "**Dungeon Level**" (1 = shallowest, 10 = deepest, adjacent to End).

## Starting Resources
| Resource | Starting Value |
|---|---|
| Health | 17 |
| Attack Dice | 1 |
| Defence Dice | 1 |
| Treasure | 0 |
| Potions | 0 |
| Items owned | none |

Health is capped at **17** (the starting value) and can never exceed it.

## Turn Sequence (7 steps, strict order)
1. **Pick next square and move into it.**
2. **Roll to determine room contents.**
3. If room contains **Treasure** → add 1 to Treasure count.
4. If room contains a **Potion** → add 1 to Potion count.
5. If room contains a **Monster** → fight to the death.
6. **Take any or all Potions** you've collected (convert Potions → Health).
7. **Exchange any Treasure** for Items at the shop.

## Movement Rules
- You start on the Start square.
- Next square must be **adjacent** to your current square. Adjacency **includes immediate diagonals** (king-move / 8-neighbourhood).
- You **may not** enter a square you have already visited.
- You **may** return to previous dungeon levels (the grid is not one-way).
- Each entered square is marked with a circle and connected to the previous square by a line, forming your traced path.
- **Lose condition:** if you maneuver into a position where no legal move can reach the End square, you lose. (The rulebook phrases this as "be careful not to block yourself from getting into End square, because if you do you've lost.")

## Room Content Determination (Level Tables)
When you enter a room, roll 1d6 and consult the table for the **dungeon level of the row** you're in. Monsters have a name and an Attack Dice count.

### Level 1
| d6 | Content |
|---|---|
| 1 | Potion |
| 2–4 | Orc (1 Attack Die) |
| 5 | Empty |
| 6 | Treasure |

### Level 2
| d6 | Content |
|---|---|
| 1 | Orc (1 AD) |
| 2 | Wolf (2 AD) |
| 3–5 | Empty |
| 6 | Treasure |

### Level 3
| d6 | Content |
|---|---|
| 1 | Orc (1 AD) |
| 2 | Wolf (2 AD) |
| 3 | Skeleton (3 AD) |
| 4 | Treasure |
| 5 | Potion |
| 6 | Empty |

### Level 4
| d6 | Content |
|---|---|
| 1 | Orc (1 AD) |
| 2 | Wolf (2 AD) |
| 3 | Skeleton (3 AD) |
| 4 | Evil Warrior (4 AD) |
| 5 | Treasure |
| 6 | Empty |

### Level 5
| d6 | Content |
|---|---|
| 1 | Wolf (2 AD) |
| 2 | Skeleton (3 AD) |
| 3 | Evil Warrior (4 AD) |
| 4 | Devil Bat (5 AD) |
| 5 | Treasure |
| 6 | Empty |

### Level 6
| d6 | Content |
|---|---|
| 1 | Skeleton (3 AD) |
| 2 | Evil Warrior (4 AD) |
| 3 | Devil Bat (5 AD) |
| 4 | Cyclops (6 AD) |
| 5 | Treasure |
| 6 | Potion |

### Level 7
| d6 | Content |
|---|---|
| 1 | Evil Warrior (4 AD) |
| 2 | Devil Bat (5 AD) |
| 3 | Cyclops (6 AD) |
| 4 | Dark Elf (7 AD) |
| 5 | Treasure |
| 6 | Empty |

### Level 8
| d6 | Content |
|---|---|
| 1 | Devil Bat (5 AD) |
| 2 | Cyclops (6 AD) |
| 3 | Dark Elf (7 AD) |
| 4 | Skeleton Lord (8 AD) |
| 5 | Treasure |
| 6 | Empty |

### Level 9
| d6 | Content |
|---|---|
| 1 | Cyclops (6 AD) |
| 2 | Dark Elf (7 AD) |
| 3 | Skeleton Lord (8 AD) |
| 4 | Wizard (9 AD) |
| 5 | Treasure |
| 6 | Potion |

### Level 10
| d6 | Content |
|---|---|
| 1 | Dark Elf (7 AD) |
| 2 | Skeleton Lord (8 AD) |
| 3 | Wizard (9 AD) |
| 4 | Demon (10 AD) |
| 5 | Treasure |
| 6 | Empty |

### End Room (Boss)
| d6 | Content |
|---|---|
| 1–6 | **Dracular!** — 9 Attack Dice + 9 Defence Dice |

> **Observation (not a rule change):** Each level table spans a contiguous, level-dependent slice of the monster progression, sliding one rung deeper per level. Levels 3, 6, 9 are the only "normal" rows that can roll a Potion.

## Combat / Fighting
All monsters except Dracular have **1 Health** — a single unblocked hit kills them. The player has 17 Health and can lose Health across multiple encounters.

### Combat Round (repeat until player or monster is dead)
1. **Monster attacks:** Monster rolls its Attack Dice. Each **6** = 1 Hit.
2. **Player defends:** Player rolls their Defence Dice. Each **6** cancels 1 incoming Hit. Remaining Hits subtract from Player Health. If Health ≤ 0 → **game over**.
3. **Player attacks:** Player rolls Attack Dice. Each **6** = 1 Hit.
4. **Monster defends:** Monster rolls its Defence Dice (if any). Each **6** cancels 1 Hit. Any Hit that survives kills the monster.

Only Dracular has Defence Dice (9). All other monsters skip step 4 effectively.

## Resource Use: Potions
- During step 6 of the turn, the player may "use" any number of accumulated Potions.
- Each Potion used → **–1 Potion, +1 Health**.
- Health may not exceed the starting cap of **17**.
- Potions bought from the shop in step 7 are explicitly usable **only on the next turn** ("Any Potions purchased can be used in your next turn.").

## Economy: Treasure → Shop (step 7 only)
Spend Treasure from the counter to buy items. Any combination that fits the current Treasure total is allowed.

| Cost (Treasure) | Item | Effect | Constraint |
|---|---|---|---|
| 1 | **Buckler** | +1 Defence Die | Can NOT be combined with Shield |
| 1 | **1 Potion** | Adds 1 Potion (usable next turn) | — |
| 2 | **Shield** | +2 Defence Dice | Can NOT be combined with Buckler |
| 2 | **3 Potions** | Adds 3 Potions (usable next turn) | — |
| 3 | **Big Sword** | +1 Attack Die | Can NOT be combined with Big Axe |
| 3 | **6 Potions** | Adds 6 Potions (usable next turn) | — |
| 4 | **Big Axe** | +2 Attack Dice | Can NOT be combined with Big Sword |
| 5 | **Spiky Armour** | +2 Attack Dice AND +1 Defence Die | (see Ambiguity: armour stacking) |
| 6 | **Magical Armour** | +5 Defence Dice | Can NOT be combined with Spiky Armour |

## Scoring / Victory Condition
There is **no** score. The game is strictly binary:
- **Win:** The player defeats Dracular in the End room (Dracular is dead while the player still has Health > 0).
- **Loss:** Any of the following:
  - Player Health reaches 0 at any point.
  - Player moves into a grid position from which the End square is no longer reachable via legal moves.
  - (Implied) Player is forced to make no legal move — though the rulebook does not explicitly state this as a loss condition beyond the blocking clause.

## Setup
1. Print or draw a 9×10 dungeon grid with Start (below row 1, middle column) and End (above row 10, middle column).
2. On the player tracker write: **Attack Dice 1**, **Defence Dice 1**, **Health 17**, **Treasure 0**, **Potions 0**.
3. Place a circle on the Start square. You begin your first turn there.
4. Gather a pool of d6 (at least enough for the biggest expected roll pool, ~12+).

## Extraction Flags (sections needing care)
The rules themselves extract cleanly. Places where the text is *technically* thin:
- **F-1** The rulebook never explicitly states what happens if you *are* in fact blocked mid-game — does the player stop moving and start losing, or is it immediate game-over? (The blocking clause is framed as prevention, not a loss-check procedure.)
- **F-2** Whether the player may drink Potions *during* a combat (between combat rounds) or only in step 6 of the turn sequence.
- **F-3** Whether the player may "unequip" an older weapon (e.g., sell Big Sword) to buy a new exclusive one, or whether the exclusivity is permanent once owned.
- **F-4** Whether Spiky Armour is considered the "armour slot" and therefore mutually exclusive with anything besides Magical Armour (e.g., can you wear Spiky Armour AND a Shield?).

All four are tracked in detail in `AmbiguousRules.md`.
