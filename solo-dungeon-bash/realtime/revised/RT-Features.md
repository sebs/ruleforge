# RT-Features — Solo Dungeon Dash

> RealTimeForge Stage RT-D5 — Revised Feature List
> Source: `output/solo-dungeon-bash/Features.csv` (45 features, turn-based) + Wave 1 analysis
> Target: 2D top-down action roguelite, ~20 min runs, PC/Steam primary, mobile secondary

This document is the feature inventory for **Solo Dungeon Dash**, the real-time translation of Solo Dungeon Bash. It exists alongside `RT-Features.csv`, which holds the structured data; this narrative explains scope, ordering, and the cuts/additions relative to the source.

---

## 1. Summary Tables

### Counts by Category

| Category | Count |
|---|---|
| core | 22 |
| combat | 18 |
| content | 21 |
| ui | 11 |
| graphics | 4 |
| audio | 5 |
| ai | 5 |
| onboarding | 4 |
| progression | 6 |
| accessibility | 9 |
| platform | 5 |
| tools | 5 |
| **Total** | **113** |

### Counts by Priority

| Priority | Count | Share |
|---|---|---|
| must | 62 | 55% |
| should | 39 | 35% |
| could | 5 | 4% |
| won't-v1 | 0 | 0% |
| (XL / L flagged) | 7 | 6% of total are L/XL effort |

Note: won't-v1 entries from the source (daily leaderboard backend, procedural dungeon shape variation) are folded into **could** because a small daily-seed mode is a realistic post-launch patch, and dungeon shape variation is explicitly cut for v1.

### Counts by Effort

| Effort | Count |
|---|---|
| S (1–2 days) | 51 |
| M (3–5 days) | 48 |
| L (1+ week) | 12 |
| XL (2+ weeks) | 2 |

Total rough effort estimate: ~400 engineering-days worst case, ~260 best case, with a 1-programmer + 1-artist + 0.5-audio team profile. This aligns with the GenreCrystallization shipping hypothesis of 6–9 months to launch.

---

## 2. MVP Scope (Vertical Test — "Can you parry an Orc?")

**Goal:** Prove the core verb works on one monster in one room with no dungeon, no shop, no meta.

The MVP is a single room loop: the player spawns, an Orc telegraphs a swing, the player parries or dodges, the player swings, the dice tray rolls, the Orc dies (or the player dies). That's it.

**Features in MVP (must-have for any playable build):**

| ID | Feature |
|---|---|
| RT-F001 | Fixed-Timestep Game Loop |
| RT-F002 | 2D Sprite Renderer |
| RT-F003 | Scene / State Machine (minimal: one scene) |
| RT-F004 | Input System (KBM only) |
| RT-F007 | Seeded RNG Service |
| RT-F015 | Room Interior Layout (one hand-built room) |
| RT-F017 | Player Controller |
| RT-F018 | Player Animation Set (idle, walk, parry, swing, hit, death) |
| RT-F019 | Combat FSM |
| RT-F020 | Parry System |
| RT-F021 | Dodge-Roll with I-Frames |
| RT-F023 | Opening Window |
| RT-F024 | Dice Tray Renderer |
| RT-F025 | Dice Roll Animation |
| RT-F026 | Count-6 Damage Resolver |
| RT-F028 | Defence Dice Roll |
| RT-F029 | Heart Pip HUD |
| RT-F030 | Damage Calculation |
| RT-F032 | Hit-Stop & Screen Shake |
| RT-F033 | Enemy AI Base FSM |
| RT-F034 | Monster Telegraph System |
| RT-F035 | Monster Guard Cycle |
| RT-F037 | Monster - Orc (AD1) |
| RT-F079 | Hand-Drawn Sprite Set (player + Orc placeholders) |
| RT-F084 | Combat SFX (minimal) |

**MVP success criterion:** A playtester who has never seen the game can parry an Orc within their first 30 seconds and feel a genuine thrill when the parry ting plays and the dice cascade. If this is not true, the rest of the scope is moot.

**MVP timeline target:** 3–4 weeks with 1 programmer + 1 artist.

---

## 3. Alpha Scope (Vertical Slice — one dungeon level end-to-end)

**Goal:** A fully playable 9-room slice with one biome, two monster types, one shop, one tutorial, and one run result screen. Not shippable — but proves the loop holds for a full run arc.

**Adds on top of MVP (all must-priority):**

| ID | Feature |
|---|---|
| RT-F005 | Save/Load System |
| RT-F006 | Asset Pipeline |
| RT-F008 | Config / Data Loader |
| RT-F010 | Procedural Dungeon Graph |
| RT-F011 | Reachability BFS Checker |
| RT-F012 | Room Content Pre-Seed |
| RT-F013 | Room Transition / Door Seal |
| RT-F014 | Cell-Snapped Movement |
| RT-F016 | Map Overlay (V Toggle) |
| RT-F027 | Cleave System |
| RT-F038 | Monster - Wolf |
| RT-F039 | Monster - Skeleton |
| RT-F054 | Shop Shrine Trigger |
| RT-F055 | Shop UI |
| RT-F056 | Item Slot Exclusivity |
| RT-F057–F062 | All 6 items (Big Sword, Big Axe, Buckler, Shield, Spiky Armour, Magical Armour) |
| RT-F063 | Potion System |
| RT-F064 | Potion Purchase Arming Delay |
| RT-F065 | Treasure Pickup |
| RT-F066 | Chest / Treasure Room |
| RT-F068 | Main Menu (basic) |
| RT-F069 | HUD - Full Layout |
| RT-F070 | Pause Menu |
| RT-F071 | End-of-Run Summary |
| RT-F077 | Tutorial Scenario |
| RT-F083 | Ambient Dungeon SFX |
| RT-F086 | Music - Dungeon Ambient |
| RT-F112 | Build / CI Pipeline |
| RT-F113 | Unit Test Harness |

**Alpha success criterion:** An external tester can complete a full 9-room level, including the shop and a quit-and-resume, without designer assistance. They can explain why they made their routing and build choices.

**Alpha timeline target:** 3–4 months from project start.

---

## 4. Beta Scope (Content Complete)

**Goal:** All content in the game. No new systems. Bug-fix and polish only from here.

**Adds on top of Alpha:**

All remaining must-priority and all should-priority features, specifically:

- **All 11 monsters (RT-F037–F046)** — Evil Warrior, Devil Bat, Cyclops, Dark Elf, Skeleton Lord, Wizard, Demon
- **Dracular boss (RT-F047–F053)** — full 3-phase FSM, all move sets, arena, boss music
- **Monster Pack Variants (RT-F036)** — grouped trash in later levels
- **Block / Posture Meter (RT-F022)**
- **Pity Heart Safety (RT-F031)**
- **Bestiary / Codex (RT-F072)**
- **Settings Menu (RT-F073)**
- **Run History & Stats (RT-F074)**
- **Tutorial Overlay (RT-F075)**
- **First-Time Tooltips (RT-F076)**
- **Help / Reference Screen (RT-F078)**
- **Room Tilesets (RT-F080)** — all biome variants
- **Animation Rigging (RT-F081)** — full skeletal system
- **VFX Library (RT-F082)** — all effects
- **Music Title + Boss (RT-F085, RT-F087)**
- **Light intro/outro cinematics (RT-F088, RT-F089)**
- **Bestiary Unlock (RT-F090)**
- **Achievement System (RT-F091)**
- **Difficulty Modes (RT-F094)**
- **All accessibility features (RT-F096–F104)**
- **Steam Deck Verified (RT-F107)**
- **Mobile Touch Input (RT-F108)** if mobile is in launch scope, otherwise defer

**Beta timeline target:** 6–7 months from project start. Should enter beta ~2 months before launch for playtest cycles.

---

## 5. Launch Scope (1.0)

**Adds on top of Beta:**

- **RT-F105** PC Steam Build (primary) — must-have at launch
- **RT-F106** itch.io Build
- **RT-F107** Steam Deck Verified (completes certification)
- **RT-F108** Mobile Touch Input (if platform strategy allows; otherwise launch PC-only and ship mobile as patch)
- **RT-F110** Telemetry / Analytics (opt-in)
- **RT-F111** Crash Reporter
- Final polish pass on all "must" features
- Localization (EN + 4 languages minimum — German, French, Spanish, Simplified Chinese)

**Post-launch "could" features (1.1 and beyond):**

- **RT-F092** Daily Seed Mode
- **RT-F093** Cosmetic Unlocks
- **RT-F095** Ghost Run Overlay
- **RT-F109** PWA Fallback

**Launch timeline target:** 9 months from project start with the 3-person team profile.

---

## 6. Must-Have Critical Path (ordered build sequence)

The order below is a suggested build sequence that minimises rework. Each step unlocks the next.

1. **RT-F001 Fixed-Timestep Game Loop** — frame budget is the bedrock; every timing feature depends on it.
2. **RT-F002 2D Sprite Renderer** — you need to draw something.
3. **RT-F004 Input System** — get KBM input working before animating anything.
4. **RT-F017 Player Controller** — a moving square on screen.
5. **RT-F018 Player Animation Set** — replace the square with a stickman; iterate sprite later.
6. **RT-F033 Enemy AI Base FSM + RT-F037 Orc** — spawn an Orc that walks at you.
7. **RT-F034 Monster Telegraph System** — Orc winds up.
8. **RT-F019 Combat FSM** — bind swing, parry, dodge.
9. **RT-F020 Parry System** — the hero moment. Tune the window until it *feels* right before anything else.
10. **RT-F021 Dodge-Roll with I-Frames** — second defensive option.
11. **RT-F023 Opening Window + RT-F035 Monster Guard Cycle** — the parry-gate rule.
12. **RT-F024 Dice Tray Renderer + RT-F025 Roll Animation + RT-F026 Count-6 Resolver** — the dice tray is the signature. Do not defer.
13. **RT-F029 Heart Pip HUD + RT-F030 Damage Calculation + RT-F028 Defence Dice Roll** — you can now win and lose.
14. **RT-F032 Hit-Stop & Screen Shake** — juice pass. Do not skip this or the playtests will lie to you.
15. **RT-F010 Procedural Dungeon Graph + RT-F011 Reachability BFS + RT-F012 Room Content Pre-Seed** — the routing corner of the identity triangle comes online.
16. **RT-F013 Room Transition + RT-F014 Cell-Snapped Movement + RT-F016 Map Overlay** — you can now walk a full dungeon.
17. **RT-F054–F056 Shop Shrine + UI + Slot Exclusivity, plus items RT-F057–F062** — the economy loop closes.
18. **RT-F063 Potion System + RT-F064 Arming Delay** — the one real healing verb.
19. **RT-F005 Save/Load** — resume mid-run, persist bestiary and settings.
20. **RT-F047–F052 Dracular Boss** — the climax. This is "beta content complete" for the combat corner.

Once the above 20 are solid, all remaining "must" features (other monsters, HUD polish, menus, tutorial, SFX/music, build pipeline) are parallelizable and do not block each other.

---

## 7. What Was Deliberately Cut From RT-Features (vs source)

The source `Features.csv` has 45 features. Several are explicitly dropped or merged in the RT translation.

### Cut outright

| Source | Reason |
|---|---|
| **F002 Grid Rendering (SVG/Canvas)** | Replaced by 2D sprite renderer; no SVG grid in RT. |
| **F003 King-Adjacency Movement** (as literal click) | Replaced by real-time cell-snap movement (RT-F014) — king-adjacency is preserved only at the room-graph level (RT-F010). |
| **F004 Move Commit & Path Drawing** | Replaced by physical walking. The ink trail becomes a visual on the map overlay (RT-F016), not a per-move commit. |
| **F006 Room Content Roll (on entry)** | Replaced by pre-seeded contents (RT-F012) — Wave 1 economic decision; reveal tension is preserved but dice roll is upstream. |
| **F009 Combat FSM (alternating rounds)** | Replaced by real-time Combat FSM (RT-F019) with parry-gate. The alternating-round cadence is preserved *as beats* not as turns. |
| **F010 Combat Dice Visualization** | Upgraded to dice tray (RT-F024). Same intent, richer execution. |
| **F011 Potion Recovery Phase (step 6)** | Cut as a discrete phase — potions now drink on hotkey in combat (RT-F063). |
| **F015 Potion Purchase Timing (next turn)** | Preserved as RT-F064 Arming Delay (20 s timer instead of "next turn"). |
| **F016 Blocked-Path Detection** | Becomes RT-F011 Reachability BFS Checker — same BFS but runs live. |
| **F018 Turn Sequence Controller (7-step FSM)** | Cut entirely. There are no turns in real time; the seven steps become phases of a continuous room-visit. |
| **F022 Save Current Run (per-turn)** | Becomes RT-F005 Save/Load with per-room checkpoint (not per turn — turns don't exist). |
| **F027 Dice Injection for Tests** | Subsumed under RT-F113 Unit Test Harness. |
| **F029 Confirmation Dialog on End Entry** | Replaced by diegetic hold-to-confirm at the End door (AgencyModel A9); no modal dialog. |
| **F030 Undo Last Move (before roll)** | Cut. In real time, walking out of a room before committing IS undo — no explicit feature needed. |
| **F039 Daily Leaderboard Backend (won't-v1)** | Downgraded to **could** (RT-F092 daily seed) without the leaderboard backend; shareable result string only at launch. |
| **F045 Procedural Dungeon Layout (varying shape)** | Cut. 9×10 is a hard identity lock from GenreCrystallization. Alternate shapes are post-launch DLC territory at most. |

### Not cut, but renamed / restructured

| Source | RT Mapping |
|---|---|
| F001 Grid Model | RT-F010 Procedural Dungeon Graph (9×10 is preserved as room-graph topology, not as a sheet of cells). |
| F005 Level Table Data | RT-F008 Config / Data Loader + data tables. |
| F007 Resource Counters | RT-F029 Heart Pip HUD + RT-F063 Potion System. |
| F008 Room Resolution | Folded into RT-F013 Room Transition + RT-F015 Interior Layout. |
| F012 Shop System | RT-F054 Shrine + RT-F055 Shop UI + RT-F056 Slot Exclusivity. |
| F017 Win/Loss State Machine | Folded into RT-F003 Scene FSM + RT-F071 End-of-Run Summary. |
| F026 Seeded RNG | RT-F007 Seeded RNG Service. |
| F041 Screen-Reader Combat Log | RT-F096. |
| F042 Keyboard Navigation | RT-F097. |
| F043 Color-Blind Palette | RT-F098. |
| F044 Sketch-Style Art | RT-F079 + RT-F080. |

### Multiplayer / live-service explicitly excluded

Neither the source nor RT scope includes multiplayer, PvP, co-op, lobbies, matchmaking, netcode, chat, spectator, trading, guilds, or any form of synchronous player interaction. Per ConflictModel §10, PvP is philosophically incompatible with this game. The netcode category in RT-Features is empty (N/A). Asynchronous social features (ghost runs, daily seed sharing) are the extent of any social layer and remain post-launch *could* items.

---

## 8. What Was Added That Had No Source Analog

The source is a turn-based pen-and-paper solo game. Roughly 40% of RT-Features have no source analog because they are **newly required to translate a dice game into a real-time action game**. This is expected: the Wave 1 analyses all said "you are not porting, you are reinterpreting."

### Real-time engine infrastructure (no source analog)

- **RT-F001 Fixed-Timestep Game Loop** — the source has no loop; a pen does not need a frame budget.
- **RT-F002 2D Sprite Renderer** — the source is paper; there is no renderer.
- **RT-F004 Input System** — the source input is "a pen."
- **RT-F006 Asset Pipeline, RT-F008 Config Loader** — source has no runtime assets.
- **RT-F009 Debug Console, RT-F110–F113 tooling** — no analog in a print-and-play PDF.

### Real-time combat verbs (no source analog)

The source has one combat verb: "roll the dice." Everything below is net-new.

- **RT-F017 Player Controller** — the source hero has no avatar.
- **RT-F018 Player Animation Set** — no animation in a pen game.
- **RT-F019 Combat FSM** — the source has no state machine; it has seven textual steps.
- **RT-F020 Parry System** — this is the RT-native skill layer added in AgencyModel §2.
- **RT-F021 Dodge-Roll with I-Frames** — no dodging in the source at all.
- **RT-F022 Block / Posture Meter** — no blocking verb in the source.
- **RT-F023 Opening Window** — the parry-gate flows from ConflictModel §2; no source equivalent.
- **RT-F024 Dice Tray Renderer** — the dice existed in the source but were physical; the tray is a new diegetic UI.
- **RT-F027 Cleave System** — grouped-trash cleave is Wave 1 §2; source is 1-monster-per-room.
- **RT-F031 Pity Heart Safety** — a Hades-style safety net; source has none (source is brutal).
- **RT-F032 Hit-Stop & Screen Shake** — pure game-feel, no source analog.

### Enemy AI (massively expanded)

The source has zero AI. Every monster in the source is a number (AD). RT needs full behavioural trees.

- **RT-F033 Enemy AI Base FSM** — the meta-system.
- **RT-F034 Monster Telegraph System** — there are no telegraphs in dice rolls.
- **RT-F035 Monster Guard Cycle** — there is no guard in the source.
- **RT-F036 Monster Pack Variants** — the source is strictly 1-per-room.
- **RT-F037–F046** — each monster has AI, animation, tells, and hitboxes entirely invented for RT.
- **RT-F047–F052 Dracular Boss** — the source Dracular is a statline (9 AD / 9 DD). RT Dracular is a three-phase arena boss with a full move set, environmental hazards, and a shadow-clone final phase. Most invented.

### Graphics and audio

The source is an unillustrated PDF with a few hand-drawn grids. Every sprite, every animation, every sound, every piece of music in RT is new.

- **RT-F079 Hand-Drawn Sprite Set** (XL effort — the single biggest cost item)
- **RT-F080 Room Tilesets**
- **RT-F081 Animation Rigging**
- **RT-F082 VFX Library**
- **RT-F083–F087 Audio** — all SFX and music
- **RT-F088, RT-F089 Cinematics** — no cutscenes in source

### UI and onboarding

The source has no UI other than a sheet of paper. All of UI is new.

- **RT-F068–F078** — menus, HUD, overlays, tutorials. The source "tutorial" was a paragraph of explanation text; RT tutorial is a scripted 3-room scenario with seeded dice and overlays.

### Meta and progression

The source has no meta-progression whatsoever. The source stories list a few "could" items (daily seed, achievements, run stats) but no implementation.

- **RT-F090 Bestiary Unlock** — bestiary entries unlock on first kill. New.
- **RT-F091 Achievement System** — Steam-integrated. New.
- **RT-F092 Daily Seed Mode** — share-friendly daily challenge. Source-adjacent but new.
- **RT-F093 Cosmetic Unlocks** — skins only, no stat progression. New.
- **RT-F094 Difficulty Modes** — source had implicit difficulty via RNG only; RT offers named difficulty tiers that modulate tell duration, damage, and boss phase length.
- **RT-F095 Ghost Run Overlay** — asynchronous ghosts (ConflictModel §10 alternative to PvP). Entirely new.

### Accessibility (massively expanded)

The source accessibility story is "a pen and some paper are accessible." In RT, accessibility is a first-class design concern because the RT layer introduces reaction-time requirements that exclude motor-impaired and cognitively-diverse players from the source audience.

- **RT-F096 Screen Reader Combat Log** — audio narration of every dice roll, parry, and hit. Source had no equivalent need; RT must have one.
- **RT-F097 Keyboard Navigation Fallback** — source stories mention this; RT formalizes it.
- **RT-F098 Colorblind Palettes** — source stories mention; RT needs 3 variants (deutan/protan/tritan).
- **RT-F101 Parry Window Scaling** — critical. Without it, the RT game becomes inaccessible to the source's "cool-headed solo" audience described in AgencyModel §1.
- **RT-F102 Slow-Motion Assist** — fully new. Lets cool-headed players play this game.
- **RT-F103 Subtitles & SFX Captions** — fully new; source had no audio.
- **RT-F104 Reduced-Motion Toggle** — fully new.

### Platform

- **RT-F105–F109** — the source "platform" was a printer. RT has a build matrix (Steam/itch/Deck/mobile/PWA).

---

## 9. Scope Discipline and Risk

The biggest scope risks in this feature list are:

1. **RT-F079 Hand-Drawn Sprite Set (XL)** — one artist can easily burn 2–3 months on sprites alone. Mitigation: prioritize MVP assets (player + Orc + Dracular silhouette) and use placeholder art for other monsters until beta.
2. **RT-F047 Dracular Boss FSM (L)** — multi-phase bosses are the most common scope overrun in indie action games. Mitigation: build Phase 1 first, ship Phase 1 only if Phase 2–3 slip; Dracular with one phase is still a climax.
3. **RT-F108 Mobile Touch Input (L)** — mobile is secondary; if it threatens PC launch it ships as a patch.
4. **RT-F095 Ghost Run Overlay (L, could)** — explicitly cut from launch. Needed a backend.
5. **RT-F109 PWA Fallback (L, could)** — explicitly cut from launch; requires web-engine parity testing that is beyond the budget.

**Rule for adding features after this list locks:** any new feature must cite a corner of the **Identity Triangle** (Routing Puzzle / Parry Combat / Dice Tray) it serves. Features that do not serve at least one corner get cut, per GenreCrystallization §8.

---

## 10. Handoff Notes to Downstream Stages

- **RT-D6 (Revised Stories):** user stories should be written against this feature inventory; one story per must-priority feature minimum, plus build-sequence stories for the critical path.
- **RT-D7 (Architecture):** the engine choice (Godot 4 vs Unity 2D vs Phaser) must be made with RT-F001, RT-F024 (dice physics), and RT-F108 (mobile touch) as primary decision factors. Deterministic RNG across platforms is also a blocker for daily seed mode.
- **RT-D8 (Balance):** RT-F020 parry window, RT-F021 dodge i-frames, RT-F030 damage table, and RT-F064 potion arming delay are the four tuning knobs that determine the entire feel of the game. Everything else is secondary.
- **RT-D9 (Assets):** the XL and L graphics/audio items (RT-F079–F089) are the long pole; asset production must begin on day 1 regardless of engineering progress.
- **RT-D10+:** prototype prompts, deployment, etc. consume this list as-is.

The feature list is content-complete for scope planning. Subsequent iterations should only update effort estimates as the project learns actual velocity.
