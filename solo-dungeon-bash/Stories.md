# User Stories — Solo Dungeon Bash

Full data in `Stories.csv`. This document provides the narrative view.

## Scoring System
All scores use Fibonacci values **1, 2, 3, 5, 8, 13**.
- **value** — benefit to the player if the story ships
- **penalty** — pain caused if the story does *not* ship
- **effort** — estimated implementation cost
- **risk** — uncertainty / technical unknowns
- **priority_score** = `value + penalty – effort – risk` (higher = ship first)

## Top 10 by Priority Score
| Rank | ID | Priority | Summary |
|---|---|---|---|
| 1 | S001 | 28 | See the dungeon grid |
| 2 | S002 | 27 | Move king-adjacent |
| 3 | S004 | 27 | Roll for room content |
| 4 | S006 | 26 | Fight monsters in dice duels |
| 5 | S012 | 26 | Warning before blocking myself |
| 6 | S015 | 26 | Enforced 7-step turn order |
| 7 | S007 | 23 | See individual dice results in combat |
| 8 | S009 | 22 | Spend treasure at shop |
| 9 | S013 | 22 | Victory declared on boss kill |
| 10 | S014 | 22 | Defeat declared on death/block |

## Story Distribution by Topic
| Topic | # stories |
|---|---|
| Grid / Movement | 3 |
| Room Content | 2 |
| Combat | 2 |
| Potions / Recovery | 1 |
| Shop | 3 |
| Blocking | 1 |
| Win/Loss | 2 |
| Turn Control | 1 |
| HUD / UI | 2 |
| Save/Resume | 1 |
| Tutorial / Help | 3 |
| RNG / Testability | 1 |
| End-Screen / History | 2 |
| Boss / Undo | 2 |
| Settings / Polish | 2 |
| Accessibility | 3 |
| PWA / Art | 2 |
| Live / Achievements | 3 |
| **Total** | **37** |

## MVP Slice (recommended cutline)
Ship everything with **priority_score ≥ 17**:
- S001, S002, S004, S006, S007, S008, S009, S010, S012, S013, S014, S015, S016, S018, S019, S020, S021, S023

That's 18 stories covering: grid, movement, content rolls, combat (with dice visibility), shop with slot enforcement, blocking warnings, win/loss states, HUD, save, tutorial, tooltips, and RNG foundation. Result: a playable and respectable MVP.

**Out of MVP** (below the 17 cutoff): Bestiary, Undo, Stats, Audio, Settings, Daily Seed, Achievements, Difficulty modes, Accessibility, PWA, Art assets.

**However**, three stories below the cutline should be promoted by product decision:
- **S033 (screen-reader combat log)** — accessibility should be in v1 regardless of score.
- **S034 (keyboard nav)** — ditto.
- **S037 (hand-drawn art)** — the aesthetic is central to the identity.

Promoted MVP: **21 stories**.

## Notes on Scoring Choices
- **S001–S006 (core loop)** have max value + max penalty because without them there's no game.
- **S012 (blocking warning)** scores high because A-1 (ambiguous rule) makes this UX failure-prone without the warning.
- **S015 (turn order enforcement)** is effort-heavy (FSM plumbing) but essential for rule correctness.
- **S023 (seeded RNG)** has high penalty because without it there is no reliable way to test combat — developer pain translates to product pain.
- **S024, S027, S031 (Bestiary, Stats, Achievements)** all score low because they are pure polish — their absence is not painful.

## Acceptance-Criteria Patterns
All stories follow a `GIVEN / WHEN / THEN` structure. Key patterns:
- **State precondition** (GIVEN): the player state or phase that must hold.
- **Trigger** (WHEN): the user action or system event.
- **Outcome** (THEN): the observable state change or UI response.

These are directly usable as test scenarios (e.g. Cypress or Playwright).

## Dependencies
Most stories map 1:1 to a feature in `Features.csv`. Complex stories (S002, S006, S009, S012, S015) span multiple features. See the `feature_id` column in the CSV — semicolons separate multi-feature references.
