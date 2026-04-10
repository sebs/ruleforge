# RT-Stories — Solo Dungeon Dash (Real-Time Revised)

> RealTimeForge Stage RT-D6
> Source: `output/solo-dungeon-bash/Stories.csv` (37 stories)
> Target: real-time action roguelite, 2D top-down, hand-drawn ink-on-parchment
> Working title: **Solo Dungeon Dash**

Full data in `RT-Stories.csv` (55 stories). This document is the narrative view.

---

## 1. Scoring Legend

All scores use Fibonacci values **1, 2, 3, 5, 8, 13**.

| Field | Meaning |
|---|---|
| **value** | Benefit to the player when the story ships |
| **penalty** | Pain caused if the story does *not* ship |
| **effort** | Estimated implementation cost |
| **risk** | Uncertainty / technical unknowns |
| **priority_score** | `value + penalty - effort - risk` (higher ships first) |

Granularity is **Story** for every entry (a few are small enough to be Task-sized but are tracked as stories for pipeline consistency).

A note on the RT version's effort profile: parry combat, boss phases, and ink-trail physics push several high-value stories into the 8–13 effort band. Risk rises correspondingly. This is a more expensive game than the source, but the core verbs carry disproportionate value and penalty — you cannot ship Solo Dungeon Dash without Roll-and-Parry, so it prices accordingly.

---

## 2. Top 10 by Priority Score

| Rank | ID | Priority | Topic | Summary |
|---|---|---|---|---|
| 1 | RT-S024 | 23 | Potions | Hotkey heal mid-combat (Q key) |
| 2 | RT-S034 | 23 | Run Start | Two-click new run, no menu walls |
| 3 | RT-S029 | 22 | Map Overlay | Hold V to toggle full 9x10 map |
| 4 | RT-S005 | 21 | Room Nav | Doors seal behind me on exit |
| 5 | RT-S001 | 19 | Movement | Continuous WASD steering with footfall feel |
| 6 | RT-S004 | 19 | Room Nav | Doors open fluidly when approached |
| 7 | RT-S009 | 19 | Combat | Dodge roll escape hatch |
| 8 | RT-S010 | 19 | Combat | Permanent Dice Tray side panel |
| 9 | RT-S021 | 19 | Shop Shrine | Diegetic shrine beat punctuates the run |
| 10 | RT-S032 | 19 | Reachability | Pulsing warning before strand |

**Observation:** the top 10 is *not* dominated by combat core stories. It is dominated by **feel verbs** (hotkey heal, start-a-run flow, map toggle, dodge, dice tray) and **routing signals** (doors sealing, reachability warning). This is correct for a real-time translation — the source's "see the grid" ranked #1 in the original Stories.csv, but in RT the grid is implicit in room-at-a-time rendering, so the primacy shifts to the moment-to-moment input idioms that define whether the game *plays* well.

The parry story (RT-S007) sits at priority 10, not top 10, because its effort (13) and risk (3) eat its huge value+penalty (26) down. It is still the single most important *feature*, but the scoring system correctly calls out that it is expensive to get right and should be planned with that budget in mind.

---

## 3. Distribution by Topic

| Topic | # stories |
|---|---|
| Movement | 3 |
| Room Navigation | 3 |
| Combat Core | 5 |
| Enemy Feel | 3 |
| Boss Fight | 4 |
| Treasure & Pickups | 2 |
| Shop Shrine | 3 |
| Potions | 3 |
| HUD Feel | 2 |
| Map Overlay | 3 |
| Reachability Warning | 2 |
| Run Start | 1 |
| Run End (Victory) | 1 |
| Run End (Defeat) | 1 |
| Bestiary | 1 |
| Daily Seed | 1 |
| Settings | 1 |
| Accessibility | 3 |
| Platform | 3 |
| Tutorial | 2 |
| Save & Resume | 1 |
| Cosmetics | 1 |
| Input | 2 |
| Graphics | 2 |
| Audio | 2 |
| **Total** | **55** |

The 25 topic buckets from the brief all have at least one story; combat core, boss fight, enemy feel, shop shrine, accessibility, map overlay, and potions have multiple stories because they subdivide naturally into (a) the verb, (b) the feedback, and (c) a policy/edge case.

---

## 4. MVP Slice (priority >= 17)

Ship everything with **priority_score at or above 17**:

- RT-S024 Potions hotkey (23)
- RT-S034 Two-click new run (23)
- RT-S029 Map overlay toggle (22)
- RT-S005 Doors seal on exit (21)
- RT-S001 Continuous steering (19)
- RT-S004 Doors open fluidly (19)
- RT-S009 Dodge roll (19)
- RT-S010 Dice Tray panel (19)
- RT-S021 Shop Shrine beat (19)
- RT-S032 Reachability warning (19)
- RT-S035 Victory screen (18)
- RT-S036 Defeat screen (18)
- RT-S011 Damage feedback (17)
- RT-S018 Dracular signature moves (17)
- RT-S027 HUD readable (17)
- RT-S030 Map current position (17)
- RT-S041 Accessibility slow-mo (17)

**MVP count: 17 stories.**

By product decision, the following stories must be **promoted above the cutline** despite lower scores:

- **RT-S007 Parry + defence dice** (priority 10, effort 13) — this is the central combat verb; the game does not exist without it. Effort is high, which is precisely why it must be scheduled first and protected from scope creep.
- **RT-S008 Post-parry attack window** (priority 15) — Parry without a conversion is half a mechanic. Ships with RT-S007.
- **RT-S015 Dracular nine phases** (priority 8, effort 13, risk 5) — the boss is the run's climax. Scored low only because of cost/risk; cannot be cut.
- **RT-S052 Hand-drawn ink style** (priority 10) — identity anchor; this is the game's look.
- **RT-S054 Signature combat audio** (priority 14) — parry *ding* is central to the combat feel.
- **RT-S046 Parry tutorial** (priority 14) — new players cannot learn Roll-and-Parry cold.

**Promoted MVP: 23 stories.** This is the shippable alpha slice.

**Out of MVP (below 17, not promoted):** Bestiary (RT-S037), Cosmetics (RT-S049), Daily Seed (RT-S038), Audio ambient (RT-S055), Settings (RT-S039), remote coop / live features (none present). These are v1.1 polish or v2 work.

---

## 5. Notable Scoring Choices

### Why RT-S024 (potion hotkey) tops the board

In the source, potions were a between-round decision with zero risk. In the real-time translation, drinking mid-combat is the single mechanic that resolves source ambiguity F-2 and prevents the "1 HP cliff" deaths that combat chain analysis flagged as a dominant failure mode. It has maximum value, maximum penalty (without it the game is brutally unfair), trivial effort (a keybind and an animation), and near-zero risk. This is the rare 23-priority story that is also the smallest codebase change.

### Why RT-S007 (parry core) scores "only" 10

RT-S007 has 13/13 for value and penalty — the game has no combat without it. But effort is 13 (parry window tuning, stagger math, visual flash, dice tray integration, telegraph detection, i-frame state machine) and risk is 3 (input latency budget, playtest-driven window duration, Steam Deck touchscreen handling). `26 - 16 = 10`. This is **correctly priced**: parry is not a cheap feature, and the team should budget for it accordingly. Low priority number, highest strategic priority.

### Why RT-S034 (run start) is tied for #1

Session ritual matters enormously for a ~20-minute roguelite. A player who clicks once to launch and once to start a run will run twice per session. A player who wades through tutorials and modals will run once. This story is basically "do not add friction" — trivial effort, low risk, huge engagement impact. It ties with the potion hotkey because both are maximum-value, minimum-cost stories.

### Why the Bestiary (RT-S037) scores 4

Same reason it scored 7 in the source: nobody is going to stop playing because there is no bestiary, and its absence does not degrade the core loop. It is pure collection polish. Ship it in a free post-launch patch.

### Why RT-S015 (Dracular nine phases) scores 8

Value 13, penalty 13 (the game has no ending without it) but effort 13 and risk 5. This is a big feature: nine phase state machine, nine distinct animations, banking logic, ring VFX, audio beats, final-phase rhythm reader, death animation. Five risk points because we have not yet prototyped whether a 90-second rhythm boss feels right or feels exhausting. This is **the** feature to playtest early.

### Why RT-S044 (mobile touch) scores only 6

Genre Crystallization locked mobile as a *port target, not a design target*. Touch parry windows are too slow for the intended skill ceiling; the story must ship with the auto-parry assist, which is itself another story (RT-S041). Effort is real (5) and risk is real (2), while value/penalty are limited because the primary audience is on PC/Steam Deck. We ship mobile because we can, not because we must.

### Why accessibility is spread across three stories, not one

Accessibility work is not a single story — it is (at minimum) screen-reader narration (RT-S040), motor/timing assists (RT-S041), and keyboard-only navigation (RT-S042). Each has distinct effort and audience. Treating them as one story would hide the 5-effort screen-reader work inside a 3-effort bucket and miss it in sprint planning.

---

## 6. What Was Removed From Source Stories

The source Stories.csv has 37 stories built around a turn-based grid. The following patterns either no longer apply or have been transformed:

### Removed because the mechanic no longer exists

- **S002 "Move into king-adjacent squares" (hover/click)** — replaced by continuous WASD steering (RT-S001). The king-adjacency constraint is now implicit in which walls have doors.
- **S003 "Illegal squares are unclickable"** — there are no clicks. Walls simply block the hero.
- **S004 "Roll d6 for room contents on entry"** — the d6 is now auto-resolved as you cross the threshold. Pre-seeded per run; the player never sees the roll UI.
- **S015 "Enforce 7-step turn order"** — there is no turn FSM in the real-time version. Phases are collapsed into continuous gameplay with the Shop Shrine as the only explicit beat.
- **S026 "Undo last move before content roll"** — no move-then-roll gap exists. Undo is replaced by the Dodge mechanic (RT-S009) and the Reachability Warning (RT-S032).
- **S011 "Bought potions usable next turn"** — bought potions are usable immediately via the hotkey. The "pending vs available" distinction is gone.

### Transformed from passive to active

- **S012 "Warn before self-blocking"** — the source story had a passive warning dialog. RT-S032 upgrades this to an **active pulsing icon plus hold-to-confirm**, surfacing the BFS result continuously rather than only at click time.
- **S025 "Boss entry confirmation dialog"** — replaced by the **commit-then-reveal** hold-to-confirm on the End door (rolled into RT-S018 and RT-S004). No modal dialog.
- **S006/S007 "Manual dice rolling in combat"** — the dice still roll, but on a permanent side-panel Dice Tray (RT-S010). The player never clicks "roll" — the Tray rolls when you parry (defence dice) or attack (attack dice). Manual dice rolling is gone; visible dice math is preserved.

### Transformed from menu beat to diegetic beat

- **S009 "Open shop at end of turn"** — the menu becomes a diegetic **Shop Shrine** (RT-S021), framed as a world object, not a pause menu.
- **S018 "Run summary screen"** — split into victory and defeat variants (RT-S035, RT-S036) because the RT version treats them as distinct emotional moments with different celebrations and restart flows.

### Rule deviation (documented)

- **Potions usable any time** — the source rule was ambiguous (F-2). The RT version **canonicalises potions as usable any time, including mid-combat** (RT-S024). This is tracked as a rule deviation and surfaces in RevisedRules.md.

---

## 7. What Was Added That Had No Source Analog

The source game is turn-based with no dexterity layer. Everything below is RT-native and has no story in the source CSV.

### Combat verbs (new)

- **RT-S007 Parry** — hold-to-roll-defence-dice timing window (220ms). The entire Roll-and-Parry mechanic.
- **RT-S008 Post-parry attack window** — the conversion: parry opens a 600ms kill window.
- **RT-S009 Dodge roll** — stamina-gated escape from a telegraph you cannot read.
- **RT-S010 Dice Tray** — permanent side panel visualizing attack and defence dice pools.
- **RT-S011 Damage feedback** — heart-pip drain with ink splash and screen kick.

### Boss as rhythm fight (new)

- **RT-S015 Nine Dice Duel** — nine-phase structure with banked dice.
- **RT-S016 Hunger phase parry heal** — mid-fight comeback moment.
- **RT-S017 Unbound Rage final phase** — rhythm-reader execution test.
- **RT-S018 Dracular signature moves** — named, color-coded telegraphs.

### Enemy feel (new)

- **RT-S012 Per-enemy telegraphs** — each monster type has a unique tell.
- **RT-S013 Hit-stop on kill** — the 1-HP monster death gets 80–120ms of screen freeze.
- **RT-S014 Visual variety** — at least 8 distinct monster silhouettes.

### Movement feel (new)

- **RT-S001 Continuous steering** — WASD/stick with footfall audio and ink trail.
- **RT-S002 Cell-snap room transitions** — camera pan + doorway seal at thresholds.
- **RT-S003 Diagonal movement feel** — no stutter at tile edges.

### Room navigation (new)

- **RT-S004 Doors open on approach** — fluid, no click-to-interact friction.
- **RT-S005 Doors seal behind** — the no-revisit rule is physical, not a rule prompt.
- **RT-S006 Door wall readability** — walls with doors look different from walls without.

### Pickups (new)

- **RT-S019 Auto-pickup magnet** — walk-over collection with counter bump.
- **RT-S020 Coin shower particle** — treasure-room juice.

### Potions as a verb (new)

- **RT-S024 Potion hotkey** — the headline new mechanic.
- **RT-S025 Heart refill animation** — visible feedback on heal.
- **RT-S026 Cap enforcement** — soft rejection at 17 HP.

### Map as a beat (new)

- **RT-S029 Hold-V overlay** — world freeze, full grid, plan a route.
- **RT-S030 Current position pulse** — orientation aid.
- **RT-S031 Ink trail consistency** — live view and map show the same path.

### Reachability (upgraded from passive to active)

- **RT-S032 Pulsing warning + hold-to-confirm** — active, not modal.
- **RT-S033 Expert-mode disable** — settings toggle for no warnings.

### Touch and accessibility assists (new)

- **RT-S041 Slow-motion + auto-parry assist** — the skill-floor ramp.
- **RT-S044 Mobile touch layer** — virtual stick, parry button, auto-parry default.

### Ink aesthetic as a first-class feature (new)

- **RT-S052 Hand-drawn consistent style** — identity anchor.
- **RT-S053 Physical ink trail** — the kinaesthetic soul translation from Stage RT-2 §10 Issue 1.

### Audio verbs (new)

- **RT-S054 Signature combat sounds** — parry *ding*, hit crunch, dice clatter.
- **RT-S055 Room-type motifs** — ambient cues on room reveal.

### Platform (expanded)

- **RT-S043 Steam Deck verified** — was not a concern in source.
- **RT-S044 Mobile PWA touch controls** — source S036 was PWA-only; RT adds touch.
- **RT-S050 Gamepad remap** — source had none.
- **RT-S051 Hotkey placement** — source was mouse-only.

---

## Acceptance-Criteria Patterns

All stories follow `GIVEN / WHEN / THEN`. New RT-specific patterns:

- **Timing windows** — many criteria specify millisecond thresholds (parry 220ms, hit-stop 80ms, dodge i-frames, hold-to-confirm 0.8s). These are directly measurable in playtest.
- **Visual feedback beats** — criteria explicitly call for specific animation durations (250ms heal swirl, 300ms tray transition, 500ms door seal) so the feel work is testable.
- **Input idioms** — verbs are named by key (Q for potion, V for map, Space for parry) to make the input story concrete and remappable.
- **Stage integration cues** — several stories reference artifacts from earlier waves (Dice Tray from RT-2, Shop Shrine from RT-4, ink trail from RT-2 §10).

These criteria are directly usable as automated playtest probes and as QA acceptance checklists.

---

## Handoff Notes

- `RT-Stories.csv` is the source of truth for sprint planning and can be imported directly into the pipeline's stakeholder export.
- The **promoted MVP of 23 stories** is the shippable alpha slice. Any story not in this list can be deferred to v1.1 without killing the core experience.
- The six promoted stories below the 17 cutline (RT-S007, RT-S008, RT-S015, RT-S046, RT-S052, RT-S054) should be tagged `promoted=true` in any downstream planning tool — their scores are deceptively low because of effort, not importance.
- Downstream stages (RT-D7 Architecture, RT-D8 Balance, RT-D9 Assets) should treat the MVP slice as their scope anchor.

*End of RT-Stories.md — Stage RT-D6 complete.*
