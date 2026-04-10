# Game Design Document — Solo Dungeon Bash (Digital)

> Source: Felbrigg Herriot (2007), *Solo Dungeon Bash*, BookRanger.co.uk. 3-page print-and-play rulebook.
> Extracted via RuleForge on 2026-04-10.

---

## 1. Overview

### Elevator Pitch
A lightning-fast solo dungeon crawl where every step down into the dungeon is a dice roll and every decision is whether to push deeper or retreat for treasure. Climb 10 dungeon levels, grab loot, trade it for weapons and armour, and face the final boss — all in a single 20-to-40-minute run. A faithful digital adaptation of a classic pen-and-paper roll-and-write.

### Genre
Solo roll-and-write dungeon crawler / light roguelite. Turn-based. Grid-based movement. Dice combat.

### Player count
**1 player.** No multiplayer variants in source rules.

### Session length
**20 to 40 minutes per run.** Runs are self-contained; no metaprogression between runs by default.

### Target platforms
- **Primary:** Web (HTML5 / mobile web).
- **Secondary:** iOS, Android (PWA).
- **Stretch:** Desktop (Steam build via web wrapper).

### Design pillars
1. **Every step is a gamble.** Every square reveals its content only on entry.
2. **Permanent progression within the run.** Each upgrade changes every future combat.
3. **Self-drawn puzzle.** You are literally drawing your own dungeon as you explore.
4. **Fair loss, clear death.** Deaths feel earned; the player can always see why.

---

## 2. Game Loop

See `GameLoop.mmd` and `GameLoop.md` for full diagram and narrative.

### In brief
Every turn follows 7 strict steps:
1. Pick a king-adjacent unvisited square and move in.
2. Roll 1d6 against the current row's Level Table.
3. If Treasure → +1 Treasure.
4. If Potion → +1 Potion.
5. If Monster → fight to the death (alternating attack/defence rounds, count 6s on d6 pools).
6. Use any number of held potions (1 Potion → +1 HP, capped at 17).
7. Spend Treasure at the shop for permanent upgrades or more potions.

The run ends when the player either:
- Reaches the End square, fights Dracular (9 AD / 9 DD), and kills him → **WIN**.
- Reaches 0 HP at any point → **LOSS (dead)**.
- Moves such that the End square is no longer reachable via unvisited squares → **LOSS (blocked)**.

### Loop granularity
- **Atomic loop** = one combat round (~2–5 seconds).
- **Primary loop** = one turn (~15–60 seconds).
- **Secondary loop** = level transitions (position-driven).
- **Tertiary loop** = one full run.

---

## 3. Mechanics

Full taxonomy: `Mechanics.md`.

### Primary mechanics
- **Dice Rolling (count 6s on d6 pool)** — the universal resolution system.
- **Grid Exploration with Roll-and-Write** — 9×10 dungeon, king-move, no revisiting.
- **Resource Management (HP / Treasure / Potions)** — three interlocking pools, HP capped at 17.

### Secondary mechanics
- **Random Encounter Tables** — 10 d6 tables, one per level, sliding in monster strength.
- **Item Shop / Loadout Tableau** — end-of-turn market with 6 permanent items across 3 exclusive slots.
- **Push-Your-Luck** — deciding when to push deeper.
- **Hand Management** — stockpiling potions; bought potions delayed by 1 turn.
- **Engine Building** — permanent stat upgrades compound.
- **HP Combat with Defence Cancels** — initiative fixed to monster; both sides roll hits vs. defence.

### Tertiary mechanics
- **Self-Blocking Pathfinding Constraint** — the no-revisit rule creates a soft puzzle layer over movement.

---

## 4. Balance Parameters

Full parameter sheet: `BalanceSheet.csv` / `BalanceSheet.md`.

### Key player values
| Parameter | Value | Notes |
|---|---|---|
| Starting HP | 17 | Also the cap |
| Starting Attack Dice | 1 | |
| Starting Defence Dice | 1 | |
| Die success threshold | 6 on d6 (1/6 ≈ 17%) | Universal |
| Maximum Attack Dice (all bought) | 5 | 1 + 2 (Big Axe) + 2 (Spiky Armour) |
| Maximum Defence Dice (all bought) | 7–8 | Depends on Spiky/Shield stacking (see A-3) |
| Dracular | 9 AD / 9 DD | Only monster with DD |
| Normal monster HP | 1 | All non-boss monsters die on first unblocked hit |
| Grid | 9 × 10 = 90 rooms | +2 special (Start/End) |

### Top 5 sensitive parameters
1. Die success threshold (6 → 5–6 would double success rate system-wide)
2. Starting HP
3. Level 10 monster max AD
4. Dracular's DD
5. Grid dimensions

### Shop / Item costs
| Cost (T) | Item | Bonus | Slot |
|---|---|---|---|
| 1 | Buckler | +1 DD | Shield (XOR Shield) |
| 1 | 1 Potion | +1 Potion | — |
| 2 | Shield | +2 DD | Shield (XOR Buckler) |
| 2 | 3 Potions | +3 Potions | — |
| 3 | Big Sword | +1 AD | Weapon (XOR Axe) |
| 3 | 6 Potions | +6 Potions | — |
| 4 | Big Axe | +2 AD | Weapon (XOR Sword) |
| 5 | Spiky Armour | +2 AD, +1 DD | Armour (XOR Magical) |
| 6 | Magical Armour | +5 DD | Armour (XOR Spiky) |

---

## 5. Digital Adaptation Notes

Full report: `AdaptationGap.md`.

### Adaptation difficulty
**Low.** 0 mechanics require redesign, 3 need simple adaptation, 6 transfer directly.

### Required adaptations
- **Item shop UI** — slot exclusivity, greyed-out purchased items, pay-wall affordance display.
- **Potion timing buffer** — separate "available now" vs. "bought this turn" counters.
- **Blocked-path detection** — BFS every move, warn/confirm before committing.

### Opportunities unique to digital
Undo last move, run history/stats, daily seeds, difficulty modes, auto-save, achievements, sound design, dice animations.

### What we should NOT add
- Combat flee mechanic (source forbids it; adds tension).
- Mid-combat potion use (breaks the strict turn sequence; see A-2).
- In-run respawns (the game is roguelike-lethal on purpose).
- Multiplayer (out of scope — the source game has no multiplayer rules).

---

## 6. Component Interaction Model

Full model: `InteractionModel.md` and `InteractionModel.mmd`.

### Core entities
- **Grid** (9×10 cells) with special Start/End
- **Level Tables** (11 = 10 levels + boss)
- **Player State** (HP, AD, DD, Treasure, Potions)
- **Inventory** (3 slots: weapon / shield / armour)
- **Room Entities** (Monster, Treasure, Potion, Empty)

### Key interactions
- Level Tables produce room content via d6 roll.
- Rooms trigger resource updates or combat.
- Combat rolls AD/DD pools and counts 6s.
- Shop purchases modify inventory which modifies AD/DD pools.
- Movement consumes cells; the grid never replenishes.

### Notable interaction chains
- **Farming Spiral** — core gameplay loop.
- **Potion Stockpile Run** — defensive build path.
- **Glass Cannon Rush** — offensive build path.
- **Blocker's Curse** — emergent fail state from greedy pathing.

---

## 7. Onboarding & Tutorial Design

See `OnboardingDesign.md` for the full tutorial script.

### Complexity rank (for tutorial ordering)
1. **Movement** (easiest) — click a highlighted adjacent square.
2. **Room rolls** — show a die, reveal content.
3. **Resources** — animate +1 Treasure / +1 Potion counters.
4. **Combat** — step through a slow first combat with explanations.
5. **Step 6 (potions)** — explain recovery phase.
6. **Step 7 (shop)** — explain currency and item slots.
7. **Exclusivity** — show Buckler/Shield slot when the player tries to buy both.
8. **Path-blocking warning** — triggered on first near-miss.
9. **Boss fight** — narrative flourish + full stakes warning.

### Progressive disclosure
- Turn 1: only the 3 legal starting squares are highlighted.
- Turn 1 combat (if rolled): slow-motion combat with tooltip per step.
- Turn 2: show shop for the first time (only if player has Treasure).
- Turn 5: first hint about path-blocking if the player is trending into a corner.
- Turn 10+: full UI unlocks, no more hand-holding.

---

## 8. Ambiguous Rules

Full list: `AmbiguousRules.md`. 10 items flagged.

### High-impact ambiguities
| ID | Summary | Default |
|---|---|---|
| A-1 | When is "blocked" checked? | Every move, BFS strict |
| A-2 | Potions usable mid-combat? | No, strict — step 6 only |
| A-3 | Spiky Armour's slot semantics | Literal — stacks with Shield & Weapon |
| A-4 | Duplicate item purchases? | No — single copy per item |

### Low-impact ambiguities
A-5 to A-10 are either trivially resolvable or cosmetic.

### Meta-note
The source rules are unusually clean for a 2007 print-and-play: the ambiguities are all edge cases rather than contradictions.

---

## 9. Confidence Assessment

**Overall Extraction Confidence: 91% (High).** See `Confidence.md` for the full breakdown.

| Section | Confidence |
|---|---|
| Rules Extraction | 95% |
| Mechanics Identification | 94% |
| Game Loop | 95% |
| Loop Validation | 93% |
| Balance Parameters | 92% |
| Adaptation Gap | 92% |
| Ambiguous Rules | 90% |
| Interaction Model | 90% |
| Architecture | 90% |
| User Stories | 89% |
| Feature List | 88% |
| Onboarding Design | 85% |
| **Overall (weighted)** | **91%** |

Sub-90% sections reflect *design judgement calls* (onboarding, feature prioritization, story scoring) rather than extraction uncertainty. The 9% residual uncertainty is dominated by item-stacking semantics (A-3), mid-combat potion timing (A-2), and the lack of empirical balance playtesting. **This extraction is READY for developer handoff.**

---

## 10. Development Considerations

### Scope for MVP
A viable MVP implements:
- Grid, movement, level tables, combat, potion recovery, shop.
- 1 difficulty mode (source-faithful).
- Simple sketch-style art.

Stretch: daily seeds, achievements, difficulty modes, bestiary.

### Tech stack recommendation (non-binding)
- **Frontend:** React or Svelte + TypeScript + Canvas/SVG.
- **State:** Plain finite state machine (e.g. XState or a hand-rolled reducer).
- **Persistence:** localStorage for save state; optional backend for leaderboards.
- **Audio:** Howler.js or native Web Audio.
- **Art:** Sketch/hand-drawn style — reinforces the "roll-and-write" origin.

### Team shape (non-binding)
One developer + one artist + one audio pass. Effort estimate: small. This is a weekend-to-two-weeks MVP for an experienced solo dev.

---

## Appendix A — Art Direction
The source is unillustrated (a pure text-and-grid rulebook). Recommended style:
- **Hand-drawn pencil / ink on graph paper** aesthetic.
- Monochrome grid with selective colour for Treasure (gold), Potions (blue), Monsters (red), current square (green).
- Monsters depicted as **tiny inked sketches** in room tooltips.
- UI chrome rendered as a **player tracker sheet**, similar to page 3 of the source PDF.
- Typography: handwritten fonts for player-facing labels; clean sans-serif for numbers.

## Appendix B — Sound Direction
- **Pen scratches** on room mark.
- **Dice clatter** on content rolls and combat.
- **Parchment rustle** on shop open.
- **Heartbeat** as HP drops below 5.
- Light ambient dungeon hum; no intrusive music.

## Appendix C — Output file map
All RuleForge artefacts for this game live in `output/solo-dungeon-bash/`. Referenced files: `ComplexityEstimate.md`, `RulesExtraction.md`, `Mechanics.md`, `AmbiguousRules.md`, `GameLoop.mmd`, `GameLoop.md`, `LoopValidation.md`, `AdaptationGap.md`, `BalanceSheet.csv`, `BalanceSheet.md`, `InteractionModel.md`, `InteractionModel.mmd`, `OnboardingDesign.md`, `Features.csv`, `Features.md`, `FeatureDeps.mmd`, `Architecture.mmd`, `Architecture.md`, `Stories.csv`, `Stories.md`, `Confidence.md`, `PrototypePrompts.md`.
