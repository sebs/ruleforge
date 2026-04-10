# System Architecture — Solo Dungeon Bash (Digital)

See `Architecture.mmd` for the full diagram. Target platform: Web (HTML5/PWA). Stack is suggestive rather than prescriptive — the architecture is framework-agnostic.

## High-Level Layers
```
UI → Controller → Domain ← Data
              ↘ Services → Persistence
```

### 1. UI Layer
Pure presentation; reads from `GameState`, dispatches actions via `InputHandler`.

| Component | Responsibility |
|---|---|
| `GridView` | Renders the 9×10 dungeon grid. Highlights legal cells. Draws the ink path. |
| `HUDView` | Persistent display of HP/AD/DD/Treasure/Potions + equipped items. |
| `CombatView` | Dice animations, hit/block tallies, combat log. Modal during combat. |
| `ShopView` | Item catalog, prices, affordability, slot conflicts. Modal at step 7. |
| `PotionView` | Potion-use modal at step 6. |
| `MenuView` | Title screen: Play, Tutorial, Settings. |
| `EndView` | Victory/Defeat screen with run stats. |
| `TutorialOverlay` | Floating tooltips during the scripted onboarding run. |

### 2. Controller Layer (FSMs)
Converts user intent into domain operations and guarantees turn-sequence correctness.

- **`TurnController`** — 7-step turn FSM. States: `pick_square`, `rolling_content`, `resolving_content`, `in_combat`, `recovery`, `shop`, `turn_end`. Disallows out-of-order actions (e.g. can't shop before combat).
- **`CombatController`** — Inner combat FSM. States: `monster_atk`, `player_def`, `dmg_check`, `player_atk`, `monster_def`, `kill_check`. Runs to completion before returning control to `TurnController`.
- **`InputHandler`** — Normalizes mouse/touch/keyboard events into action objects.

### 3. Domain Layer (Rules Engine)
Pure functions over immutable game state — easily unit-testable.

- **`GameState`** — Root aggregate. Owns `Grid`, `Player`, `turn_count`, `phase`, RNG seed, history.
- **`Player`** — `hp`, `base_ad`, `base_dd`, `treasure`, `potions_available`, `potions_pending`, `inventory`.
- **`Inventory`** — `weapon_slot`, `shield_slot`, `armour_slot`. Each is `Option<Item>`.
- **`Grid`** — 2D cell array + start/end coords. Each cell: `unvisited | visited_with_content(content_type) | current`.
- **`RulesEngine`** — Stateless functions:
  - `legal_moves(state) -> [coord]`
  - `move(state, target) -> state`
  - `roll_content(state, rng) -> state`
  - `start_combat(state, monster) -> state`
  - `combat_round(state, rng) -> state`
  - `use_potions(state, n) -> state`
  - `buy_item(state, item_id) -> state | Error`
  - `check_win_loss(state) -> Outcome`
- **`Reachability`** — BFS from current cell to End through unvisited cells (8-way). Runs after every move.

### 4. Data Layer
Static JSON assets hot-loadable for modding:
- `level_tables.json` — 11 entries (L1–L10 + Boss), each a d6→content map.
- `items.json` — Shop catalog with cost, bonuses, slot, exclusions.
- `monsters.json` — Monster IDs to AD/DD/display name/sprite.
- `balance.json` — Global constants (HP cap, starting pools, etc). Switchable by difficulty mode.

### 5. Services
- **`RNG`** — Seeded PRNG (e.g. Mulberry32 or SplitMix). Test mode accepts an injectable dice sequence.
- **`SaveService`** — Serializes `GameState` to localStorage after every turn. Loads on startup.
- **`Audio`** — Plays one-shot SFX; manages ambient music loops.
- **`AchievementTracker`** — Listens to domain events; unlocks on specific conditions (boss kill, no-potion run, etc).
- **`Analytics`** — Optional, opt-in. Tracks basic funnel metrics (tutorial completion, run outcomes) — **must respect do-not-track and GDPR**.

### 6. Persistence
- **`LocalStore`** — Browser localStorage / IndexedDB. Fields: current save, settings, run history, achievements.
- **`CloudSync`** — v2 stretch. Not needed for MVP.

## Data Flow (one full turn)
1. Player clicks a legal cell in `GridView`.
2. `InputHandler` dispatches `{type: "move", target: (r,c)}`.
3. `TurnController` validates phase == `pick_square`, calls `RulesEngine.move()`.
4. `RulesEngine` returns new state with the cell marked visited and pending content roll.
5. `TurnController` transitions to `rolling_content`, calls `RulesEngine.roll_content(state, rng)`.
6. Based on content, transitions to `resolving_content`. If monster → `in_combat`, otherwise directly to `recovery`.
7. `CombatController` runs its inner FSM until combat ends, then returns control.
8. `TurnController` advances to `recovery`. `PotionView` opens if player has potions.
9. After recovery, transitions to `shop`. `ShopView` opens.
10. After shop, transitions to `turn_end`. `SaveService.save(state)`. State → UI re-renders. `TurnController` returns to `pick_square`.

## Deployment Architecture (MVP)
- **Single static SPA** served from a CDN (e.g. Netlify / Cloudflare Pages).
- No backend required for MVP. All state is client-side.
- Service Worker provides offline capability (PWA).
- Optional lightweight backend (Cloudflare Worker + KV) later for daily seed leaderboards.

## Testability
- The Domain layer is **pure functions** — 100% unit-testable without mocking.
- `RNG` dice injection (F027) lets combat scenarios be deterministically replayed.
- Loop boundary tests: verify every `GameLoop.mmd` state transition.
- Snapshot tests: `(state, action) → next_state` pairs covering edge cases.

## Performance Budget
- Grid is 90 cells; trivial render cost.
- BFS reachability is O(90) per move; zero perf concern.
- Dice animations ≤ 60fps on any modern device.
- Save state ≤ 5 KB JSON; instant write.

## Security / Privacy
- No personal data collected by default.
- If analytics is added, it is opt-in and documented.
- Cloud sync (v2) would require account system and proper auth.
- No server-authoritative game state — this is a solo game, cheating is a non-issue.

## Scaling Considerations
Not applicable for v1 (single-user, client-side). If daily-seed leaderboards are added:
- Backend is write-once-per-run, read-many; Cloudflare KV or D1 is sufficient.
- Rate-limit submissions (1 per seed per user per day).
- Store only seed, outcome, run length — no PII.

## Risks & Mitigations
| Risk | Mitigation |
|---|---|
| Players feel dice are rigged | Deterministic RNG with visible seed; combat log of every roll |
| Tutorial is skipped, players confused | First-time tooltips (F024) trigger in the real game too |
| Path-blocking feels unfair | Warn-before-commit on move (F016 + F029) |
| Shop slot conflicts confusing | Greyed out items with explanatory tooltip (F013) |
| Save corruption | Double-buffer save slots (current + previous) |
| Performance on low-end devices | Simple rendering + low domain complexity means this is a non-issue |
