# RT-Architecture — Solo Dungeon Dash

> RealTimeForge Stage RT-D7: Revised Architecture
> Source: `output/solo-dungeon-bash/Architecture.md`, `Architecture.mmd`, and the Wave 1 analyses (`GenreCrystallization.md`, `TemporalMap.md`, `SpatialModel.md`, `ConflictModel.md`).
> Target: 2D top-down action roguelite, 60fps, PC / Steam Deck / mobile, single-player.
> Sibling file: `RT-Architecture.mmd` (system diagram).

---

## 0. Ground Facts Restated

These are locks from upstream stages; the architecture must honour them and is not free to revisit them:

- **Title (working):** Solo Dungeon Dash
- **Genre:** Action Roguelite / Dungeon Crawler, beat-based parry combat
- **Dimension:** 2D top-down, room-at-a-time, hand-drawn ink-on-parchment
- **Dungeon:** 9x10 cell grid + Start + End protrusions, pre-seeded content, king-adjacency, no-revisit
- **Combat identity:** Roll-and-Parry with a visible Dice Tray — every attack and defence still counts sixes on d6s, but the dice roll is triggered by real-time parry/swing events
- **HP model:** 17 discrete hearts (player), 1 HP per monster with Guard + Windowed Opening, Dracular = 9 DD defensive wall
- **Match length:** median 20 minutes (12-35 min spread)
- **Platforms:** PC (Steam + itch) primary, Steam Deck verified, mobile secondary
- **Concurrency:** single-player, no netcode, no co-op, no PvP
- **Monetization:** one-time purchase, no MTX

Everything that follows exists to serve these locks.

---

## 1. Engine Choice + Justification

**Picked engine: Godot 4.3 (C# + GDScript hybrid).**

### Trade-off table

| Criterion | Godot 4 | Unity 2D | Phaser TS |
|---|---|---|---|
| **Licence** | MIT, fully free, no runtime fee | Personal free up to $200k; enterprise above. Runtime-fee history (2023) is a trust hazard | MIT, fully free |
| **Engine size / install footprint** | ~90 MB editor, ~40 MB export template, export binary ~35-70 MB | ~3 GB editor, ~25-50 MB slim build | Library is small (~1 MB) but needs a browser runtime; native wrap via Electron/Tauri bloats |
| **2D tooling** | First-class 2D: dedicated 2D renderer, TileMap, Area2D, CanvasLayer, shader language tuned to both | 2D is a layer over the 3D renderer; serviceable but second-class | Literal 2D canvas, no native viewport, no physics beyond Arcade / Matter |
| **Hand-drawn 2D workflow** | Direct import of PNG / Aseprite, TextureAtlas, shader-based ink wobble, AnimationPlayer | Excellent with 2D Pixel Perfect and Cinemachine 2D, but overkill here | Fine for flat sprites; limited shader support without WebGL plumbing |
| **State machine tooling** | Built-in `AnimationTree` state machine + GDScript / C# classes | Unity Animator Controller, Bolt / Visual Scripting, Behaviour Designer | None built-in, roll-your-own FSM |
| **Save system** | Built-in `user://` path, `FileAccess`, `ConfigFile`, `Resource` save/load | `PlayerPrefs` (trivia only) + file I/O; ScriptableObjects for config | `localStorage` / IndexedDB; fragile across platform wraps |
| **PC export** | One-click Windows / Linux / macOS, tiny binaries | One-click but larger binaries | Requires Electron, Tauri, or NW.js |
| **Steam Deck export** | Linux native export runs natively; Proton also works | Linux native export is supported, Proton is backup | Chromium wrap runs but input latency concerns |
| **Mobile export** | iOS and Android officially supported, modest effort | iOS and Android flagship; Unity is the de facto mobile engine | Cordova / Capacitor wraps; touch input is fine, 60fps harder |
| **Input abstraction** | `InputMap` handles gamepad + keyboard + touch uniformly | `Input System` package, more verbose, best-in-class | Phaser input plugin, three code paths for kb/gamepad/touch |
| **Community & docs** | Growing fast, official docs solid, smaller asset store | Huge community, huge asset store, enterprise support | Strong web-game community, moderate resource size |
| **Source control** | Plain text scenes (`.tscn`), diff friendly | Binary scenes by default, `YAML merge` required | Plain text, trivial to diff |
| **Determinism** | Deterministic fixed-step `_physics_process`, integer-safe math via C#/GDScript; easy to gate all RNG | Deterministic if you force `Time.fixedDeltaTime`, but Unity's `Random` needs isolation | Fully deterministic if you own the loop; Phaser's physics can drift |
| **Risk** | Some Godot 4.x APIs still maturing (save/load Resource edge cases), smaller commercial track record | Runtime-fee trust debt; heavier for a 2D indie | Not a game engine proper; we re-invent scene mgmt, physics, audio routing |

### Why Godot 4 wins for Solo Dungeon Dash specifically

1. **The scope is tiny and 2D-only.** One player avatar, 1-6 enemies on screen, a 9x10 grid, a dice tray overlay, and an ink-parchment shader. Unity is optimised for games far larger than this; paying for its weight is wasteful. Phaser TS is optimised for web distribution, which is only our tertiary channel.
2. **Hand-drawn ink-on-parchment is a shader job.** Godot's shader language (`gdshader`) ships a native 2D CanvasItem shader that maps directly onto this aesthetic. We need wobble, paper grain, and ink spread effects — Godot handles these cleanly via fragment shaders on CanvasItem. Unity can match this but with more ceremony; Phaser TS cannot without writing raw WebGL.
3. **Steam Deck support is decisive.** The genre lock mandates Steam Deck verification. Godot exports a Linux native binary that runs on the Deck without Proton translation, preserving input latency. Unity's Linux exports work but sometimes hit the Steam Deck graphics driver quirks (vkBasalt, DXVK mismatches). Phaser in a wrap runs on the Deck's browser layer but has historically bad touchpad behaviour.
4. **Mobile is secondary, not primary.** We need mobile to work, not to be flagship. Godot's mobile export is good enough for a game that runs 60fps on a 15-year-old PC. We are not pushing the hardware.
5. **Licensing and footprint matter to indie budget.** Godot is MIT with zero fees. Unity's 2023 runtime-fee episode created real industry trust damage and made every future Unity commitment load-bearing on Unity Inc's goodwill. For a one-time-purchase indie title with small revenue expectations, the Godot licence is the rational choice.
6. **Text-based scenes support a small team's git workflow.** Godot's `.tscn` format is plain text and diffs cleanly. Unity's scene merges are historically painful. Phaser has no scene concept at all.
7. **Combat FSM tooling is native.** Godot ships an `AnimationTree` + `AnimationNodeStateMachine` that pairs naturally with the Roll-and-Parry combat cadence (telegraph -> strike -> parry window -> opening). We can author the monster state machine as an AnimationTree and layer our logic FSM on top via a small C# class. Unity would need the Animator Controller + a behaviour layer; Phaser would need a hand-written FSM library.

### Defending the pick against the others

- **Why not Unity 2D?** Strictly heavier than we need, licensing trust debt, scene-merge cost for a small git-based team, and the 2D renderer is a layer over the 3D pipeline (wasted memory). Unity would be the right pick for a 3D game or a team of 10+; we are neither.
- **Why not Phaser TS?** Phaser is a web-first rendering library, not a game engine. We'd re-implement scene management, audio routing, fixed-timestep physics, input abstraction, and save serialisation ourselves. Native / Steam Deck distribution requires an Electron/Tauri wrap that bloats the download and imposes Chromium-sized RAM overhead. We lose the "60fps on every target platform" guarantee.
- **Why not GameMaker, Love2D, Bevy, MonoGame?** Each is viable but smaller ecosystem / harder onboarding. Godot's footprint-to-capability ratio is the sweet spot for a 1-3-person indie team making a 2D roguelite.

**Final lock: Godot 4.3.** Language: C# for performance-critical systems (CombatFSM, DiceTray, Reachability BFS), GDScript for glue code, UI, and tutorial scripting.

---

## 2. Architecture Pattern

**Picked pattern: Hybrid — OOP scene tree for presentation + components/services for logic, with a pure-functional rules core.**

### Why not full ECS

ECS (Entity Component System) shines when you have:
- Hundreds to thousands of entities simultaneously updating,
- Highly parallelisable transforms (physics, particle systems, AI flocks),
- A need for cache-friendly contiguous data.

Solo Dungeon Dash has none of those. At any given moment the scene contains:
- 1 player avatar,
- 0-6 enemies (rare rooms only push to 3),
- ~10-30 particles during combat peaks, briefly up to ~200 during boss phases,
- A dice tray showing up to 10 dice physics bodies.

Peak simultaneous entities: ~220 including VFX. That is well inside the range where any architecture performs fine. ECS would buy us nothing and cost us clarity.

### Why not pure OOP everywhere

A pure OOP scene-graph would bury rules inside `MonoBehaviour`-style lifecycle methods. The source game already proved that the **rules engine should be pure functional**, because that's what made 100% unit-testable determinism achievable (see `Architecture.md` section 3). We keep that discipline in the real-time port.

### The hybrid layout

| Layer | Pattern | Language | Rationale |
|---|---|---|---|
| **Presentation** (sprites, camera, VFX, HUD, menus) | Godot Node-tree (OOP) | GDScript | Godot's native idiom; editor-authorable |
| **Game Logic** (FSMs, managers, controllers) | Services + FSMs | C# | Testable, fast, easy to mock |
| **Rules / Math** (dice rolls, damage calc, reachability BFS, legal moves) | Pure functions on immutable state | C# static methods | Identical discipline to the board game port; deterministic replay; easy unit tests |
| **Data** (level tables, monster defs, items) | Godot `Resource` + JSON | GDScript-safe | Editor-authorable by designer; hot-reloadable |

This mirrors Clean Architecture: the rules core has zero dependencies on Godot types. The game logic layer consumes the rules core and is consumed by the presentation layer. The presentation layer can be rebuilt without touching rules.

### Concrete shape

- `GameState` is a plain C# record: immutable, serialisable, the single source of truth for a run.
- `RulesEngine` is a static class of pure functions: `RulesEngine.Move(state, target, rng)`, `RulesEngine.RollAttack(state, rng)`, `RulesEngine.BuyItem(state, itemId)`, etc.
- `CombatFSM` is a C# class holding the current beat state (`Telegraph`, `StrikeFrame`, `ParryWindow`, `Opening`, `Recover`) and driving transitions based on time and input events.
- `Player`, `Enemy`, `DiceTray`, `Room` are Godot Node classes that read from `GameState` and dispatch events back.

---

## 3. Core Loop Architecture

### Tick model

We run two interleaved loops, as is standard for 2D action games:

- **Fixed-timestep logic loop** at 60 Hz (16.67ms per step) for: input capture, player state, enemy AI, combat FSM, physics, reachability checks.
- **Variable-timestep render loop** at monitor refresh rate (60 / 120 / 144 / 165 Hz) for: sprite interpolation, VFX, camera smoothing, audio scheduling.

The logic loop is bound to Godot's `_physics_process(delta)`, which Godot clocks at a fixed rate by default. The render loop is `_process(delta)`.

### Target frame rates

- **Logic:** 60 Hz fixed, non-negotiable.
- **Render PC:** 60 Hz default, 120 Hz optional for high-refresh monitors. Player sprite interpolates between logic frames for smoothness above 60 Hz.
- **Render Steam Deck:** 60 Hz (the Deck's panel is 60 Hz, and we target the handheld's battery life rather than pushing to 90 Hz even on hw-capable revisions).
- **Render mobile:** 60 Hz where battery allows; adaptive 30 Hz fallback on older devices.

### Update order per tick

This is the authoritative order. Any system that reads from another must come after it in the order:

1. **Input Poll** — drain the input event queue, build the `InputFrame` record (WASD axis, parry button, dodge button, potion button, swing held/pressed).
2. **Time System tick** — advance `gameTime`, `combatBeatTime`, `pauseFlag`.
3. **Scene / State Manager tick** — advance menu transitions, handle save/load side effects.
4. **Player Controller update** — apply the `InputFrame`, move the player, update `PlayerState`.
5. **Enemy AI update** — each enemy advances its behaviour state machine (telegraph / attack / guard / opening / recover).
6. **Combat FSM tick** — process combat beat transitions, schedule parry windows, resolve strikes. Drives the DiceTray when a roll is due.
7. **Dice Tray animator tick** — advance dice tumble physics, schedule `RollFinished` event when the tumble is complete.
8. **Physics / Collision step** — resolve swing-arc hurtbox overlaps, player-enemy contact, pickup triggers.
9. **Reachability check** (conditional) — only runs when the player commits to a new cell; BFS over the 9x10 grid.
10. **Economy / Inventory update** — apply pickups, potion usage, shop purchases.
11. **Camera update** — smooth-follow to the current room framing, apply shake if requested.
12. **Animation update** — advance sprite state machines, drive skeletal poses.
13. **HUD / UI update** — sync hearts, dice tray, potion stack, ink trail to current `GameState`.
14. **Render** — draw everything.
15. **Audio tick** — schedule any SFX queued this frame, update music layers.
16. **Save hook** — if the current frame crosses a room boundary and the player has just committed to a new cell, enqueue an auto-save (executed asynchronously on a worker thread so the main loop does not stall).

In Godot terms, steps 1-10 live inside `_physics_process`, steps 11-16 live inside `_process`. The split is deliberate: logic is deterministic and fixed, presentation is smooth and interpolated.

---

## 4. Systems Breakdown

Each entry lists: responsibility, update frequency, dependencies, key data.

### 4.1 Input System
- **Responsibility:** Normalise keyboard / gamepad / touch into an `InputFrame` struct per tick. Maintain `InputMap` rebinding.
- **Update frequency:** Every logic tick (60 Hz). Edge events (button pressed this frame) are latched.
- **Dependencies:** Godot `Input` singleton; platform-specific touch layer on mobile.
- **Key data:** `InputFrame { moveAxis: Vec2, parryEdge: bool, dodgeEdge: bool, swingHeld: bool, swingEdge: bool, potionEdge: bool, mapToggleEdge: bool, pauseEdge: bool }`.

### 4.2 Time System
- **Responsibility:** Own `gameTime`, `combatBeatTime`, `timeScale`, `isPaused`. Provide `dt` and `fixedDt`. Support slow-mo on hit confirms.
- **Update frequency:** Every tick.
- **Dependencies:** None; it is the root of the update graph.
- **Key data:** `TimeState { gameTime: double, combatBeatTime: double, timeScale: float, isPaused: bool, framesSinceLaunch: long }`.

### 4.3 Scene / State Manager
- **Responsibility:** Own the top-level state (`MainMenu`, `Run`, `Paused`, `Summary`, `Settings`, `Bestiary`). Handle transitions. Trigger save/load.
- **Update frequency:** Every tick (state-machine transition checks are O(1)).
- **Dependencies:** SaveService, RoomManager, HUD.
- **Key data:** `SceneState` enum + transition history for debug.

### 4.4 Dungeon Generator
- **Responsibility:** From a seed, produce a 9x10 `Grid` with per-cell content rolled against the level tables. Runs once per run at `RunStart`. Deterministic.
- **Update frequency:** Once per run.
- **Dependencies:** SeedPRNG, LevelTables, MonsterDefs, BalanceCfg.
- **Key data:** `Grid { cells: Cell[9, 10], start: Coord, end: Coord }`, where `Cell { row, col, contentType, monsterId?, itemId?, revealed }`.

### 4.5 Room Manager
- **Responsibility:** Track current room, orchestrate room transitions (seal behind, open ahead), manage door walls during combat lock. Fire `RoomEntered`, `RoomCleared`, `RoomExited` events.
- **Update frequency:** Every tick for seal animation; event-driven for transitions.
- **Dependencies:** DungeonGenerator (grid data), SceneManager, VFX, Camera, Audio, SaveService, AchievementTracker.
- **Key data:** `currentCoord: Coord`, `doorStates: Map<Dir, DoorState>`, `roomLocked: bool`.

### 4.6 Reachability Checker
- **Responsibility:** After every committed move, run BFS over unvisited king-adjacent cells from the new current cell. If the End cell is no longer reachable, fire `RunLost(Blocked)`.
- **Update frequency:** On demand, once per committed move (roughly every 1-3 seconds of gameplay). Maintains a 10 Hz "about to block yourself" warning light on the HUD.
- **Dependencies:** Grid, HUD.
- **Key data:** `bool[9, 10]` reachable mask. Cached from the last BFS run.
- **Perf:** O(90) per call. Cost is ~4 microseconds on any modern CPU. Zero risk.

### 4.7 Player Controller
- **Responsibility:** Convert `InputFrame` into player movement and combat actions. Manage cell-snap locomotion (~0.4s per cell walking, ~0.25s hustle), cancel window, swing intent, parry intent, dodge intent, potion-drink action.
- **Update frequency:** Every tick.
- **Dependencies:** InputSystem, Grid, RoomManager (for commit check), CombatFSM, Economy, BalanceCfg.
- **Key data:** `PlayerState { pos: Vec2, cellCoord: Coord, velocity: Vec2, actionState: PlayerActionState, swingCharge: float, drinkCooldown: float }`.

### 4.8 Enemy AI Controller
- **Responsibility:** Drive one enemy per room (up to 3 in rare rooms, up to 6 in the boss arena's ad-spawn phase). Each enemy runs the cycle: `Idle -> Telegraph -> StrikeFrame -> Recover -> Guard -> Opening -> Guard -> Telegraph`. Boss uses a rotated pattern stack (Fast Jab / Heavy Slam / AoE Spin).
- **Update frequency:** Every tick.
- **Dependencies:** CombatFSM, PlayerState (for facing), MonsterDefs, SeedPRNG, BalanceCfg.
- **Key data:** `EnemyState { monsterId, hp (always 1 except boss), beatState, beatTimer, currentPattern, patternStack (boss only) }`.

### 4.9 Combat System (Roll-and-Parry FSM)
- **Responsibility:** Top-level beat sequencer for an encounter. Holds phase `BeatStart -> Telegraph -> StrikeFrame (parry window) -> StrikeResolve -> Opening -> SwingResolve -> Recover -> BeatStart`. Mediates between EnemyAI and PlayerController. Calls DiceTray for rolls. Applies damage to hearts.
- **Update frequency:** Every tick when a combat encounter is active.
- **Dependencies:** DiceTray, PlayerController, EnemyAI, RulesEngine, Audio, VFX, HUD, BalanceCfg.
- **Key data:** `CombatState { encounterId, beatPhase, beatTimer, lastRollResult, openingActive, pendingHits }`.

### 4.10 Dice Tray System
- **Responsibility:** Own the visible dice pool. On `RequestRoll(count, streamId)` from CombatFSM, spawn `count` physics dice, tumble them for ~500ms, sample the SeedPRNG sub-stream `streamId`, snap final faces, count 6s, fire `RollFinished(hits)`.
- **Update frequency:** Every tick while a roll is in flight.
- **Dependencies:** SeedPRNG, VFXSystem, Audio, CombatFSM.
- **Key data:** `DiceTrayState { dicePool: Die[], rollInFlight: bool, rollStartTime: double, rolledFaces: int[], hitsCount: int }`. The DiceTray has both a *logical* result (deterministic from the PRNG) and a *visual* tumble (purely presentational). The visual tumble MUST land on the pre-computed logical faces — we cheat the physics by nudging the final pose to match the already-decided result.

### 4.11 Economy System
- **Responsibility:** Track treasure count, potion stockpile, inventory slots (weapon / shield / armour). Apply pickups, apply potion usage, apply shop purchases.
- **Update frequency:** Event-driven (on pickup, purchase, drink). Polled each frame for HUD sync.
- **Dependencies:** ItemCatalog, BalanceCfg, HUD, Audio, AchievementTracker.
- **Key data:** `EconomyState { treasure: int, potions: int, pendingPotions: int (5s cooldown), weapon: Item?, shield: Item?, armour: Item? }`.

### 4.12 Shop System
- **Responsibility:** When the player enters a Shop Shrine cell, pause enemy AI, open the shop UI, enforce slot exclusivity (only one weapon, one shield, one armour), validate affordability, apply purchase, close.
- **Update frequency:** Event-driven.
- **Dependencies:** Economy, ItemCatalog, MenuUI, TimeSystem (pause), Audio.
- **Key data:** `ShopState { availableItems: Item[], currentSelection: int?, isOpen: bool }`.

### 4.13 HUD System
- **Responsibility:** Render the persistent overlay: 17 hearts, treasure count, potion stack, dice tray, current AD/DD, potion hotkey, map toggle hint, blocking warning light.
- **Update frequency:** Every render tick. Pulls from GameState, does not own state.
- **Dependencies:** GameState, Localisation, Render.
- **Key data:** None owned; all reactive.

### 4.14 Camera System
- **Responsibility:** Frame the current room tightly (room-at-a-time camera), smooth-follow the player within the room, apply shake on hits / explosions, letterbox the screen edges for mobile (keep the dungeon square centred).
- **Update frequency:** Every render tick.
- **Dependencies:** PlayerState, RoomManager, TimeSystem (for shake decay), BalanceCfg (framing constants).
- **Key data:** `CameraState { targetPos: Vec2, currentPos: Vec2, zoom: float, shakeAmp: float, shakeDecay: float }`.

### 4.15 Animation System
- **Responsibility:** Drive sprite state machines for player, enemies, environment props. Integrate with AnimationPlayer and AnimationTree.
- **Update frequency:** Every render tick.
- **Dependencies:** PlayerState, EnemyStates, VFX.
- **Key data:** Per-entity `AnimState { currentAnim, frameIndex, loopFlag }`.

### 4.16 Audio System
- **Responsibility:** Route SFX to a bus, play music layers, crossfade between ambient / combat / boss music. Manage master volume, ducking during menus.
- **Update frequency:** Every render tick for music mixing; event-driven for SFX.
- **Dependencies:** Godot AudioServer, Balance (mixer settings).
- **Key data:** `AudioState { masterVolume, sfxVolume, musicVolume, activeMusicLayers, duckingAmount }`.

### 4.17 Save System
- **Responsibility:** Serialise `GameState` + `EconomyState` + `PlayerState` + RNG state to `user://runs/current.json` after every committed room. Double-buffer with `current.json` + `current.bak` for corruption protection. Load on startup and offer "Continue Run". Maintain separate save slots for bestiary, settings, achievements.
- **Update frequency:** Event-driven on room transitions (one write per ~5-15 seconds). Writes are async on a worker thread.
- **Dependencies:** RoomManager (transition hook), Godot FileAccess, platform-specific cloud sync (Steam Cloud, iCloud, Google Drive).
- **Key data:** `SaveBundle { version, gameState, playerState, economyState, rngState, timestamp }`.
- **Atomic write protocol:** write-to-temp, fsync, rename-to-current, delete-backup-if-temp-fsynced-ok.

### 4.18 Procedural Content System
- **Responsibility:** Own the SeedPRNG. Provide sub-streams for each subsystem (one for dungeon generation, one for combat dice, one for AI decisions, one for shop inventory), so that consumer drift in one area does not desynchronise another.
- **Update frequency:** On demand.
- **Dependencies:** None (it is a pure library).
- **Key data:** `PrngState { rootSeed, substreamCounters[Stream] }`.

### 4.19 Telemetry / Analytics
- **Responsibility:** Opt-in funnel metrics: tutorial completion rate, death row distribution, average run length, boss kill rate, common cause-of-death. Strictly anonymous.
- **Update frequency:** Event-driven on milestone events.
- **Dependencies:** A platform-agnostic HTTP client. GDPR-compliant opt-in flow on first launch.
- **Key data:** `TelemetryEvent { eventType, timestamp, runSeed, runElapsed, payload }`. Never contains PII.

### 4.20 Achievement Tracker
- **Responsibility:** Listen to domain events (`MonsterKilled`, `BossKilled`, `RunWon`, `NoPotionRun`, `PerfectParryStreak`), maintain progress, fire unlocks to Steam / platform achievement APIs and to the local bestiary.
- **Update frequency:** Event-driven.
- **Dependencies:** SteamAPI, SaveService, local achievements DB.
- **Key data:** `AchievementProgress { achievementId, progress, unlocked }`.

---

## 5. Data Architecture

Three layers: static config, runtime state, persistent state.

### Static config (read-only at runtime)

Shipped inside the binary as Godot `Resource` files or JSON, loaded once at boot.

- **`level_tables.json`** — 11 entries (L1-L10 + Boss), each a `d6 -> content map`. Maps row to encounter distribution.
- **`monsters.json`** — Every monster: id, display name, AD, DD, sprite ref, telegraph anim, strike anim, recover anim, flavor text.
- **`items.json`** — Shop catalog with cost, slot (weapon / shield / armour / potion-pack), AD bonus, DD bonus, exclusions.
- **`balance.json`** — Global constants (HP cap, starting AD/DD, potion cap, walking speed, hustle speed, parry window, commit window, swing cooldown, drink cooldown, cell pixel size, room pixel size, camera zoom, dice tumble duration).
- **`strings.json`** — Localised strings (EN default; JA + DE stretch).
- **`audio_manifest.json`** — SFX bus routing, music layer crossfades.

### Runtime state (in-memory, lost on quit without save)

- **`GameState`** — Root aggregate. Owns `Grid`, `Player`, `Economy`, `turnCount`, `runElapsed`, `RngState`.
- **`PlayerState`** — Position, cell coord, action state, swing charge, drink cooldown.
- **`EconomyState`** — Treasure, potions, pending potions, inventory slots.
- **`CombatState`** — Current encounter id, beat phase, beat timer, pending hits.
- **`RoomManagerState`** — Current room, door seals, lock flag.
- **`DiceTrayState`** — Dice pool, tumble progress, last result.
- **`CameraState`** — Target, current, zoom, shake.

### Persistent state (written to disk)

Kept in `user://`:

- **`current_run.json`** — The active run's snapshot. Written after every room commit. Reloaded on "Continue Run".
- **`current_run.bak`** — Previous atomic write. Restored if `current_run.json` fails to parse.
- **`settings.json`** — Master volume, SFX volume, music volume, input bindings, colour-blind mode, screen shake intensity, accessibility flags.
- **`bestiary.json`** — Monsters encountered, monsters killed, first-kill timestamps, damage dealt, damage taken.
- **`achievements.json`** — Unlocked achievements with timestamps. Mirrored to Steam API where available.
- **`telemetry_queue.json`** — Pending telemetry events waiting for upload (offline-first).

### Determinism ownership

The `RngState` is captured inside `GameState` so that save/load is bit-for-bit reproducible. A replay from a save writes the exact same combat rolls and the exact same dungeon content as a fresh run from the same seed.

---

## 6. Netcode

**N/A — single-player only.** This is a deliberate lock from Stage RT-6 (`GenreCrystallization.md` section 10, "What This Is Not"): no PvP, no co-op, no leaderboards in MVP. Consequences:

- No authoritative server.
- No lag compensation.
- No rollback.
- No prediction / reconciliation.
- Save files are local-first with optional Steam Cloud mirror.

If daily-seed leaderboards are added in v2, they would use a thin write-once HTTP POST from the client on run completion — not a real-time network protocol. See section 13 for anti-cheat implications.

---

## 7. Platform Targets & Minimum Hardware

### PC minimum spec (Windows 10+ / Linux / macOS 11+)

- CPU: Dual-core 1.6 GHz (e.g. Intel Atom, Celeron N, AMD E-series)
- RAM: 2 GB
- GPU: Integrated (Intel HD 3000 or newer)
- Storage: 300 MB
- Input: Keyboard + mouse or Xbox / PlayStation gamepad

### Steam Deck profile

- Target: 60 fps at 1280x800
- Power profile: 6W TDP cap, should comfortably sustain 60 fps with battery life >6 hours
- Input: native Steam Deck controls; full gamepad navigation of all menus
- Trackpad: map to mouse cursor for map overlay convenience
- Verified checklist: all UI text >=12pt at 1280x800, all interactions gamepad-reachable, no mouse-only input, save/resume works in sleep state, no keyboard prompts shown on the Deck

### Mobile min spec

- **iOS:** iPhone 8 / 2017 (A11) or newer, iOS 14+
- **Android:** 2018+ devices with OpenGL ES 3.0, Android 9+
- RAM: 2 GB
- Storage: 300 MB
- Input: on-screen virtual stick (left thumb) + three action buttons (right thumb: parry, swing, dodge) + tap-to-map-overlay
- Adaptive framerate: 60 fps default, fall back to 30 fps when thermal throttling or battery save kicks in

### Render / asset budget per platform

| Resource | PC | Steam Deck | Mobile |
|---|---|---|---|
| Sprite atlas size | 2048x2048 | 2048x2048 | 1024x1024 (downscaled at build time) |
| Max on-screen entities | 250 | 250 | 150 |
| Max particle count | 400 | 400 | 200 |
| Audio channels | 32 | 32 | 16 |
| Anti-aliasing | MSAA 2x | MSAA 2x | off |

---

## 8. Tick Budget per Frame

Target: 16.67 ms per frame at 60 Hz.

| System | Budget (ms) | Worst-case (ms) | Notes |
|---|---|---|---|
| Input poll | 0.5 | 1.0 | Trivial; mostly reading Godot's input queue. |
| Time system | 0.1 | 0.2 | Increment counters. |
| Scene manager | 0.2 | 0.4 | State-machine transition checks. |
| Player controller | 1.0 | 1.8 | Movement resolution, swing intent, collision request. |
| Enemy AI | 1.8 | 3.0 | Up to 6 enemies running the beat FSM. ~0.3 ms per enemy. |
| Combat FSM | 2.5 | 3.5 | Phase transitions, dice request, damage apply. |
| Dice tray tick | 1.5 | 2.5 | Physics tumble for up to 10 dice during a roll. Only ~500ms per combat exchange. |
| Physics / collision | 1.0 | 1.8 | Area2D overlaps for swing arcs and pickups. Very cheap for our entity count. |
| Reachability BFS | 0.0 | 0.3 | Runs only on cell commits, not every frame. Amortised zero. |
| Economy / inventory | 0.1 | 0.2 | Rare events. |
| Camera update | 0.4 | 0.7 | Smooth-follow + shake decay. |
| Animation system | 1.5 | 2.5 | Player + up to 6 enemies + VFX sprites. |
| HUD / UI update | 0.8 | 1.2 | Heart pips, dice tray panel, potion stack. |
| Render | 4.5 | 6.0 | 2D canvas draw calls. The ink-parchment shader is the heaviest cost but still cheap at 2D. |
| Audio tick | 0.4 | 0.8 | Bus mixing, SFX scheduling. |
| Save hook (amortised) | 0.1 | 0.5 | Async worker thread; main loop only enqueues. |
| **Subtotal** | **14.4** | **26.4** | |
| **Slack** | **2.27** | ~-9.7 (worst case) | |

**Interpretation.** The average budget lands at ~14.4 ms, leaving 2.27 ms slack. A pathological worst-case frame (all systems at worst, mid-combat, mid-dice-roll, with boss ad spawn) could blow the budget, but (a) this only happens during the 500 ms dice-tumble window, and (b) we allow the render loop to skip to the next vsync without dropping a logic tick. The logic loop is fixed-step and does not miss beats.

**Reality check.** In practice we expect to sit at ~8-10 ms per frame for most of the game (walking around rooms, no combat). Combat peaks at ~12-14 ms. The boss fight is the only place that approaches the budget. This is a 2D game with ~220 peak entities on machines that can comfortably render thousands. **There is no performance risk.** The budget exists to catch regressions, not to be a binding constraint.

---

## 9. Determinism Strategy

### The seed chain

Every run starts from a single 64-bit seed. That seed is:

- **Daily-seed runs:** derived from the calendar date (`YYYYMMDD`) hashed with a fixed salt.
- **Custom-seed runs:** typed by the player in the menu.
- **Freestyle runs:** drawn from `os.randi()` at run start.

The root seed is fed into a `Mulberry32` PRNG at run boot. From that single stream, we split into **named sub-streams** so that consumer drift in one system does not desynchronise another:

```
root_seed
  |
  |-> dungeon_stream     (used once at RunStart to place content)
  |-> combat_stream      (used for monster attack rolls)
  |-> defence_stream     (used for player defence rolls)
  |-> player_attack_stream (used for player attack rolls)
  |-> ai_behaviour_stream (used for monster pattern selection)
  |-> shop_stream        (used for shop inventory generation)
  |-> cosmetic_stream    (used for visual dice tumble nudges)
```

Each sub-stream is seeded from `SplitMix64(root_seed, stream_name_hash)` at boot. When the run saves, we persist each sub-stream's current state (counter + position). When the run loads, we restore all sub-streams. A replay of a save produces byte-identical dice results.

### Why sub-streams matter

If the player opens the map overlay (which uses the cosmetic stream to animate the ink flourish) mid-combat, that must not cause the *next combat roll* to land differently. Without sub-streams, pulling one number from a shared RNG for the flourish would shift every subsequent combat roll by one position, and the run would diverge from its recorded state. Sub-streams isolate concerns cleanly.

### Replay

Given a seed and a recorded input log (input actions + timestamps), the game can replay a run bit-for-bit. This is used for:

- **Daily-seed leaderboard verification** (v2).
- **Regression tests** (play a known-good run; verify final state).
- **Bug reports** (user submits seed + input log; developer replays locally).

The replay system is not in the MVP launch scope but the determinism discipline exists so we can add it later at low cost.

### Dice tumble vs logical roll

A critical detail: the visual dice tumble is NOT deterministic if driven by the Godot physics engine (which is internally non-deterministic across builds). Instead:

1. `DiceTray.Roll(count, stream)` immediately computes the logical dice faces by pulling `count` values from the sub-stream.
2. The tumble animation runs the physics simulation for ~500 ms with the logical result decided up front.
3. At the end of the tumble, each die is *snapped* to the pre-computed face. The visual tumble is a cosmetic lie; the logical result is deterministic and was decided before the animation started.
4. `RollFinished(hits)` fires with the pre-computed hit count.

This gives us visually satisfying "dice clatter" physics without sacrificing replay fidelity.

---

## 10. Save Strategy

### Auto-save cadence

Save points fire on `RoomCleared` (after a room is fully resolved: combat done, pickups collected, potions consumed, doorways open). This is roughly every 5-15 seconds of gameplay. The save contains:

- Current `GameState` (grid, visited cells, current cell).
- `PlayerState` (pos, HP).
- `EconomyState` (treasure, potions, inventory).
- `RngState` (all sub-stream counters).
- `runElapsed` (for run summary stats).

### Atomic write protocol

Saves use a double-buffered atomic write to prevent corruption from power loss:

1. Write bundle to `current_run.json.tmp`.
2. `fsync` the temp file.
3. Rename `current_run.json` to `current_run.bak`.
4. Rename `current_run.json.tmp` to `current_run.json`.
5. `fsync` the directory.

On load, the loader tries `current_run.json` first; if parse fails, falls back to `current_run.bak`. If both fail, the player is taken to the main menu with a "your previous run could not be restored" message.

### Save slots

| Slot | Purpose | When written | Read at |
|---|---|---|---|
| `current_run.json` | The active run | Every `RoomCleared` | "Continue Run" button on main menu |
| `bestiary.json` | Cross-run metaprogression: monsters seen, first-kills | After any `MonsterKilled` event | Bestiary menu; affects nothing gameplay-wise |
| `settings.json` | Volume, rebinds, accessibility flags | On any settings change | Every launch |
| `achievements.json` | Local achievement progress mirror | On any unlock | Achievements menu |

### Resume flow

1. On app launch, `SaveService.LoadCurrentRun()` returns either `Some(bundle)` or `None`.
2. If `Some`, the main menu shows "Continue Run" as the highlighted default option.
3. On select, the game restores all state and drops the player back into the current cell, with a short 1 s fade-in.
4. After restore, all sub-stream counters are at the exact position they were at save time, so the next combat roll is deterministic.

### Save NOT during combat

A key design decision: saves only fire **between** rooms, not during combat. This means:

- A mid-combat crash returns the player to the start of that room.
- The player cannot save-scum a bad dice roll.

This is a deliberate parallel to the source board game's "you committed, deal with it" ethic.

### Cloud sync

Optional, platform-dependent:

- **Steam:** Steam Cloud syncs `user://` files automatically if configured in `steam_appid.txt`.
- **iOS:** iCloud Documents container mirrors the save directory.
- **Android:** Google Play Games Services Saved Games (if signed in).

Cloud sync is best-effort. The local save is authoritative; cloud is a backup.

---

## 11. Testability Strategy

### Pure functional domain layer

The rules core (`RulesEngine`, `ReachabilityChecker`, `DamageCalculator`) is pure: same inputs produce same outputs, no side effects. This lets us write unit tests that cover every rule path:

```csharp
[Test]
public void MonsterAD4_Vs_PlayerDD1_ExpectedHits()
{
    var state = TestStates.FreshRun();
    var rng = new DeterministicRng(new[] { 6, 6, 3, 1 }); // inject fake rolls
    var result = RulesEngine.RollMonsterAttack(state, monsterId: "demon", rng);
    Assert.That(result.hitsLanded, Is.EqualTo(2));
}
```

### Injectable RNG

Every system that needs randomness takes an `IRng` interface, not a concrete PRNG. Production uses `Mulberry32Rng`; tests use `DeterministicRng(int[] sequence)` that replays a canned sequence. Combat scenarios become fully scriptable.

### Snapshot tests for room states

For each room content type (empty / treasure / potion / monster L1-10 / shop / boss), we store a known-good `GameState` snapshot after resolution. The test runs `RulesEngine.ResolveRoom(stateBefore, action) -> stateAfter` and compares `stateAfter` to the snapshot. Any drift is a failing test.

### Integration tests

Headless Godot (`godot --headless`) runs scripted scenarios:

- "Walk from Start to End avoiding monsters" — asserts the player can reach End.
- "Dead-end suicide" — walks the player into a known stranding and asserts `RunLost(Blocked)` fires.
- "Dracular fight" — scripts 9 parry inputs and asserts `BossDead` fires.

### Property tests

We use FsCheck-style property tests for invariants:

- "For any seed, the generated dungeon has at least one path from Start to End."
- "For any sequence of legal moves, player HP never exceeds 17 and never goes below 0 without triggering Death."
- "For any dice pool size N, counting 6s matches the known binomial expectation within statistical bounds over 10,000 rolls."

### Test coverage target

- Rules core: 100% line coverage, 100% branch coverage.
- Game logic (FSMs, managers): 80% line coverage.
- Presentation layer (HUD, VFX, sprites): smoke-tested only; visual regression is out of scope for automated tests.

---

## 12. Performance Budget & Profiling

### Baseline claim: there is no performance risk

This game has:
- 1 player avatar,
- <=6 simultaneous enemies (rare; usually 1),
- ~200 particle peaks,
- A 10-die physics tray,
- 9x10 static grid,
- 2D sprites with a simple ink shader.

On a ~$200 integrated-graphics laptop from 2016, this runs at 60 fps with several milliseconds of slack. On a 2023 gaming PC it runs at 240+ fps unlocked. The Steam Deck handles it at low power draw. Mobile runs it at 60 fps on hardware from 2018+.

### Why we still budget

- To catch regressions. If a content-creator mod adds 500 simultaneous dice to the tray and the frame drops to 30 fps, we want a CI failure, not a shipped bug.
- To gate memory budgets on mobile. iOS backgrounds apps aggressively; we need the whole game to fit in 180 MB RAM on the iPhone 8.
- To keep the ink shader honest. Fragment shaders can trivially eat frame budget if authored carelessly. We cap shader-per-pixel cost at 30 ALU ops.

### Profiling tooling

- **Godot built-in profiler** for CPU time per system.
- **Tracy** (via `gdnative-tracy` or similar) for fine-grained tracing in debug builds.
- **Godot render debugger** for draw call counts and overdraw visualisation.
- **Mobile:** Xcode Instruments (iOS), Android GPU Inspector (Android).

### Perf CI

We run an automated headless benchmark on each build:

1. Load a canned save at the boss room.
2. Script 60 seconds of combat inputs.
3. Sample frame time every 16.67 ms.
4. Assert: p99 frame time < 20 ms on the reference benchmark machine (2018 mid-range laptop).

Any regression trips the CI red.

---

## 13. Security / Anti-Cheat

**N/A for MVP — single-player.** There is no server, no leaderboard, no competitive context. Cheating is a non-issue because there is no one to cheat.

### v2 considerations (daily-seed leaderboards)

If daily-seed leaderboards ship later:

- The client submits `(seed, final_game_state, input_log, run_elapsed)` to a backend.
- The backend **verifies** by replaying the input log against the seed inside a headless engine. If the replay produces a different `final_game_state`, the submission is rejected.
- This is the same verification strategy used by `speedrun.com` for replay-based runs.
- Rate limit submissions to 1 per seed per account per day.
- Store only seed, outcome, run length — no PII.

### Privacy

- No personal data collected by default.
- Telemetry is opt-in, anonymous, GDPR-compliant.
- Save files are local-first. Cloud sync is opt-in and platform-native (Steam Cloud, iCloud).
- No third-party analytics SDKs, no ad networks, no fingerprinting.

---

## 14. Risks

### Godot-specific risks

| Risk | Severity | Mitigation |
|---|---|---|
| **Godot save API edge cases with C# classes.** The `Resource`-based save flow is well-trodden for GDScript but has occasional pitfalls when mixing C# records. | Medium | Serialise `GameState` to JSON via `System.Text.Json`, not via Godot's `Resource.save()`. This sidesteps the Godot serialiser entirely for the domain state. Godot Resource save is still used for `settings.json` (simple KV). |
| **Godot 4.x C# export binary size.** Mono/.NET runtime inclusion adds ~40-70 MB to the binary on each platform. | Low | Acceptable; still under 300 MB total install on all platforms. |
| **Godot 4.x mobile export maturity.** Mobile export was rewritten for Godot 4 and has had occasional input-latency reports early on; this is improving with each patch. | Medium | Target latest stable Godot (4.3+) at ship time. Dogfood the game on Steam Deck and mobile from week 1 of development; do not defer platform testing. |
| **Godot hot-reload fragility for C# scripts.** C# does not hot-reload cleanly in Godot; editor restart is often required after script changes. | Low | GDScript glue code where hot-reload matters (menus, tutorial); C# for systems that don't change frequently (rules core, combat FSM). |

### Engine-independent risks

| Risk | Severity | Mitigation |
|---|---|---|
| **Ink shader performance on older mobile GPUs.** A poorly written fragment shader can tank frame rate on low-end Androids. | Medium | Cap the shader at 30 ALU ops per pixel. Profile early on the worst reference device (Samsung Galaxy A10). Fall back to a baked ink texture if shader is too expensive. |
| **Determinism drift under floating-point math.** Cross-platform floats can produce different bit patterns for the same computation. | Medium | All rules-core math is integer-based. Dice rolls, BFS, hit counts, heart pips are all ints. Floats are only used for presentation (positions, shake, shader parameters) and do not enter the save state. |
| **Save corruption on sudden power loss (mobile background kill).** iOS and Android kill backgrounded apps aggressively. | Medium | The atomic write protocol + double-buffered saves handle this. Additionally, mobile background hooks trigger an immediate save before suspend. |
| **Save file format drift across patches.** A new build loads an old save; fields are missing or extra. | Medium | Save bundle includes a `version` field. On load mismatch, run a migration pipeline. If no migration exists for the version gap, show a "save is from a version too old to upgrade" message and offer to start fresh. |
| **Input rebinding conflicts.** Players rebind `Parry` to `E`, then use `E` for the menu, causing a conflict. | Low | Godot `InputMap` detects duplicates and prompts on rebind. Show a warning dialog. |
| **Accessibility: dice tray is visually heavy.** Players with dyslexia or visual processing differences may struggle to count 6s in real time. | Medium | Every rolled die shows its face as both a pip pattern and a numeral. 6s flash / pulse. An auto-count indicator shows the hit total numerically. This is already in scope per `AccessibilityAudit.md`. |

---

## 15. Diagram Reference

See `RT-Architecture.mmd` for the full system diagram. The diagram has six labelled layers:

1. **Platform layer** — Godot 4.3 + platform build targets (PC, Steam Deck, Mobile).
2. **Core systems layer** — Game Loop, Time, Input, Physics/Collision, Rendering, Audio, Save Core.
3. **Game logic layer** — Scene Manager, Room Manager, Dungeon Generator, Reachability, Combat FSM, Dice Tray, Player Controller, Enemy AI, Economy, Shop, Procedural Content.
4. **Presentation layer** — HUD, Dice Tray UI, Map Overlay, Menus, Tutorial Overlay, VFX, Camera, Animation.
5. **Data layer** — Level Tables, Monster Defs, Item Catalog, Balance Config, Seed PRNG, Localisation.
6. **Services layer** — Save Service, Telemetry, Achievements, Steam/Platform APIs.

Edges in the diagram are labelled with the type of dependency (read / event / tick). Colour coding is by layer.

The **Dice Tray** appears as its own component in the Game Logic layer (owning the tumble physics and hit calculation) AND in the Presentation layer (owning the visible panel). This is deliberate: the dice tray is both a simulation and a diegetic UI element, and splitting it across the two layers captures the distinction between the logical roll and the visual lie.

---

## Final Architecture Lock

- **Engine:** Godot 4.3, C# (systems) + GDScript (glue, UI, tutorial)
- **Pattern:** Hybrid — Godot scene-tree OOP for presentation + services for logic + pure-functional rules core
- **Loop:** 60 Hz fixed logic, variable render, `_physics_process` / `_process` split
- **Concurrency:** Single-player, no netcode, no authoritative server
- **Save:** Atomic double-buffered JSON after every room, never during combat
- **Determinism:** Single root seed, sub-streams per system, integer-only rules math
- **Platforms:** PC (primary), Steam Deck (verified), Mobile (secondary)
- **Perf risk:** None for MVP; budget exists only as regression guardrail
- **Risk profile:** Green — scope is well within Godot's sweet spot

All downstream stages (RT-D8 Balance, RT-D9 Assets, RT-D10 Prototypes, RT-D11 Deployment) consume this architecture as a fixed input. Any stage that wants to contradict a lock above must cite a specific reason why.
