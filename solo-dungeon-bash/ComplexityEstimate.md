# Complexity Estimate — Solo Dungeon Bash

## Document Snapshot
- **Source:** `projects/solodungeonbash/SoloDungeonBash.pdf`
- **Author:** Felbrigg Herriot (design), Ben Nelson (layout)
- **Publisher:** BookRanger.co.uk, 2007
- **Pages:** 3 (p1 core rules + turn sequence, p2 item shop + fighting + all 10 level tables + boss, p3 blank player sheets)
- **Format:** Free print-and-play roll-and-write

## Rules Density Scan
| Signal | Observation |
|---|---|
| Turn sequence length | 7 steps, all single-sentence |
| Distinct phases | 4 (Movement → Encounter → Recovery → Shop) |
| Sub-systems | 3 (Movement/Grid, Dice Combat, Economy/Shop) |
| Tables | 11 (Level 1–10 + End room) + 1 item/shop table |
| Numerical parameters | ~35 (health, item costs, dice counts, monster stats, grid dimensions) |
| Exception / edge-case rules | ~4 (Buckler⇔Shield exclusivity, Sword⇔Axe, Armour exclusivity, Health cap at 17) |

## Complexity Rating: **SIMPLE**
Matches the "Simple" bucket (1–8 pages, 1–2 primary mechanics) with only mild lift because of the 10 level tables and item exclusivity constraints.

- **Primary mechanics:** Dice combat, grid-based movement with no-revisit constraint.
- **Secondary mechanics:** Resource management (Treasure / Potions / Health), tableau-like item loadout.

## Expected Extraction Confidence: **HIGH (~90%)**
Rules are short, self-contained, and written in imperative voice. The only significant ambiguity is around a few edge cases in movement blocking and item stacking (see `AmbiguousRules.md`). No external card text, no campaign state, no variable player powers.

## Estimated Downstream Effort
| Artifact | Effort |
|---|---|
| GDD | Low — single-document pass |
| Balance sheet | Low — all numbers are on one page |
| Architecture | Low — single-player, single-session, no network |
| Card / component DB | N/A (no cards; monsters live in level tables) |

## Warnings
- **None.** This is well within the "no manual review required" band, though final confidence still depends on how we resolve two edge cases (blocking the End square; multiple potions used mid-combat — see Stage 4).
- The game lacks narrative/story hooks, so an adaptation will likely want to add a light theming layer (monster flavor text, room descriptions).
