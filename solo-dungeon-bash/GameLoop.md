# Game Loop — Solo Dungeon Bash

## Diagram
See `GameLoop.mmd` for the Mermaid flowchart. Render with any Mermaid-compatible tool.

## Nested Loop Hierarchy

Solo Dungeon Bash has **four nested loops** running at different granularities:

### 1. Atomic Loop — Combat Round (seconds)
The innermost loop, spun up only inside a Monster room:
```
Monster attack roll → Player defence → HP damage → Player attack roll → Monster defence → kill check → (repeat)
```
- **Exit:** Monster dies OR Player HP ≤ 0.
- **Duration:** Usually 1 round for normal monsters (they have 1 HP). Dracular can take many rounds because of 9 DD.

### 2. Primary Loop — Turn (minutes)
The visible game loop the player experiences turn-by-turn:
```
Pick square → Roll contents → Resolve (empty/treasure/potion/monster) → Potion recovery → Shop → next turn
```
- **Exit (turn):** End of step 7 — next turn begins.
- **Exit (run):** Entering End (boss fight), Player HP ≤ 0, or structural blockage.
- **Duration:** 10–90 turns per run depending on path length.

### 3. Secondary Loop — Level Progression (rounds/minutes)
Implicit rather than explicit: as the player moves up through rows, the level table in use changes, and the monster mix/reward mix updates. There's no explicit "end of level" event — level is a function of position. But a "level loop" can be described as:
```
Enter new row → New risk profile → Decide to stay / ascend / descend → …
```
- **Exit:** Crossing to an adjacent row, or the End square.

### 4. Tertiary Loop — Game Arc (one run)
The whole expedition:
```
Start (HP 17, 1AD/1DD, empty inventory) → Early farming on L1–L3 → Mid upgrade phase L4–L6 → Deep push L7–L10 → Dracular → Win/Loss
```
- **Exit:** Dracular dead (win), player dead, or blocked (loss).
- **Duration:** One full run ≈ 20–40 minutes.

## Loop Properties
| Property | Value |
|---|---|
| Loop completeness | Complete — every state has a defined exit (win / lose / next turn). |
| Termination guarantee | Guaranteed — the grid is finite and every move consumes one square. Max ~92 moves. |
| Parallel loops | None — strictly sequential. Combat pauses the turn loop, shop/potions wrap every turn. |
| Player agency points per turn | 2 (pick square, choose potion usage, choose purchases). Combat itself is deterministic (no choice). |

## Exit Conditions Summary
| Exit | Where it fires | Outcome |
|---|---|---|
| Monster killed player | Atomic loop, combat step 2 | **LOSS — Dead** |
| Player killed monster | Atomic loop, combat step 4 | Combat exits; if boss → **WIN**; else → continue turn |
| Player entered End & killed Dracular | Turn loop, boss path | **WIN — Dungeon Cleared** |
| Player has no legal move or End is unreachable | Turn loop, step 1 | **LOSS — Blocked** (see A-1) |
| Player chooses next square | Turn loop, step 1 | Turn proceeds normally |

## Where Friction Lives
- **Step 1 (pick square)** is the main decision point — it determines the level table, the path topology, and the risk profile.
- **Step 7 (shop)** is the main progression point — it's where long-term power increases happen.
- Steps 2–5 are deterministic-once-rolled; step 6 is a small tactical heal choice.

## Digital Implementation Notes
- The main player loop (Primary) is a simple finite state machine with states: `pick_square`, `rolling_content`, `resolving_content`, `in_combat`, `recovery_phase`, `shop_phase`, `turn_end`.
- Combat (Atomic) is a smaller FSM: `monster_atk`, `player_def`, `check_dead`, `player_atk`, `monster_def`, `check_monster_dead`.
- No timers, no turn clock — the game is strictly player-paced. This makes the digital loop trivially pausable/save-anywhere.
