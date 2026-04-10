# Digital Adaptation Gap Report — Solo Dungeon Bash

Solo Dungeon Bash is a rare "high-fidelity digital fit" case: almost every mechanic can be digitized verbatim. The game has no physical dexterity, no face-to-face negotiation, no hidden hands, no real-time elements, no table talk, and no campaign state between runs.

## Per-Mechanic Assessment

| ID | Mechanic | Rating | Notes |
|---|---|---|---|
| M1 | Dice Rolling (d6 pool, count-sixes) | **No Change Required** | Trivial RNG. Use a seeded PRNG for reproducibility + debugging. Show the dice visually so the player feels the roll. |
| M2 | Grid Exploration / Roll-and-Write | **No Change Required** | The grid becomes a clickable/tappable UI. King-adjacency + unvisited checks are trivial to enforce programmatically. |
| M3 | Resource Management (HP/Treasure/Potions) | **No Change Required** | State counters with automated enforcement of the HP cap. |
| M4 | Random Encounter Table Lookup | **No Change Required** | Hardcoded tables in a data file. Data-driven so it can be modded. |
| M5 | Item Shop / Loadout Tableau | **Simple Adaptation** | Needs UI for slots, exclusivity warnings, cost display, and greying out already-owned items (per A-4 resolution). |
| M6 | Push-Your-Luck | **No Change Required** | Already entirely in the player's head; the digital version inherits it free. |
| M7 | Hand Management (Potion timing) | **Simple Adaptation** | Need to distinguish "available now" potions from "bought this turn / next turn only" potions. UI tag or separate counter. |
| M8 | Character Progression | **No Change Required** | Permanent stat increases. Trivial. |
| M9 | HP Combat Subsystem | **No Change Required** | A deterministic inner loop. Add animation to avoid feeling instant. |
| M10 | Self-Blocking Pathfinding | **Simple Adaptation** | Needs a BFS reachability check on every move. Optionally warn the player *before* they commit to a move that would block them. |

## The Three "Pinch Points"

### Pinch 1: Making randomness feel fair (M1, M4, M9)
A physical game justifies "bad luck" through the tactile ritual of rolling real dice. A digital game has to earn that trust — players *will* believe a digital RNG is rigged when they lose. Mitigations:
- Show the dice visually and individually (no precomputed result flashes).
- Allow the player to see the random seed for the run (optional).
- Log every roll in a visible combat log.
- Consider adding a "Pity" system (e.g., guaranteed 1 hit after N rounds of no 6s) — **optional house rule; flag as tunable.**

### Pinch 2: Replacing the tactile "drawing" satisfaction (M2)
Marking squares by hand is itself pleasurable. In digital form the act of exploration loses that tactility. Mitigations:
- Use a hand-drawn aesthetic (sketch lines, pencil textures).
- Animate the "ink trail" between visited squares.
- Let the player scribble annotations on squares ("came back for treasure later").
- Subtle audio: pen scribble on room entry.

### Pinch 3: Making the blocked-loss feel fair (M10)
Physical players blocking themselves is a "whoops, my fault" moment. A digital game should either:
- Prevent the player from making the blocking move in the first place (warn & confirm).
- Or show a post-hoc "you lost access to End" animation that clearly demonstrates the block.

Preferred: **warn before commit** (since it preserves player agency without feeling cheap).

## Items That Do NOT Require Redesign
Most of the game fits the "No Change" column:
- Turn structure is literal and sequential.
- The shop is a simple menu.
- The monster table is a data lookup.
- Win/lose conditions are boolean checks.
- No multiplayer, so no networking, matchmaking, turn-based notifications, or table-talk handling.

## Additions Digital Enables (Opportunities)
Things the physical game *cannot* do that the digital version *can* (at low cost):

| Opportunity | Effort | Impact |
|---|---|---|
| **Undo last move** (before committing content roll) | Low | Large quality-of-life win |
| **Run history / stats** (deaths per level, average run length, monster bestiary) | Low | Encourages replay |
| **Daily Seed** — everyone plays the same random seed, leaderboard by outcome | Medium | Competitive hook |
| **Procedurally generated dungeon layouts** (different grids, not just 9×10) | Medium | Extends longevity |
| **Difficulty modes** (Easy: +HP, Hard: -HP or smaller grid) | Low | Broader audience |
| **Auto-save & resume** mid-run | Low | Mobile-friendly |
| **Sound/music** atmosphere (dungeon ambiance, combat stings) | Medium | Mood transformation |
| **Animations** for combat rolls, room reveals, boss fight | Medium | Feel upgrade |
| **Achievements** (defeat Dracular under X HP, no potions run, etc.) | Low | Engagement |
| **"No mistakes" mode** — disables the block-yourself loss for new players | Low | Onboarding |

## Accessibility Considerations (Preview — full audit in F24)
- **Visual:** Grid needs high-contrast mode; colour-blind safe palette for Start/End/visited. Dice results should be verbal (screen reader).
- **Motor:** All input can be point-and-click; offer keyboard nav (arrow keys + diagonals via shift+arrow, or hex-number pad).
- **Cognitive:** The game's decision points are sparse and uncomplicated — already low cognitive load. Tooltips should explain monster tables and item exclusivity.
- **Hearing:** No audio is gameplay-critical in the source; maintain that in digital.
- **Communication:** N/A (solo game).

## Redesign-Required Items: **NONE**
This is unusually clean. Nothing about the physical game requires rethinking to work digitally. That makes Solo Dungeon Bash an ideal candidate for a "lightweight web prototype" pipeline — the adaptation is essentially a faithful transliteration plus polish.

## Summary Table
| Category | Count |
|---|---|
| No Change Required | 6 mechanics |
| Simple Adaptation | 3 mechanics |
| Redesign Required | 0 mechanics |
| **Overall adaptation difficulty** | **Low** |
