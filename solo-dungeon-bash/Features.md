# Feature List — Solo Dungeon Bash

Full data in `Features.csv`. Dependency graph in `FeatureDeps.mmd`.

## Priority Legend
| Priority | Meaning |
|---|---|
| **must** | MVP-critical — the game is not playable without this |
| **should** | Expected by players of a polished game |
| **could** | Nice-to-have; adds depth or longevity |
| **won't-v1** | Deferred to a later release |

## Effort Legend
- **S** = 1–2 days
- **M** = 3–5 days
- **L** = 1+ week

## Summary by Category
| Category | Count | Must | Should | Could |
|---|---|---|---|---|
| core | 22 | 22 | 0 | 0 |
| foundation | 2 | 2 | 0 | 0 |
| onboarding | 3 | 2 | 1 | 0 |
| polish | 6 | 1 | 4 | 1 |
| content | 3 | 0 | 1 | 2 |
| live | 4 | 0 | 0 | 4 |
| accessibility | 3 | 0 | 3 | 0 |
| **Total** | **43** | **27** | **9** | **7** |

## MVP Scope (all Must features)

### Core loop (22)
F001–F022 form a playable game. Implementing every "must" feature produces a working digital Solo Dungeon Bash with save/resume, full rules enforcement, end-game screens, and correct item slot behaviour.

### Onboarding (2)
- F023 Tutorial Flow
- F024 First-Time Tooltips

### Foundation (2)
- F026 Seeded RNG
- F027 Dice Injection for Tests

### Polish (1)
- F029 Confirmation Dialog on End Entry (safety rail for Dracular fight)

## Recommended Post-MVP ("should")
These features elevate the game from "functional prototype" to "shippable product":
- F025 Help Screen
- F030 Undo Last Move
- F031 Run History Log
- F032 Sound Effects
- F034 Settings
- F040 PWA / Offline Play
- F041 Screen-Reader Combat Log
- F042 Keyboard Navigation
- F043 Color-Blind Safe Palette
- F044 Sketch-Style Art Assets

## Could-Have (engagement features)
- F028 Monster Bestiary View
- F033 Background Music
- F035 Daily Seed Challenge
- F036 Achievements
- F037 Run Stats Database
- F038 Difficulty Modes

## Deferred (won't v1)
- F039 Daily Leaderboard (requires backend)
- F045 Procedural Dungeon Layout (deviates from source game)

## Critical Path for MVP
1. **Foundation** — Seeded RNG (F026), Dice Injection (F027)
2. **Data layer** — Level Tables (F005)
3. **State model** — Grid (F001), Resources (F007)
4. **Game logic** — Movement (F003/F004), Content Roll (F006/F008), Combat FSM (F009), Potion Recovery (F011), Shop + Slot Enforcement (F012/F013/F014/F015), Turn Controller (F018), Win/Loss (F017)
5. **UI** — Grid rendering (F002), Player HUD (F019), Dice Vis (F010), End screens (F021), Start screen (F020)
6. **Session** — Save/Resume (F022), Blocked detection (F016), End confirmation (F029)
7. **Onboarding** — Tutorial (F023), Tooltips (F024)

Build order groups features by dependency: data → state → logic → UI → polish.

## Known Gaps / Open Questions
- **Art assets (F044):** Counted as "should" but if the sketch aesthetic is core to the identity, it may need to move to "must" for launch.
- **Accessibility features:** Currently "should" — a live product should promote these to "must" to meet modern accessibility expectations. Flagging for product review.
