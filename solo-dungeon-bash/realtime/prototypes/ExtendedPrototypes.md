# Extended Prototype Prompts — Solo Dungeon Dash (RT)

> RealTimeForge Stage RT-A6
> Two specialised prototypes added to the base three documented in `../design/PrototypePrompts.md`.
>
> Tooling targets: **Godot 4 (GDScript)** as primary engine; a tiny **Cloudflare Workers / Firebase / Node-Express** backend is required only for Prototype 4.

---

## 1. Introduction

The base `PrototypePrompts.md` file (RT-9) specifies three prototypes that form the critical path for proving Solo Dungeon Dash as a real-time 2D top-down action roguelite:

1. **Core Loop Prototype** — Godot 4, ~24 dev-hours (one weekend). Single-room, single-enemy, proves that the parry-and-dice-tray verb loop "feels good". Placeholder art.
2. **Conflict Prototype (Multi-Enemy Parry Arena)** — Godot 4, ~40 dev-hours (one week). Arena with three enemy types, hit-stop, particles, screen shake, gamepad. Proves combat variety + feel polish.
3. **Full Vertical Slice (Pitch Demo)** — Godot 4, ~160 dev-hours (one month). 3×3 mini-dungeon + Shop Shrine + one-phase Dracular boss + hand-drawn art. Proves the pitch to publishers/playtesters.

Those three cover the **single-player local loop** end to end. But two important technical questions are not answered by them:

- **Networking:** Solo Dungeon Dash is single-player, but **Mode A (Ghost Races)** — daily/weekly asynchronous leaderboards with playable ghost overlays — is an explicit feature on the RTGDD feature list, and the RTGDD also reserves a "potential future Mode B co-op" slot. Ghost Races need a backend, a deterministic-seed replay system, an anti-cheat pass, and a UI for leaderboard + ghost playback. None of the base three prototypes touch these.
- **AI/NPC depth:** The base three prototypes cover at most **3** of the planned **11** monster types, and none of them prove the full roster behaves distinctively under automated stress testing. The world-feel question — *"does the dungeon feel alive, or are the monsters recolored damage sponges?"* — is load-bearing for the pitch and for balance tuning, and cannot be answered from Prompt 2's three-enemy arena.

Prototypes 4 and 5 in this document plug those two gaps. Neither is required to build Prototype 3 — they run in parallel with Prototype 3, not before it — but both must be de-risked before the **Alpha** milestone.

Because Solo Dungeon Dash is fundamentally single-player, Prototype 4 is **not** a traditional real-time networked multiplayer prototype. There is no authoritative server simulation, no client-side prediction, no reconciliation, no rollback netcode. Instead, Prototype 4 validates an **asynchronous ghost-race pipeline**: upload a recorded run, download other players' ghosts, play them back as cosmetic overlays on top of your own local deterministic run. That is the correct and minimum-complexity networking layer for this game.

---

## 2. Summary Table — All Five Prototypes

| # | Name                       | Scope (dev-hours) | Dependencies | Engine                  | Priority   | Milestone            |
|---|----------------------------|-------------------|--------------|-------------------------|------------|----------------------|
| 1 | Core Loop                  | 24h               | none         | Godot 4                 | P0 (first) | Weekend hack         |
| 2 | Multi-Enemy Conflict       | 40h               | Prompt 1     | Godot 4                 | P1         | Week 1               |
| 3 | Full Vertical Slice        | 160h              | Prompts 1+2  | Godot 4                 | P2         | Month 1 (pitch demo) |
| 4 | Ghost Race Networking      | 40h               | Prompt 1     | Godot 4 + tiny backend  | P3         | Alpha (month 2+)     |
| 5 | AI/NPC Stress Test         | 50h               | Prompt 1     | Godot 4                 | P2         | Parallel to Prompt 3 |

**Recommended build order:** **1 → 2 → 5 → 3 → 4**

Rationale for ordering:
- Prompt 1 is the prerequisite for everything. The player controller, dice tray, HUD, and combat FSM are all reused downstream.
- Prompt 2 proves multi-enemy combat feel before we invest in art or AI depth.
- Prompt 5 (AI Stress Test) runs *before* Prompt 3 because the 11 monster profiles must be tuned before the vertical slice locks in its 3 hero monsters and its Dracular boss. Deferring AI tuning to after the slice means re-tuning after art is already locked. The AI stress test also surfaces balance regressions early.
- Prompt 3 is the pitch demo. Build it only once Prompts 1, 2, and 5 have de-risked combat, feel, and AI.
- Prompt 4 is last because Ghost Races are a Mode A feature that is **nice-to-have for launch, not required for the pitch**. It can ship with a first post-launch update if schedule slips.

Dependency diagram:

```
Prompt 1 (Core Loop)
   |
   +----> Prompt 2 (Multi-Enemy)
   |          |
   |          +----> Prompt 3 (Vertical Slice)
   |
   +----> Prompt 5 (AI Stress Test) ----> Prompt 3 (AI-informed)
   |
   +----> Prompt 4 (Ghost Race Networking)
```

---

## 3. Prototype 4 — Ghost Race Networking Prototype

> **Goal:** Prove that the deterministic seed + ghost-run recording + playback pipeline works end-to-end for the **Mode A — Ghost Races** feature. Specifically: one player finishes a run, uploads a compact replay, and a second player can see that run as a translucent overlay racing alongside their own attempt on the same seed. Also prove the backend rejects tampered runs.
>
> **Engine:** **Godot 4.x GDScript** (primary, client). **Cloudflare Workers + KV** (primary, backend); Firebase Firestore or a tiny Node + Express process is an acceptable fallback for local dev.
> **Dimension:** 2D top-down (same as Prompt 1).
> **Input:** mouse + keyboard (inherited from Prompt 1).
> **Scope:** ~40 developer-hours.

### 3.1 Architecture overview

This prototype has **two halves**:

1. **Client (Godot 4)** — a "Race Day" scene that reuses Prompt 1's player + single-enemy room, adds a run recorder, a ghost playback layer, an upload/download client, and a simple leaderboard HUD.
2. **Backend (Cloudflare Worker + KV, or Firebase, or Node Express)** — a stateless HTTPS service with three endpoints:
   - `POST /run` — accept a run record, verify it, store it.
   - `GET /leaderboard/:seed/:metric` — return top-N runs for a given daily seed, sorted by metric.
   - `GET /run/:run_id` — return a full run record (for ghost playback).

**Sync model:** fully **asynchronous**. There is no real-time sync, no authoritative server simulation, no rollback, no lockstep. Each player plays locally against a deterministic seed. After their run, they upload a compact JSON record. Before their *next* run they pull 1–5 ghosts of other players' runs on the same seed.

This is the minimum-complexity networking model that satisfies Mode A. It is essentially "share-a-replay with scoreboard", which is a solved problem.

### 3.2 Protocol

Transport: **HTTPS POST** with JSON body. No WebSockets. No UDP. No long polling.

**`POST /run` request body:**
```json
{
  "schema_version": 1,
  "run_id": "uuid-v4",
  "player_id": "anonymous-uuid-v4-persisted-locally",
  "client_version": "0.4.0",
  "seed": "2026-04-10-daily",
  "seed_chain_hash": "sha256-of-the-seed-chain",
  "timestamp_ms": 1744300800000,
  "outcome": "victory",
  "duration_ms": 87420,
  "rooms_visited": 9,
  "parries_landed": 14,
  "parries_missed": 3,
  "hearts_lost": 4,
  "treasure_earned": 11,
  "monsters_killed": { "orc": 3, "wolf": 2, "skeleton": 1, "dracular": 1 },
  "input_log": "base64-encoded-compressed-30hz-delta-input-stream"
}
```

**`GET /leaderboard/:seed/:metric`** returns:
```json
{
  "seed": "2026-04-10-daily",
  "metric": "duration_ms",
  "top": [
    { "run_id": "...", "player_id": "...", "value": 74320, "outcome": "victory" },
    { "run_id": "...", "player_id": "...", "value": 75990, "outcome": "victory" }
  ]
}
```

**Metric enum:** `duration_ms` (fastest), `hearts_lost` (cleanest), `parries_landed` (most stylish), `treasure_earned` (richest).

### 3.3 Input log format

- **Tick rate:** 30 Hz (every 33.3 ms). The game simulates at 60 Hz but the input log only captures 30 Hz — state changes between ticks are interpolated on playback.
- **Delta encoding:** only record frames where input state changes. For a 90-second run that is typically 400–1200 events, averaging ~20 bytes each before compression, ~8–24 KB per run before `gzip` → usually ~2–6 KB compressed.
- **Frame schema:** `(tick, input_mask, mouse_x_quantized, mouse_y_quantized)` where `input_mask` is an 8-bit bitfield (WASD, LMB, RMB, Space, E).
- **Serialization:** bit-packed to `PackedByteArray`, then gzipped, then base64'd into the JSON payload.

### 3.4 State sync

**NONE.** Each player plays locally. Ghosts are strictly cosmetic overlays — they cannot damage the player, cannot interact with world objects, and cannot affect the RNG stream. The ghost's character pose, position, and attack animation are all driven by replaying the ghost's input log against the same seed.

### 3.5 Determinism & anti-cheat

- **Determinism requirement:** the game must be fully deterministic given `(seed, input_log)`. This means:
  - Single fixed-point physics (or a disciplined integer/fixed-step loop)
  - All RNG draws from a single seeded PCG32 stream derived from `seed`
  - No `randi()` / `randf()` / `OS.get_ticks_msec()` in gameplay code — only in UI/VFX
  - No dependency on frame time for simulation (fixed 60 Hz update)
- **Server verification:** on `POST /run`, the Cloudflare Worker (or Node server) runs a **headless Godot replay** (or a Rust/JS reimplementation of the simulation kernel) that re-executes the input log against the seed and checks that the reported outcome, duration, hearts lost, and parry counts match within a tolerance of 0. Any mismatch → HTTP 422, run is rejected.
- **Replay kernel:** for the prototype, a slimmed-down Godot headless build invoked by the worker via a sidecar service is acceptable. For production, porting the combat kernel to Rust/WASM makes the verification fit in a 10ms Worker request.
- **Rate limit:** 1 run per player per 10 seconds. Stops floods.

### 3.6 Ghost rendering

- **Overlay layer:** a new `GhostLayer` Canvas in the Godot scene tree, drawn above the floor but below the player.
- **Per-ghost node:** a semi-transparent sprite (alpha 0.35) that follows the ghost's replayed position, plays the same animations as the live player character, and shows a small floating name-tag above its head ("ghost_a7f3", anonymous).
- **Simultaneous count:** 1–5 ghosts max. UI toggle: show top 1, top 3, top 5, or none.
- **Fade:** ghosts finish their run earlier or later than the live player. On ghost finish, the sprite fades to 0 over 800ms and displays an ink-splash "finish" marker at their final position with their run time.
- **Color coding:** each ghost has a different hue tint (green, cyan, magenta, yellow, orange) to distinguish them.
- **Decluttering:** if more than 3 ghosts overlap within 1 meter, slight random positional jitter (±10 px) is applied to keep them legible.

### 3.7 Leaderboard

- Backend stores a sorted set per `(seed, metric)` using Cloudflare KV or Firestore indexed on metric.
- Default metric: **fastest time among victory runs**.
- Only the top 100 runs per `(seed, metric)` are kept; older entries pushed out.
- **Cold-start problem:** a brand-new seed starts with 5 pre-recorded "developer ghost" runs (a "dev_bot" player_id). These act as benchmarks: a "Bronze" (slow), "Silver" (average), "Gold" (good), "Platinum" (fast), and "Dev" (hand-tuned fastest known) ghost. Players always see at least the Bronze ghost for comparison.
- Client refreshes the leaderboard via `GET /leaderboard/:seed/:metric` on:
  - Entering the Race Day scene
  - Completing a run
  - Pressing the refresh button on the leaderboard HUD

### 3.8 Privacy, GDPR, and anti-cheat policy

- `player_id` is a v4 UUID generated locally on first launch and persisted to `user://player_id.txt`. **Never** a real name, email, or device identifier.
- No PII is ever transmitted. No IP is logged beyond Cloudflare's default log retention window.
- **Right-to-delete:** a simple email (`delete@gameserver.example`) that admins can process manually during the prototype. For production, a `DELETE /player/:player_id` endpoint.
- **Terms:** a single "By uploading, you agree to have your anonymous run times shown in leaderboards" one-click confirm on first upload, stored locally.
- **GDPR:** since no PII is stored, compliance is minimal — the only data subject attribute is `player_id` itself, which is pseudonymous and can be deleted on request.

### 3.9 Testing plan

- **Same-machine test:** two Godot instances run on the same developer machine, both pointing at a `localhost:8787` Wrangler dev server. Player A runs the seed, uploads; Player B joins, downloads Player A's ghost, races against it. Both runs appear on the leaderboard.
- **Tamper test:** hand-edit a run's `outcome` field from `defeat` to `victory` before upload; server rejects with 422.
- **Replay drift test:** run the same seed + input log 10 times through the replay kernel; all 10 must produce identical state traces (bitwise identical RNG state at every tick).
- **Ghost count stress:** spawn 5 ghosts simultaneously + live player; measure FPS on target hardware (Steam Deck baseline). Must hold 60fps.

### 3.10 File structure

```
prototype4/
  backend/
    wrangler.toml                 — Cloudflare Worker config
    src/
      index.ts                    — Worker entrypoint, HTTP routing
      verify.ts                   — Replay verification harness
      storage.ts                  — KV wrapper (or Firestore fallback)
      schema.ts                   — Zod schemas for the request/response
    test/
      verify.test.ts              — unit tests for replay verification
      storage.test.ts             — KV mock tests
    README.md                     — "how to run locally with wrangler dev"
  client/                         — Godot 4 project
    project.godot
    scenes/
      race_day.tscn               — Race Day scene (leaderboard + live run + ghosts)
      race_day.gd
      ghost_layer.tscn
      ghost.gd
      leaderboard_hud.tscn
      leaderboard_hud.gd
      player.tscn                 — re-used from Prompt 1
      enemy.tscn                  — re-used from Prompt 1
    scripts/
      backend_client.gd           — HTTPS client wrapping the 3 endpoints
      input_recorder.gd           — captures 30 Hz delta input log
      input_playback.gd           — replays a ghost's input log
      seed_chain.gd               — daily seed derivation + chain hash
      rng.gd                      — deterministic PCG32 wrapper
      run_record.gd               — Godot dictionary → JSON serializer
    tests/
      determinism_test.gd         — regression test; runs the same seed 10x
  shared/
    schema.json                   — JSON Schema for the run record
    README.md                     — protocol docs
```

### 3.11 Scene description — Race Day

A single scene that shows:

- **Background:** same parchment-ink room from Prompt 1, slightly wider to fit a leaderboard panel on the right.
- **Top-center:** seed header — "Daily Seed · 2026-04-10 · Rank 12 / 247" with a small refresh button.
- **Center:** the playable room. Same dimensions as Prompt 1's room.
- **Overlay (GhostLayer):** up to 5 ghost character sprites playing back their recorded runs in parallel with the live player's own run. Each ghost is 35% alpha with a distinct hue.
- **Right panel (LeaderboardHUD):** scrollable list of the top 10 runs for this seed, each row showing `rank · player_id_short · duration · outcome_icon`. A dropdown selects the sort metric (`duration_ms` / `hearts_lost` / `parries_landed` / `treasure_earned`). A toggle selects "show 1 / 3 / 5 / 0 ghosts".
- **Bottom-center:** Dice Tray (reused from Prompt 1).
- **After run:** a modal shows the player's final stats, their leaderboard rank, and an "Upload" button. On upload success → toast "Run uploaded." and the leaderboard refreshes.

### 3.12 Acceptance criteria

- [ ] Two Godot clients running on the same machine can both complete a run on the same daily seed, upload, and both appear in the top-N leaderboard within 2 seconds
- [ ] A recorded ghost run plays back bitwise-identically to the live run that generated it (verified via state trace comparison — seed + input log → same position at every tick)
- [ ] Ghost overlay layer renders 5 ghosts simultaneously at 60fps on Steam Deck baseline hardware without dropped frames during combat
- [ ] Backend rejects a tampered input log (outcome field altered, or input log edited to produce an impossible result) with HTTP 422 and a clear error message
- [ ] Leaderboard refreshes and displays the new run within 2 seconds of the upload completing

### 3.13 Known risks

- **Backend cost scaling** if the game gets popular. **Mitigation:** Cloudflare Workers + KV has a very generous free tier (100k requests/day free) and the workload is small per request; a viral spike is cheap. At scale, batching verification into async queues costs near-zero.
- **Replay desync** if any non-determinism creeps into the simulation kernel (a single `randf()` slipping into gameplay code, a frame-time-dependent physics step, a font-metric-dependent UI that feeds back into gameplay). **Mitigation:** a disciplined `rng.gd` that is the only source of randomness in gameplay code; a determinism regression test (`determinism_test.gd`) that runs the same seed ten times in CI and fails on any state drift; a code-review rule that any gameplay PR must touch `rng.gd` explicitly if it adds randomness.
- **Ghost visual clutter** when 5 ghosts overlap in a small room. **Mitigation:** alpha 0.35, distinct hues, positional jitter, fade older-finisher ghosts, configurable "show 1 / 3 / 5 / 0" toggle.
- **Privacy** — player_id must be fully anonymous, and nothing PII-adjacent must ever be logged. **Mitigation:** locally-generated UUID, no device fingerprinting, no analytics beyond run metrics, terms of upload on first use, a written privacy note in the about screen.
- **GDPR compliance** for EU users — right-to-delete must be honoured. **Mitigation:** manual delete-by-email process for the prototype, `DELETE /player/:player_id` endpoint added before public release.
- **Replay verification cost** — a headless Godot per request is expensive. **Mitigation:** for the prototype, accept the cost (runs are rare per user). For production, port the combat kernel to Rust/WASM so the worker can verify a run in <10ms.
- **Cheater arms race** — anti-cheat is not a solved problem. **Mitigation:** the seed + input reproduction check is strong against casual tampering. Against determined attackers, accept that Ghost Races are cosmetic leaderboards, not competitive ranked, and the risk is low.

---

## 4. Prototype 5 — AI/NPC Behavior Stress Test

> **Goal:** Tune enemy AI behaviour and prove the dungeon world "feels alive". Specifically: prove that the **11 monster types** behave distinctively enough that a player can recognise each within 2 minutes of play; prove that parry reflexes match the source game's strategic feel; prove that each monster's expected-time-to-kill lands within ±20% of the balance simulation's target; and prove that Dracular's three-phase fight runs end-to-end in isolation.
>
> **Engine:** **Godot 4.x GDScript** (primary).
> **Dimension:** 2D top-down (same as Prompts 1–3).
> **Input:** mouse + keyboard + optional bot player.
> **Scope:** ~50 developer-hours.

### 4.1 Why this prototype exists

Prompt 2 only exercises 3 monster types. The full game has 11 (Orc, Wolf, Skeleton, Evil Warrior, Devil Bat, Cyclops, Dark Elf, Skeleton Lord, Wizard, Demon, Dracular-boss). If all 11 are implemented only during the Vertical Slice or Alpha, AI tuning collides with art, audio, and dungeon scaffolding, and balance regressions become painful to isolate. Prototype 5 exists so that monster AI can be tuned **in isolation** with an overnight bot harness, before it is wired into a dungeon.

This prototype also addresses one of the known risks in the Faithfulness Audit: *"AI feels samey if only telegraph timing differs."* By prototyping all 11 monsters side-by-side in one arena with live-tunable parameters, the team can verify that each monster has a **distinct shape** of attack — directional / radial / ranged / airborne / multi-hit / phased — not just a different number on a timer.

### 4.2 Monster behavioral profiles

Each of the 11 monster types must have a full AI implementation (Behavior Tree or Finite State Machine — designer's choice per enemy, whichever expresses the enemy more clearly). Each must have a distinct *shape* of behaviour, not merely different numbers.

| # | Monster       | Movement                              | Attack shape                       | Telegraph | Parryable? | Notes                                               |
|---|---------------|---------------------------------------|------------------------------------|-----------|------------|-----------------------------------------------------|
| 1 | Orc           | Slow straight approach, 2 m/s         | Directional overhead chop          | 1200 ms   | Yes        | The "tutorial" enemy; calm, readable                |
| 2 | Wolf          | Fast zig-zag approach, 3.5 m/s        | Short directional lunge            | 500 ms    | Yes (tight)| Feels aggressive; punishes slow reflexes            |
| 3 | Skeleton      | Erratic approach, 2.5 m/s, stops+starts| Clattering sword swing            | 900 ms    | Yes        | Unpredictable tempo                                 |
| 4 | Evil Warrior  | Shield-up, 1.8 m/s; shield breaks on parry| Shield-bash into riposte        | 800 ms    | Yes (two phases) | Must hit twice: break shield, then damage      |
| 5 | Devil Bat     | Airborne, 4 m/s, hovers then dives    | Aerial dive-attack                 | 600 ms    | Yes (ground only) | Forces player to watch the sky                |
| 6 | Cyclops       | Slow steady approach, 1.5 m/s, never retreats | Radial ground-slam (4m radius) | 1500 ms   | Yes        | Forces positioning / running away, not parry-dodge  |
| 7 | Dark Elf      | Flanks player (sidestep arc), 3.2 m/s | Multi-hit dagger combo (3 strikes) | 400 ms per hit | Yes (per strike) | Each hit in the combo has its own parry window |
| 8 | Skeleton Lord | Stays back, 1.2 m/s backward, 2.0 forward | Summons 2 ranged bone-shards    | 1100 ms   | No (dodge only) | First ranged enemy; changes spatial calculus   |
| 9 | Wizard        | Kiting, 2.5 m/s, runs away from player| Projectile spell                   | 900 ms    | No (dodge only) | Pure dodge check; parry is useless             |
| 10| Demon         | Slow approach 1.8 m/s, stationary cast| Arena-wide attack (forces movement)| 2500 ms   | No (positioning only) | Massive telegraph, forces corner movement |
| 11| Dracular (boss)| Three phases; see §4.3               | Phase-dependent                    | Varies    | Yes/No per phase | Full 3-phase fight                              |

Implementation note: the "shape" axes are `{directional, radial, ranged, airborne, multi-hit, positional}`. No two monsters share all axes. The differentiation is *mechanical*, not cosmetic.

### 4.3 Dracular — the three-phase boss

Dracular must be playable in isolation in this prototype (not deferred to Prompt 3, which will use a simplified one-phase version). The full three-phase fight:

- **Phase 1 (100% → 66% HP)** — melee phase. Dracular uses Orc-like directional swings with 800 ms telegraphs but 6 Attack Dice. Parryable.
- **Phase 2 (66% → 33% HP)** — evasion phase. Dracular teleports to a new arena corner every 3 seconds, casts Wizard-style unparryable projectiles. Dodge-only.
- **Phase 3 (33% → 0% HP)** — frenzy phase. Dracular uses a Cyclops-style radial telegraph plus a Dark-Elf-style 3-hit combo, alternating. Tight parry windows (150 ms), heavy punishment for mistakes.
- **Phase transitions:** a 1-second invulnerability window + a phase-change VFX + a new music cue.

### 4.4 Scene description — Arena

A single square arena (14×14 meters) with visible walls.

- Player spawn at center.
- Enemy spawner at top of arena.
- **Press `[N]`** → despawn current enemy, spawn next monster type (cycles through all 11).
- **Press `[G]`** → spawn 3 of the current monster type simultaneously (group stress test).
- **Press `[B]`** → spawn Dracular in his own isolated arena (the arena walls reshape to a boss room).
- **Press `[F1]`** → toggle Debug HUD.
- **Press `[F2]`** → toggle Bot player (replaces human input with the scripted test bot).
- **Press `[F3]`** → start overnight bot harness (runs 100 fights per monster unattended, writes a CSV).

### 4.5 Metrics overlay

The Debug HUD (`[F1]`) displays in real-time:

- Current enemy type name + HP
- Current enemy state (`Idle / Telegraphing / Attacking / Guard / Opening / Recovering`)
- Current telegraph timing (ms into window)
- Parry window visibility (ms remaining)
- Player DPS (rolling 5s average)
- Fight duration (current fight)
- Player HP + hearts lost this fight
- Parries landed / parries attempted ratio this fight
- Frame time and physics-step time

### 4.6 Live-adjustable AI parameters

A debug menu (opens on `[F5]`) exposes per-enemy parameters as sliders:

- `telegraph_ms` — base telegraph duration
- `parry_window_ms` — length of the parry window inside the telegraph
- `parry_window_offset_ms` — when the parry window begins inside the telegraph
- `attack_dice` — number of Attack Dice rolled on hit
- `movement_speed_m_s` — ground speed
- `aggressiveness` — 0..1 weight on how eagerly the AI closes distance vs. waits
- `telegraph_variance_pct` — ±% random variance per-attack to avoid robotic feel

Parameters are adjustable **without restarting the scene**. On change, the current enemy's AI re-reads the values at the next state transition. A "Save preset" / "Load preset" row at the bottom reads/writes a `presets.json` file per enemy.

### 4.7 Procedural AI variance

Within a single monster type, each attack draws a small randomised offset on `telegraph_ms` of ±15% to avoid metronomic predictability. The variance is seeded, so the overnight bot harness is still deterministic for reproducibility. The variance band is per-enemy-instance and re-rolled each attack.

### 4.8 Player bot (automated testing)

An optional scripted bot replaces human input (`[F2]` toggle). The bot:

- Moves in a simple "orbit the enemy at 2.5m" pattern with small random perturbations
- Attempts to parry every enemy telegraph with **70% accuracy** (the 30% failure models a competent-but-not-perfect player)
- Drinks a potion (`B` / `Q` key) when HP ≤ 4
- Dodges projectiles and radial telegraphs based on exact spatial checks
- Attacks during every Opening state

The bot is **not** meant to be a perfect player. It is a repeatable simulator for measuring expected-time-to-kill and damage-taken under average play.

### 4.9 Overnight bot harness

`[F3]` starts a scripted test run that:

1. Iterates through all 11 monster types (including all 3 Dracular phases)
2. For each monster: runs 100 fights at default difficulty
3. Logs to `test_runs/{timestamp}/{monster_name}.csv` with columns:
   - `fight_id, outcome, duration_ms, hearts_lost, parries_landed, parries_missed, dodges, potions_used, hits_landed, sixes_rolled`
4. Writes a summary `test_runs/{timestamp}/summary.md` with per-monster means, medians, stdev for each metric
5. Runs overnight (expected ~6–8 hours for 100 × 11 = 1100 fights)

The summary is then compared to the balance simulation's targets (from `revised/RT-BalanceSheet.md`). Any monster whose bot-measured expected-time-to-kill is >±20% off target is flagged for re-tuning.

### 4.10 Replay logging (for debugging)

Every fight writes a minimal replay log (`.replay` file) containing the seed, the input log, and the outcome. On a failed-acceptance bot run, the developer can load the replay in the arena and watch it play back at 0.25× speed to diagnose what went wrong.

### 4.11 File structure

```
prototype5/
  project.godot
  arena.tscn                      — single square arena, 14x14m
  arena.gd                        — handles spawner cycling, keybinds [N][G][B][F1-F5]
  player.tscn                     — re-used from Prompt 1, all verbs functional
  player.gd
  debug_hud.tscn                  — the metrics overlay
  debug_hud.gd
  debug_menu.tscn                 — the parameter-tuning sliders [F5]
  debug_menu.gd
  enemies/
    enemy_base.gd                 — shared FSM base class, parry/guard/opening/telegraph
    orc.gd                        — slow approach, directional chop
    wolf.gd                       — fast zig-zag, lunge
    skeleton.gd                   — erratic, clattering swing
    evil_warrior.gd               — shield-bash + riposte (two-stage)
    devil_bat.gd                  — airborne dive
    cyclops.gd                    — radial ground-slam
    dark_elf.gd                   — 3-hit combo, per-strike windows
    skeleton_lord.gd              — ranged bone-shards, kites backward
    wizard.gd                     — kiting projectile caster
    demon.gd                      — arena-wide telegraph, forces movement
    dracular.gd                   — 3-phase boss controller
    dracular_phase1.gd
    dracular_phase2.gd
    dracular_phase3.gd
    scenes/
      *.tscn                      — one scene per enemy
  ai/
    behavior_tree.gd              — shared BT node types (sequence, selector, leaf)
    fsm.gd                        — shared FSM helper
    ai_tuning.gd                  — runtime-adjustable parameter system
    presets.json                  — default tuning values per enemy
  bot/
    scripted_bot.gd               — 70%-accuracy player bot [F2]
    harness.gd                    — overnight 100-per-monster runner [F3]
    metrics.gd                    — CSV writer
    replay_logger.gd              — records every fight to .replay
    replay_player.gd              — loads a .replay and plays it back
  test_runs/
    .gitkeep                      — harness writes here
```

### 4.12 Acceptance criteria

- [ ] All 11 monster types have **distinct** behavioural profiles recognisable within 2 minutes of play by a first-time player (verified by blind-test: show a tester 2 minutes of each enemy and ask them to describe the difference; correct descriptions for all 11)
- [ ] Debug menu (`[F5]`) can adjust all per-enemy AI parameters in real-time without restarting the scene
- [ ] Automated bot harness (`[F3]`) can complete 100 fights per monster type unattended overnight and write a correct summary CSV with outcome, duration, and damage-taken statistics
- [ ] Each monster's bot-measured expected-time-to-kill, under its default tuning, lands within **±20%** of the target from the balance simulation in `revised/RT-BalanceSheet.md`
- [ ] Dracular full three-phase fight completes end-to-end (phase 1 → phase 2 → phase 3 → victory) without stuck states, under both human play and bot play

### 4.13 Known risks

- **11 monsters × AI tuning = a lot of manual iteration.** **Mitigation:** the overnight bot harness. One full sweep produces objective time-to-kill and damage-taken numbers for all 11 monsters; tuning becomes a data-driven loop instead of vibes-based.
- **AI feels samey if only telegraph timing differs.** **Mitigation:** enforce the "distinct attack shape" rule — directional / radial / ranged / airborne / multi-hit / positional. Each monster must have a unique combination. This is a code-review gate on `enemies/*.gd`.
- **Dracular's three phases are complex.** **Mitigation:** implement him *last* in the prototype and allocate a full week of the 50-hour budget to his fight. Isolated prototyping prevents entangling Dracular bugs with mob AI bugs. Three separate `dracular_phaseN.gd` scripts keep each phase as its own module.
- **AI desync with combat FSM.** Enemies must respect the `Guard / Opening / Recovering` state machine from Prompt 1, otherwise parries become unreliable. **Mitigation:** `enemy_base.gd` enforces the shared FSM; per-enemy scripts can override behaviour inside each state but cannot skip states. A unit test in `test_runs/fsm_contract.gd` verifies every enemy traverses at least once through each required state during a test fight.
- **Live-tuning without restart** can produce non-deterministic tuning history (developer A changes a slider, developer B cannot reproduce the session). **Mitigation:** every slider change is written to a timestamped log in `test_runs/tuning_history.csv` so changes are auditable. "Save preset" snapshots the current state to `presets.json`.
- **Balance-simulation target mismatch.** If the bot harness consistently produces times outside the ±20% target, the fault may be the balance simulation, not the AI. **Mitigation:** treat a failed acceptance criterion as a *conversation*, not a tuning-only bug — sometimes the target was wrong and should be updated.
- **Bot is too simple to surface nuanced tuning problems.** A 70%-parry bot will miss edge cases that a skilled human finds. **Mitigation:** the bot is a regression safety net, not a replacement for human playtests. Human playtests during Prompt 2 and Prompt 3 remain the ground truth for "does this feel good?"

---

## 5. All Five Prototypes — Dependencies, Shared Contracts, Build Order

### 5.1 Summary table (duplicated for convenience)

| # | Name                    | Scope (dev-hours) | Dependencies | Engine                 | Priority |
|---|-------------------------|-------------------|--------------|------------------------|----------|
| 1 | Core Loop               | 24h               | none         | Godot 4                | P0       |
| 2 | Multi-Enemy Conflict    | 40h               | Prompt 1     | Godot 4                | P1       |
| 3 | Full Vertical Slice     | 160h              | Prompts 1+2  | Godot 4                | P2       |
| 4 | Ghost Race Networking   | 40h               | Prompt 1     | Godot 4 + backend      | P3       |
| 5 | AI/NPC Stress Test      | 50h               | Prompt 1     | Godot 4                | P2       |

**Total budget:** 314 developer-hours (~8 weeks for one developer working full-time, or ~4 months at half-time).

### 5.2 Build order

**Recommended:** `1 → 2 → 5 → 3 → 4`

Phasing rationale:

- **Phase A (Prompts 1, 2, 5) — 114 hours, ~3 weeks.** All combat feel + all AI proven. At the end of this phase, the team knows whether the combat system is fun and whether the 11 monsters have enough variety to carry a full game. If Phase A fails, the project should pivot or restart, *before* investing in art and the Vertical Slice.
- **Phase B (Prompt 3) — 160 hours, ~4 weeks with 1 programmer + 1 artist part-time.** The pitch demo. Only built if Phase A succeeds. Reuses all combat, feel, and AI work from Phase A.
- **Phase C (Prompt 4) — 40 hours, ~1 week.** The networking layer. Built after the Vertical Slice, because Ghost Races are a post-pitch feature — not required for the pitch demo. Can be deferred to post-launch if schedule slips.

### 5.3 Shared contracts across prototypes

Several contracts are shared across prototypes and must not drift:

- **`player.gd` contract** — movement, attack, parry, dodge state machine. Defined in Prompt 1, reused by 2, 3, 4, 5. Any change requires updating all downstream prototypes.
- **`enemy_base.gd` FSM** — `Idle / Telegraphing / Attacking / Guard / Opening / Recovering`. Defined in Prompt 1, extended in 2, 3, 5. Prompt 5 enforces the contract via a unit test.
- **Dice Tray** — defined in Prompt 1, reused unchanged by 2, 3, 5. Prompt 4 reuses it inside the Race Day scene.
- **`rng.gd` deterministic PCG32** — defined in Prompt 1 but only critical from Prompt 4 onward (ghost replays require determinism). Any gameplay randomness must go through this module.
- **`run_record.gd`** — the JSON run record format. Defined in Prompt 4. Prompt 5's bot harness could also write run records for debug purposes, but this is optional.

### 5.4 File-structure recap for all five prototypes

```
prototype1/                       — Core Loop (24h)
  project.godot
  main.tscn, main.gd
  scenes/player.*, enemy.*, dice_tray.*, hud.*
  assets/sfx/*.wav

prototype2/                       — Multi-Enemy Conflict (40h)
  scenes/arena.tscn
  scenes/enemies/orc.*, wolf.*, cyclops.*
  scenes/vfx/hit_spark.*, ink_splash.*, dust.*
  scripts/enemy_base.gd, orc.gd, wolf.gd, cyclops.gd
  scripts/wave_spawner.gd, combat_fx.gd, gamepad_config.gd

prototype3/                       — Full Vertical Slice (160h)
  scenes/main_menu.tscn
  scenes/tutorial/*.tscn          (5 tutorial rooms)
  scenes/dungeon/room_*.tscn      (9 dungeon rooms)
  scenes/dungeon/shop_shrine.tscn
  scenes/dungeon/boss_arena.tscn  (Dracular 1-phase)
  scenes/end_screen.tscn
  art/                            (~80 hand-drawn sprites)
  audio/                          (3 music tracks + ~20 SFX)

prototype4/                       — Ghost Race Networking (40h)
  backend/                        (Cloudflare Worker + KV)
    wrangler.toml, src/index.ts, src/verify.ts, src/storage.ts
    test/
  client/                         (Godot 4)
    scenes/race_day.*, ghost_layer.*, ghost.*, leaderboard_hud.*
    scripts/backend_client.gd, input_recorder.gd, input_playback.gd
    scripts/seed_chain.gd, rng.gd, run_record.gd
    tests/determinism_test.gd
  shared/schema.json

prototype5/                       — AI/NPC Stress Test (50h)
  arena.tscn, arena.gd
  player.tscn (re-used from Prompt 1)
  debug_hud.*, debug_menu.*
  enemies/enemy_base.gd + 11 monster scripts + dracular phase scripts
  enemies/scenes/*.tscn (11 + dracular)
  ai/behavior_tree.gd, fsm.gd, ai_tuning.gd, presets.json
  bot/scripted_bot.gd, harness.gd, metrics.gd, replay_logger.gd, replay_player.gd
  test_runs/
```

### 5.5 Scene description recap for all five prototypes

- **Prompt 1:** one room, one player, one enemy, a Dice Tray, a 17-heart HUD. Fixed camera.
- **Prompt 2:** one arena (14×14m), waves of 1–3 enemies across 3 types, hit-stop + particles + gamepad. Fixed camera.
- **Prompt 3:** menu → tutorial (5 rooms) → 3×3 dungeon + Shop Shrine + Boss room → end screen. Per-room camera framing with pan transitions.
- **Prompt 4:** one "Race Day" room with the live player, 1–5 ghost overlays, and a leaderboard panel on the right. Fixed camera.
- **Prompt 5:** one debug arena, spawner cycles through all 11 monster types + Dracular, live-tunable AI parameters, overnight bot harness. Fixed camera.

### 5.6 Cross-prototype acceptance criteria summary

| Criterion                                    | P1 | P2 | P3 | P4 | P5 |
|-----------------------------------------------|----|----|----|----|----|
| Player verb loop works (<50ms latency)        | Y  | Y  | Y  | Y  | Y  |
| Enemy telegraph is readable                   | Y  | Y  | Y  | -  | Y  |
| Parry window is landable                      | Y  | Y  | Y  | -  | Y  |
| Dice Tray feels satisfying                    | Y  | Y  | Y  | Y  | Y  |
| Multiple enemy types feel distinct            | -  | 3  | 3  | -  | 11 |
| Hit-stop + screen shake feel good             | -  | Y  | Y  | -  | -  |
| Gamepad input parity                          | -  | Y  | Y  | -  | -  |
| Hand-drawn art style                          | -  | -  | Y  | -  | -  |
| Shop Shrine integrates naturally              | -  | -  | Y  | -  | -  |
| Dracular full fight                           | -  | -  | 1p | -  | 3p |
| Deterministic run replay                      | -  | -  | -  | Y  | -  |
| Leaderboard + ghost overlay                   | -  | -  | -  | Y  | -  |
| Backend rejects tampered runs                 | -  | -  | -  | Y  | -  |
| Overnight bot harness runs 100 fights × 11    | -  | -  | -  | -  | Y  |
| Live-tunable AI params without restart        | -  | -  | -  | -  | Y  |

Legend: `Y` = required, `-` = not applicable, `3` = 3 enemies, `11` = 11 enemies, `1p`/`3p` = 1-phase/3-phase Dracular.

### 5.7 Known risks across the prototype suite

- **Scope creep.** The biggest risk. Each prototype has a strict scope; do not let "while we're in there" additions extend any prototype beyond its budget by more than 20%. If a prototype exceeds budget by >20%, stop and reassess.
- **Contract drift between prototypes.** `player.gd`, `enemy_base.gd`, and the Dice Tray must stay consistent. **Mitigation:** shared-module directory `shared/` that all five prototypes depend on via Godot's asset library or a git submodule.
- **Determinism regression from Prompt 1 onward.** Any gameplay randomness that bypasses `rng.gd` will break Prompt 4. **Mitigation:** a CI regression test that runs the same seed 10 times and asserts identical state traces, enabled from Prompt 1 onward even though only Prompt 4 depends on it.
- **Art timeline blocking Prompt 3.** The Vertical Slice needs ~80 hand-drawn sprites from one artist over two weeks. **Mitigation:** start the artist working on Prompt 3 assets during Phase A (parallel to Prompts 1, 2, 5), using the enemy list already known from the source game.
- **Backend operational risk for Prompt 4.** A public Cloudflare Worker means the prototype is exposing an endpoint to the internet. **Mitigation:** during the prototype, keep the endpoint behind a Wrangler dev server on localhost; only go public after Prompt 3 ships and a privacy/security review is done.
- **AI tuning drift.** Prompt 5 produces tuned parameters, but those parameters must flow forward into Prompts 3 and the Alpha. **Mitigation:** `presets.json` from Prompt 5 is the single source of truth for enemy tuning; Prompt 3 imports it directly.

---

## 6. Glossary & References

- **Mode A — Ghost Races:** asynchronous leaderboard feature; see `revised/RT-Features.md`.
- **Mode B — Co-op (future):** not implemented in any prototype; reserved for a potential post-launch update.
- **Parry Window / Opening State:** see `analysis/AgencyModel.md` and `analysis/ConflictModel.md` for timing rationale.
- **Dice Tray:** the d6 UI that is the game's signature visual motif; see `revised/RT-Mechanics.md`.
- **Seed Chain:** the deterministic RNG chain used for daily seeds + run reproducibility; see `revised/RT-BalanceSheet.md` for parameter tables.
- **Balance targets:** all expected-time-to-kill targets referenced by Prompt 5's acceptance criteria come from `revised/RT-BalanceSheet.md`.
- **Onboarding:** the Tutorial Room in Prompt 3 follows the script in `revised/RT-OnboardingDesign.md`.
- **Architecture:** Prompt 3 should match the technical spec in `revised/RT-Architecture.md`. Prompt 4's backend should conform to the "External Services" section of the same document (to be added when MultiplayerDesign.md lands in `architecture/`).

---

*End of ExtendedPrototypes.md — RealTimeForge Stage RT-A6.*
