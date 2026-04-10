# Genre Crystallization — Solo Dungeon Bash → Real-Time

> RealTimeForge Stage RT-6: Synthesis of Wave 1 analyses into a concrete genre identity.
> Inputs: `TemporalMap.md`, `SpatialModel.md`, `AgencyModel.md`, `InfoArchitecture.md`, `ConflictModel.md`, `EconomyModel.md`.

---

## Working Title

**Solo Dungeon Dash** *(working title — preserves the source's initials SDB→SDD, swaps "bash" for the active-verb "dash")*

Alternates for review: *Sixes*, *Dracular's Keep*, *Ink & Dice*, *One-Hit Dungeon*, *Pocket Dungeon* (already taken), *Parchment Crawl*.

---

## 1. Synthesis of Wave 1 Picks

| Axis | Wave 1 Pick | Source |
|---|---|---|
| **Temporal feel** | Beat-based real-time, player-paced locomotion, combat beats ~1.5–3s for normal mobs, 60–120s boss | RT-0 |
| **Spatial format** | **Top-down 2D**, room-at-a-time camera, octagonal chamfered rooms, snap-to-grid transitions, hand-drawn map overlay | RT-1 |
| **Primary agency** | Mouse + keyboard PC + gamepad for Steam Deck/console + simplified mobile port. Skill axes: routing, parry timing, resource management | RT-2 |
| **Combat pattern** | **Roll-and-Parry** — parry-riposte + dodge-roll fallback (Dead Cells ∩ Sekiro), monsters have Guard + Windowed Opening, 1-HP kept, dice visible as a side tray that multiplies cleave on opening | RT-2, RT-4 |
| **HP model** | 17 discrete hearts, lethal/bursty damage preserved | RT-4 |
| **Info model** | No layout fog + content fog, full transparency of level tables, ink-trail breadcrumb, post-run dashboard | RT-3 |
| **Economy & shop** | Diegetic "Shop Shrine" inside rooms (not a menu pause), single ~30-min run, bestiary/cosmetic metaprogression only, no PvP | RT-5, RT-2 |

### The one conflict to reconcile
RT-0 targeted **~15-minute matches**; RT-5 targeted **~30-minute matches**. I reconcile toward **~20 minutes as the median, 12–35 minute 80th-percentile spread**. Rationale:

- The source game runs ~20–40 min for a careful player. A 15-min RT match would feel like an arcade sprint, losing the "dungeon expedition" weight. A flat 30-min match would feel too long for "just one more run" sessions on mobile or at lunch.
- The beat-based combat cadence (~2–3s per mob, 60–120s for Dracular) plus ~25–60 rooms per run yields ~10–25 min of pure action, plus traversal and shop beats.
- Target: median 20 min; fastest speedrun routes clear in ~11–12 min; patient explorers who farm all 90 cells can push to 35 min but not beyond.

---

## 2. Primary Genre

**Action Roguelite / Dungeon Crawler with Dice-Driven Combat.**

Specifically: a **single-player, single-run, procedurally seeded, top-down action roguelite** in the tradition of *Hades*, *Enter the Gungeon*, and *Dead Cells* — but with a **distinctive dice-pool combat system** that keeps the source's count-successes-on-d6 identity visible on screen at all times. The **routing and the dice tray are the differentiators**; without them, this would be a generic Hades clone.

### Why not "RTS," "ARPG," "Bullet Heaven"?
- **Not RTS:** no base building, no fog-of-war against opponents, no resource gathering in the usual sense, no unit control. Only one player avatar.
- **Not ARPG (Diablo-style):** no stat trees with dozens of points, no loot cascades, no endless scaling. Gear is tiny (3 slots, 6 items). Sessions are short.
- **Not Bullet Heaven (Vampire Survivors):** not auto-attacking with waves; encounters are 1-on-1, player-initiated on room entry, and ended by skilled parries, not attrition.
- **Not Slay the Spire-like deckbuilder:** no deck, no draw mechanic. The "dice pool" is a permanent inventory-defined number, not a shuffled resource.

### Closest to *Hades* for flow and *Into the Breach* for structure
The match arc — choose next room → fight → loot → shop → repeat, with a boss at the end — is Hades's room-to-room structure. The *weight of each decision* and the *deterministic-once-committed* nature of each room is Into the Breach's discipline. Combat itself is closer to Dead Cells / early Hollow Knight: 2D parry-and-punish.

## 3. Dimensional & Visual Choice

**2D top-down, cell-snapped room-at-a-time.** Justification:

1. **2D, not 3D.** The source is 3 pages of hand-drawn grid. 3D would introduce navigation complexity unmoored from the source. 2D preserves the "look at a map" directness. Cost is lower too — this is a one-to-three-person indie scope, not a studio project.
2. **Top-down, not side-scroll or iso.** Top-down preserves the king-move 8-neighbourhood grid literally. Side-scroll would discard 4 of the 8 move directions. Isometric works but adds rendering complexity without benefit for a game where every room is the same size.
3. **Room-at-a-time camera, not whole-dungeon.** The camera frames one room. Doorways and walls close behind the player, enforcing the no-revisit constraint. A toggleable map overlay (V key / middle-click) shows the full 9×10 grid with visited rooms inked in — this is where the roll-and-write tactility lives.
4. **Visual style: hand-drawn ink-on-parchment.** Consistent with the source (a print-and-play PDF) and the style guide already chosen in `GDD.md` and `PrototypePrompts.md`. Wobbly ink lines, cross-hatch shading, handwritten fonts for labels, sans-serif for numbers. Selective color: gold for treasure, blue for potions, red for monsters/danger, green for current cell, grey for visited.

The hand-drawn style is **central to identity**, not a skin. It directly descends from the source medium.

---

## 4. Secondary Tags (4)

- **Parry-Punish Combat** — the core combat verb is "wait for an opening, then strike." Dead Cells's parry, Sekiro's deflect, Hollow Knight's pogo — but with a dice tray side-panel that says "your cleave hit for +2 because you rolled two 6s."
- **Procedural / Single-Run** — every run is a fresh 9×10 seeded dungeon. Deaths reset. No save-scumming.
- **Run-Based Roguelite** — one-life-per-run, pre-roll commitment, identity preserved. Slight metaprogression via cosmetic/bestiary unlocks only — **no** permanent stat buffs between runs.
- **Routing Puzzle** — the no-revisit / must-not-block-End constraint is preserved as a live reachability check that pings the player when they're about to strand themselves. This is the game's secret subgenre, layered underneath the combat.

**Deliberately rejected tags** (and why):
- ~~Co-op~~ — source has no multiplayer; adding co-op would dilute identity and triple scope.
- ~~PvP~~ — the dice-pool feel is non-competitive.
- ~~Base Building~~ — zero base building in source.
- ~~Card-Driven~~ — no deck. Would be a different game entirely.
- ~~Permadeath across runs~~ — runs already reset; no need for "meta-permadeath."

---

## 5. Reference Titles (3) + what each teaches us

| Ref | What it gives us |
|---|---|
| **Hades (Supergiant)** | Room-to-room structure with a pause-for-shop beat between zones. Upgrade pacing: you get ~5–8 upgrade choices per run, each substantial. Boss fight progression: you fight three boss phases with different moves. *Solo Dungeon Dash* borrows the room-flow and the shop cadence. |
| **Dead Cells (Motion Twin)** | 2D parry-and-punish combat with lethal enemies. "One good hit = dead mob" is the core promise. Dead Cells proves that enemies with short HP but good telegraph cycles create satisfying combat. *SDD* borrows the Guard + Windowed Opening system directly. |
| **Into the Breach (Subset Games)** | Deterministic-once-committed spatial planning; every move is a tiny puzzle of routing and resource flow; runs are short (~30 min) and replayable; defeat is a whole-run reset with meta-unlocks. *SDD* borrows the run structure, the commitment weight, and the "pick your next tile from a constrained neighbourhood" feeling. |

## 6. The Unexpected Genre

**Puzzle platformer — "Dungeon Solitaire."**

The alternate framing: forget the combat entirely. Ship the game as a **ritualistic dice-puzzle** where the player methodically walks through the grid, revealing cells, placing dice into slots, and watching dice cascade the way *Threes!* or *Into the Breach* deals cascade. Combat becomes a small minigame (pick dice to attack/defend with), not an action-combat verb. The game becomes meditative, quiet, and drawing-focused.

**Why this genre hides inside SDB:** the source game is fundamentally a *tactile ritual*. The pleasure is not "defeating Dracular" but the act of drawing the path, rolling the dice, and watching the sheet fill. A puzzle-platformer version that leans into that ritual instead of hiding it under action combat would hit the "warm solo ritual" emotional beat perfectly. It would ship faster, require one artist instead of three, and be mobile-native.

**Why it's not the primary genre:** it loses too many players in the modern market, and it loses the "dungeon crawler feel" that the source name promises. We keep it as **Variant B in `GenreVariants.md`** (Stage RT-A1).

---

## 7. Player Experience Statement

> **"I carve my own path through an ink-sketched dungeon. Every room is a gamble. Every fight is a parry-dance with dice that make the damage sing. I will reach Dracular, or I will die trying, and then I will start over — because my last run told me a better path exists."**

- **Core verb:** carve. (Not "fight," not "explore" — *carve*. You are scratching a line into the paper.)
- **Core emotion:** the pleasure of a good route combined with the pressure of a good parry.
- **Core failure mode:** self-inflicted — a misjudged route, a late parry, a greedy push.
- **Core success mode:** clean, intentional runs with no wasted rooms and a boss fight you learned.

---

## 8. The Identity Triangle

```
                ROUTING PUZZLE
                     / \
                    /   \
                   /     \
                  /       \
                 /         \
                /           \
               /             \
  PARRY COMBAT ————————— DICE TRAY
```

Every mechanic in *Solo Dungeon Dash* must serve at least one of these three corners. If a mechanic doesn't serve any, it gets cut.

- **Routing Puzzle** corner = no-revisit, king-adjacency, block-detection warning, pre-seeded contents, per-run procedurally generated dungeons, path-drawn-in-ink visualization.
- **Parry Combat** corner = Guard + Windowed Opening, dodge-roll fallback, no-flee commitment, monster telegraphs, hit-stop feedback, boss phases.
- **Dice Tray** corner = visible d6 pool that represents AD/DD, dice roll on each attack/defence, count-6s mechanic, shop upgrades add dice, 1-HP monsters cleaved by dice cascades.

The **Dice Tray** is the weird one — it's what makes SDD not-just-another-roguelite. Every combat verb flows through the dice tray: the player sees which dice rolled 6s and feels the math happen. This is how we keep faith with the source game's "count-6s-on-d6" identity without asking players to pause combat to roll physical dice.

---

## 9. Reconciliation Log (tensions resolved)

| Tension | RT-0 | RT-5 | Resolved |
|---|---|---|---|
| Match length | ~15 min | ~30 min | **Median 20 min, 80th pct 12–35 min** (compromise favouring slightly shorter than RT-5 to keep mobile viable) |
| Boss fight length | 60–120s | Not specified | **90s median** for Dracular, extendable to 180s on Nightmare difficulty |
| Shop delivery | Not specified | Diegetic shop shrine | **Shop Shrine rooms** (dedicated room type) + safe-room altars |
| Combat identity | Beat-based RT | Parry-riposte + dodge fallback | **Unified as "Roll-and-Parry":** beats = combat rounds, player actions = parry + dodge + attack, dice tray visualizes the math |
| Pre-seeded vs roll-on-entry | Pre-seeded | Not specified | **Pre-seeded per run** — reproducibility wins; the reveal still feels stochastic to the player |

---

## 10. What This Is Not (Anti-Scope)

To defend the scope, here is what *Solo Dungeon Dash* will **not** become:

- **Not an open-world ARPG.** One grid, one dungeon, one session.
- **Not a deck-builder.** Gear is not cards; it's 3 permanent slots.
- **Not a bullet heaven.** Enemies do not swarm. Encounters are 1-on-1 (with rare 2-3-enemy rooms as a variant).
- **Not a tactical simulator.** No unit management, no party members. One player avatar.
- **Not a narrative RPG.** There is no plot. Dracular is the antagonist; that's the whole story. Light flavor text per monster type.
- **Not a live-service.** One-time purchase, no season pass, no battle pass, no microtransactions.

---

## 11. Shipping Hypothesis

If we ship *Solo Dungeon Dash* as described:
- **Primary platform:** PC (Steam) + itch.io. Hand-drawn aesthetic, 2D, small install footprint.
- **Secondary platforms:** Steam Deck (native), mobile (iOS/Android PWA or native).
- **Price point:** $7–$12 one-time. Indie roguelite territory.
- **Team size:** 1 designer/programmer + 1 artist + part-time audio. 3–5 months to shippable alpha, 6–9 months to launch. (See `deployment/Roadmap.md` for the honest version.)
- **Target audience:** roguelite players who liked Hades but want shorter runs; solitaire/solo-board-game players looking for digital; speedrunners who love deterministic seeds; players nostalgic for 2000s print-and-play games.
- **Discoverability pitch:** "Hades meets Into the Breach meets a dungeon master's notebook."

---

## Final Decision Lock

- **Title (working):** *Solo Dungeon Dash*
- **Primary genre:** Action Roguelite (Dungeon Crawler)
- **Dimension:** 2D top-down, room-at-a-time, hand-drawn ink style
- **Secondary tags:** Parry-Punish Combat · Procedural / Single-Run · Routing Puzzle · Run-Based Roguelite
- **References:** Hades · Dead Cells · Into the Breach
- **Unexpected genre (Variant B):** Puzzle platformer / "Dungeon Solitaire" (meditative, dice-cascade)
- **Match length:** median 20 min (12–35 min spread)
- **Boss length:** 90s median
- **Platforms:** PC/Steam primary, Steam Deck/mobile secondary
- **Monetization:** one-time purchase, no MTX

All downstream stages use these locks as given inputs. Any stage that wants to contradict a lock must cite a specific reason why.
