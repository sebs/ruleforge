# Solo Dungeon Dash — Real-Time Game Design Document

> Working title: **Solo Dungeon Dash** (descendant of *Solo Dungeon Bash*, Felbrigg Herriot, 2007, BookRanger.co.uk)
> Document type: Primary deliverable of RealTimeForge (Stage RT-7)
> Written for: a developer or small team who has never seen the original board game. This document is self-contained.

---

## 1. Concept Statement

**Solo Dungeon Dash is a 2D top-down action roguelite where every room of a hand-drawn dungeon is a gamble, every fight is a parry-dance, and a visible tray of six-sided dice makes the damage sing.** You carve a path upward through a 9×10 procedurally seeded dungeon, fight 1-HP-but-well-guarded monsters with a parry-and-riposte combat system, and try to reach the Big Bad Boss Dracular without dying or stranding yourself from the exit. Every run is 20 minutes, every death is a clean restart, and every successful parry rolls your dice tray on screen — because this game descends from a dice-pool roll-and-write, and we are not hiding that heritage. We are putting it on screen.

The promise: **Hades's room-flow, Dead Cells's parry, Into the Breach's commitment, with the tactile soul of a dungeon master's notebook.**

---

## 2. Core Loops

### The 30-second loop (moment-to-moment)
```
Enter room → read the foes → wait for telegraph → parry / dodge → dice tray rolls → ink kill → scoop treasure → pick next door
```

This is the atomic pleasure: a clean parry, a cascade of dice rolling on the right side of the screen, a cluster of 6s landing, a monster bursting into an ink spill, coins skittering across the flagstones. Under three seconds of flow that you can feel in your hands.

### The 5-minute loop (session arc)
```
Start room → early rooms (low-dice monsters) → first Shop Shrine → gear choice → push deeper → mid-tier fight → potion recovery → commit to a higher level row → risk climbs
```

Within five minutes a player has made 2–3 major gear decisions, cleared 8–12 rooms, found 1–2 Shop Shrines, and started to feel the shape of the run. The mid-run is where the **routing puzzle** layer activates: "do I go left across row 5 for more treasure, or straight up through 6 to save turns?"

### The 20-minute loop (full match)
```
Start → early farm → first shop → mid push → boss arena decision → commit to End → Nine Dice Duel → victory or death → run summary → tap to retry
```

A complete run. Median 20 minutes, 80th-percentile spread 12–35 minutes. Three macro phases:

1. **Opening (0–5 min)** — you're weak, monsters are weak, rooms are mostly calm. You're scouting and collecting treasure.
2. **Midgame (5–14 min)** — monsters have meaningful bite. Every encounter is a real fight. Routing starts to matter because the grid is filling up with "visited" cells.
3. **Endgame (14–20 min)** — you're geared or you're not. You commit to the End square. The 90-second **Nine Dice Duel** vs Dracular is the climax.

Every loop is **death-terminated or victory-terminated** — no continues, no revives. A death resets the dungeon but unlocks bestiary entries and cosmetics.

---

## 3. Player Experience Goals

Five statements, each expressing an emotion the player should reliably feel:

1. **"That parry felt amazing."** The player should feel competent and in control during combat, even when monsters are dangerous. The parry window must be generous enough to land regularly but tight enough to feel earned. *Target: at least one "nice!" parry per minute of mid-run combat.*

2. **"I knew that was a bad route."** The player should feel responsible for their losses, not victimized by them. When a run ends from routing misjudgement, the player should see their own path on the post-run screen and nod. *Target: fewer than 10% of deaths feel unfair to post-run survey.*

3. **"Just one more run."** The 20-minute run length plus the bestiary tease plus the daily seed challenge should produce strong replay pull. *Target: average session has 2.5+ runs.*

4. **"I'm holding the dice."** The visible Dice Tray should make players feel the randomness physically. When 3 sixes come up on a 5-dice attack, the player should feel a surge. *Target: players describe the game as "it feels like rolling real dice."*

5. **"Dracular was a fight, not a wall."** The final boss should feel climactic — punishing, but learnable. Players should die to Dracular several times, learn his phases, and feel the moment they finally beat him. *Target: median "first Dracular kill" happens in run #4–7.*

---

## 4. Dimensional & Visual Direction

### Dimension
**2D top-down, room-at-a-time camera.**
- 2D over 3D — matches the hand-drawn source, keeps scope indie-sized, supports mobile well.
- Top-down over isometric/side-scroll — preserves the 8-way king-adjacency movement of the source literally.
- Room-at-a-time camera — one octagonal room fills the screen; doorways transition via a ~300ms camera pan. Enforces the "one decision per room" rhythm.

### Camera
- Default: fixed, framing the current octagonal room with a small overhang showing the next doorways' silhouettes.
- Shake on hit.
- Zoom-out during boss fight to show the wider End arena.
- Map overlay (V / Select / pinch-out) smoothly zooms out to a full 9×10 inked grid view showing visited cells and current position. Releasing the key returns to the room camera.

### Art Style
**Hand-drawn ink-on-parchment.** This is not cosmetic; it is fundamental to identity. Specifics:
- Wobbly black ink line work, cross-hatch shading, faint parchment grain background
- Monochrome base with selective color: gold (treasure), blue (potions), red (monsters and incoming damage telegraphs), green (current cell, safe), grey (visited cells)
- UI chrome drawn as if inked on a notebook sheet — hearts are tiny ink sketches, dice are drawn cubes with hand-lettered faces
- Typography: handwritten (Caveat or Homemade Apple) for labels; clean sans-serif (Inter) for numbers
- Monsters drawn as tiny bestiary-style inked sketches; each has a signature silhouette
- Dracular drawn in extra detail as the climactic set-piece

### Reference Images (mood board direction)
- Game: *Night in the Woods* (hand-drawn 2D feel)
- Game: *Gorogoa* (pen-and-ink illustration style)
- Game: *Slay the Princess* (ink-line character art)
- Book: Edward Gorey, *The Doubtful Guest* (cross-hatching)
- Game: *West of Loathing* (stick-figure comic aesthetic — contrast reference, we want more polished)

---

## 5. Temporal System

From `analysis/TemporalMap.md`. How time works in Solo Dungeon Dash:

- **Continuous real-time at 60 Hz.** The world does not pause during combat unless the player opens the map overlay or pause menu.
- **Player-paced locomotion.** Outside combat there is no timer. You can stand in an empty room for 10 minutes if you want to study the map.
- **Beat-based combat cadence.** Combat has a built-in rhythm:
  - Monster telegraph starts at T=0ms
  - Parry window opens at T=400ms and closes at T=600ms (200ms window)
  - Monster attack resolves at T=700ms (parried → Opening; unparried → damage)
  - During Opening (1000ms after a successful parry) the player can safely attack and the Dice Tray rolls
  - Monster recovers and enters next telegraph cycle
- **Room transitions** take ~300ms of camera pan. During the transition, the player is not taking input.
- **Shop Shrines pause the action** locally (the room is safe, no timer) — they are the only "pause" beat in the game other than the menu.
- **Reachability warning** fires in the brief commit window before a move commits: a red icon pulses if the next move would strand you from End.
- **Match timer is hidden by default** but can be toggled on for speedruns.

No combat cooldowns in the source; we add a few:
- **Dodge cooldown:** 700ms after a dodge ends, you cannot dodge again. This prevents dodge-spam.
- **Global cooldown:** none. All other inputs are instant.

---

## 6. Spatial System

From `analysis/SpatialModel.md`.

### The dungeon
- **9 columns × 10 rows = 90 octagonal rooms**, plus Start (attached below row 1, middle column) and End (attached above row 10, middle column).
- Each room is an octagonal chamber with up to 8 doorways (N, NE, E, SE, S, SW, W, NW). Grid-edge rooms have correspondingly fewer doors.
- Rooms are ~12×12 meters in simulated scale, rendered at ~800×800 pixels at 1080p.
- **Doorways seal behind you** — a physical ink-chain or collapsing floor animation blocks the door you came through after ~1 second. This enforces no-revisit literally.

### Procedural seeding
- The dungeon layout is the same 9×10 shape every run (fixed topology; no randomized room shapes).
- The *contents* of each cell are pre-seeded from the run seed and the row's level table probabilities. The player encounters randomness at generation time, not at encounter time.
- Daily seed mode uses a deterministic hash of the date.

### Movement
- **Continuous WASD / left stick** inside each room. The player has free movement within room walls.
- **Doorway transitions are commit-locked** — walking into a doorway for ~200ms commits to that room transition. Within the commit window the player can reverse direction. After commit, camera pans to the new room (~300ms), doors close behind.
- **King-adjacency is preserved as door connectivity:** each room has doors only where king-adjacent rooms exist.

### Reachability
- A background BFS runs continuously. If a candidate doorway move would leave End unreachable, the doorway's ink glow turns red and a warning icon pulses. The player can still commit, but they've been warned.

### Map overlay
- Hold **V** (M+K), press **Select** (gamepad), or pinch-out (touch). Zooms out to a full 9×10 inked overview with visited cells filled in, current cell highlighted, monster icons revealed for defeated enemies, and path lines drawn in ink.
- Releasing the input smoothly zooms back to room view.

---

## 7. Action Vocabulary

From `analysis/AgencyModel.md`. The full list of verbs the player commands:

### Movement
| Action | Input | Feel |
|---|---|---|
| Walk | WASD / left stick | Responsive, ~4 m/s, no acceleration curve |
| Dash through doorway | WASD + hold toward door | Commit lock, then camera pan |
| Open map overlay | V / Select / pinch-out | Smooth zoom-out to full dungeon |

### Combat
| Action | Input | Feel |
|---|---|---|
| Light attack | Left click / X button / tap | ~300ms swing, rolls Dice Tray on hit, only damages enemies in Opening state |
| Parry | Right click / RB / parry-tap | 200ms timing window, perfect parry creates Opening + negates damage |
| Dodge roll | Spacebar / A / swipe | 350ms i-frames, 400ms recovery, 700ms cooldown |
| Heal (drink potion) | Q / LB / potion hotkey | 500ms animation, +1 heart per potion, capped at 17 |

### Interaction
| Action | Input | Feel |
|---|---|---|
| Interact with Shop Shrine | E / Y / tap altar | Opens radial item menu with costs; time doesn't pause but the shrine room is safe |
| Buy item | Click item / select + A / tap | Instant purchase if affordable; grey out if conflict |
| Pick up treasure | Auto-pickup when adjacent | Coin-shower particles + audio chime |

### Meta
| Action | Input | Feel |
|---|---|---|
| Pause | Escape / Start | World freezes; shows pause menu + settings + quit |
| Quick restart | End-of-run screen → R / A | Instant new run with new seed |

### Deliberately excluded
- **Aim/ranged attack** — no ranged combat in v1. Melee-only.
- **Ability tree** — no skill tree.
- **Magic spells** — no magic system.
- **Inventory management beyond 3 slots + potions** — kept minimal.
- **Jumping** — 2D top-down, no verticality.

---

## 8. Information & UI

From `analysis/InfoArchitecture.md`.

### HUD wireframe (ASCII)
```
┌────────────────────────────────────────────────────────────────┐
│ ♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥            ROW 7                   ⚠ MAP [V] │
│                                                                │
│                                                                │
│                       [GAME ROOM VIEW]                         │
│                                                                │
│                                                                │
│                                                                │
│                                                                │
│                                                                │
│ 🗡⚔⚔     [DICE TRAY: ⚀⚁⚂⚃⚄⚅]   🛡⛨⛨⛨   💰 14   🧪 3          │
└────────────────────────────────────────────────────────────────┘
```

### What each element shows
- **Hearts (top-left):** 17 heart pips, fills deplete on damage. Red flashes on hit.
- **Row indicator (top-center):** which dungeon row the current cell belongs to (1–10).
- **Reachability warning (top-right):** ⚠ icon pulses when End is at risk; smoothly fades when safe.
- **Map toggle hint (top-right):** reminds the player V opens the map.
- **Dice Tray (bottom):** 5 primary dice slots, filled based on current Attack Dice (left half) and Defence Dice (right half). Unused slots are greyed. On roll, dice visibly animate, 6s pulse gold.
- **Currency (bottom-right):** Treasure icon + count.
- **Potion hotkey (bottom-right):** Potion icon + count. Shows both "available" and "pending (bought this room)" if different.

### Fog of war
- **No layout fog.** The full 9×10 grid topology is visible on the map overlay from the start. (This preserves the source's "you have the rulebook, you know the shape.")
- **Content fog.** The contents of unvisited rooms are hidden until entry. The map overlay shows visited rooms filled in and unvisited rooms as empty ink outlines.

### Randomness visibility
- **Full transparency.** The level tables are visible in the in-game Codex from turn 1 — no unlock gating. The player can pull up a probability reference any time. This honors the source's "you have the rulebook" trust model.

### Combat information flow
- **Damage numbers** float up briefly over hit target (disableable in settings).
- **Telegraph highlight:** enemy sprite flashes red at telegraph start.
- **Parry window marker:** a yellow ring pulses on the enemy briefly, indicating the exact moment to parry.
- **Hit confirmation:** 80ms hit-stop on a landed blow + brief slowmo on Dice Tray cascade.
- **Miss:** dice land on non-6s with a muted clatter.
- **Kill:** ink-spill particle effect, unique SFX.

### Post-run dashboard
On death or victory, show:
- Outcome (Victory / Died / Blocked)
- Dungeon path overlay (inked trail on 9×10 grid)
- Monsters defeated (tally by type)
- Treasure earned, potions used, parries landed, parries missed
- Run time
- New bestiary entries unlocked
- New cosmetics unlocked
- Share button (copy share string to clipboard)
- Play Again button

---

## 9. Conflict Systems

From `analysis/ConflictModel.md`.

### The Roll-and-Parry combat system
Every encounter unfolds like this:

1. **Player enters room.** Fog-of-content dissolves; monster(s) revealed. Camera settles. Monster aggros immediately.
2. **Monster telegraphs attack.** Visible wind-up animation ~600ms. A red glow at 400–600ms marks the parry window.
3. **Player chooses a response:**
   - **Parry** (right-click / RB) — if timed to the parry window, perfectly blocks + stuns monster into Opening state for 1 second.
   - **Dodge** (spacebar / A) — i-frames for 350ms, avoids damage but doesn't create Opening.
   - **Tank** (do nothing) — monster's attack dice roll into the player's Defence Dice Tray. Each 6 in monster's pool = 1 Hit, each 6 in player's Defence Tray cancels 1 Hit, remaining Hits = lost hearts.
4. **If Opening was created:** player has 1 second to attack with light attack. Each light attack rolls the **Attack Dice Tray** — every 6 is a guaranteed lethal hit (remember: monsters are 1 HP). If no 6 is rolled, the monster shrugs it off; try again within the Opening window.
5. **If no Opening was created:** monster recovers and enters next telegraph cycle. Player damages themselves by tanking.
6. **Loop** until player or monster is dead.

### 1-HP monsters preserved
Every non-boss monster has exactly 1 HP. The hard part is **landing the hit**, not grinding HP. The Dice Tray means even with 5 attack dice the player has about an 60% chance of landing a kill on each swing — the rest of the time they swing and miss. This creates a distinctive "whiff and then BLAM" rhythm.

### Dice Tray as combat spectacle
- The Dice Tray lives on the bottom of the HUD.
- On each attack, the Attack Dice physically animate rolling.
- Each 6 that lands pulses gold, screen-shakes briefly, and contributes to damage.
- When the player upgrades Attack Dice (Big Sword: +1, Big Axe: +2, Spiky Armour: +2), new dice visually slot in with a "ka-chunk" animation.

### Enemy variety
11 monsters plus Dracular. Each has a distinct silhouette, telegraph pattern, and signature animation beat:
- **Orc (1 AD)** — slow overhead chop, huge telegraph window
- **Wolf (2 AD)** — fast lunge, tight parry window
- **Skeleton (3 AD)** — clattering sword swing
- **Evil Warrior (4 AD)** — shield-bash into riposte
- **Devil Bat (5 AD)** — aerial dive, parry from above
- **Cyclops (6 AD)** — ground slam with tremor effect
- **Dark Elf (7 AD)** — fast dagger flurry, multi-hit combo
- **Skeleton Lord (8 AD)** — summons minor ranged shots
- **Wizard (9 AD)** — projectile magic, dodge instead of parry
- **Demon (10 AD)** — massive scythe sweep, arena-wide telegraph
- **Dracular (9 AD / 9 DD)** — boss, see Section 9.6

### The Nine Dice Duel (Dracular boss)
The boss fight is an RT-NATIVE invention for the climax. Three phases:

**Phase 1 — "The Gambler" (first 30 seconds):**
Dracular sits on a throne, visible dice tray on his side rolling constantly. He picks up dice from the table and throws them as magical bolts at the player. Player dodges or parries each dice-bolt. Every successful parry *adds a die to Dracular's incoming attack*, increasing risk but creating Openings. Dracular takes no damage in phase 1 — he is protected by 9 defence dice that the player must "burn through" before phase 2.

**Phase 2 — "The Duelist" (next 30 seconds):**
Dracular steps off his throne and engages in melee. His telegraphs are fast (300ms parry window, not 200ms). Each player-landed 6 reduces his defence dice by 1 (visible on screen). When his dice count reaches zero, phase 3 begins.

**Phase 3 — "The Ink Storm" (final 30 seconds):**
Arena darkens; ink swirls. Dracular becomes vulnerable and desperate — high attack but low defence. A single clean parry + full Attack Dice Tray roll with even one 6 can kill him. This is the climactic "land the finishing blow" moment.

### Feel problems mitigated
- **Death feels fair:** every hit that kills a player comes from a monster they could see telegraphing. No ambushes.
- **1-HP kills don't feel cheap:** because the hit is *hard to land* (must be an Opening + must roll a 6), landing one feels earned.
- **Dice don't feel rigged:** the tray is literally visible; players see every die roll.

### No PvP
Single-player only. No competitive mode. The game is not designed to be balanced between two human opponents.

---

## 10. Economy & Progression

From `analysis/EconomyModel.md`.

### Resources
- **Hearts (17 max).** Lost to unblocked damage. Healed by potions 1:1.
- **Treasure (unbounded).** Gained from Treasure rooms (auto-pickup). Spent at Shop Shrines.
- **Potions (unbounded stockpile, 1-room delay on bought).** Consumed via hotkey to heal 1 heart each.

### Shop Shrines
- Diegetic altars in dedicated **Shop Shrine rooms** (rare, ~1 per 8 rooms on average, seeded per run).
- Approach the altar → interact → radial menu pops up with the 9 shop items.
- Item catalog (same as source, same costs, same effects, same slot exclusivity):
  - 1T Buckler (+1 Defence Die) — XOR Shield
  - 1T 1 Potion
  - 2T Shield (+2 Defence Dice) — XOR Buckler
  - 2T 3 Potions
  - 3T Big Sword (+1 Attack Die) — XOR Big Axe
  - 3T 6 Potions
  - 4T Big Axe (+2 Attack Dice) — XOR Big Sword
  - 5T Spiky Armour (+2 Attack Dice, +1 Defence Die)
  - 6T Magical Armour (+5 Defence Dice) — XOR Spiky Armour
- Bought potions enter a "pending" counter and become "available" when the player exits the current room.
- The Shrine room is combat-safe: no monster spawns.

### Match end conditions
- **Victory:** Dracular defeated in the End room.
- **Defeat (death):** 0 hearts.
- **Defeat (blocked):** every room king-adjacent to the current cell is visited and none is End. This is the only case where the reachability warning *should* have saved the player — if they ignored it, they lose.
- **No timer-based defeat.**

### Match length
- Median 20 min; 80th percentile 12–35 min.
- Fastest possible speedrun (straight-line through middle column): ~11 min (not feasible under-geared).
- Explorer completion (clear all 90 cells): ~35 min.

### No catch-up
- The source has zero catch-up mechanics. We preserve this. A bad start compounds into a bad run. But you can always restart — the game respects your time with quick reset.

### Metaprogression
- **No permanent stat buffs between runs.** Each run starts at 1 AD / 1 DD / 17 hearts.
- **Unlocks:** bestiary entries (one per monster type first killed), cosmetics (dice skins, character skins, dungeon palettes), achievements.
- **Daily seed challenge** tracked locally; no server leaderboard in v1.

### Scoring (optional, off by default)
- Speed run timer
- Parries landed
- Treasure total
- "Clean run" bonus for hitting no more than X damage
- These feed achievements but are hidden from the main UI unless toggled on.

---

## 11. Preserved Board Game DNA

Eight things we kept intact from the source:

1. **The 9×10 grid.** Literal, preserved, unchanged.
2. **King-adjacency (8-way).** Preserved as doorway connectivity.
3. **The 10 level tables + boss table.** Preserved verbatim as the pre-seeding data source. Players can view them in the Codex.
4. **The shop: 9 items, 3 slots, exact costs.** Every item from the source is in the game with identical cost and effect.
5. **17 max HP.** Preserved as 17 hearts.
6. **1-HP monsters, lethal combat.** Preserved through the parry-gate mechanic.
7. **Dracular (9 AD / 9 DD).** Preserved as the final boss, same stats, same name.
8. **Count-6s on d6 pools.** Preserved as the Dice Tray — the on-screen visualization of the exact math the board game uses.

**Why these matter:** Each of these is a load-bearing identity element. Removing any of them would make this "inspired by Solo Dungeon Bash" instead of "Solo Dungeon Bash in real-time." We refused to remove them.

---

## 12. What Was Dissolved

Things that existed in the source but do not exist in RT, and why:

1. **Manual dice rolling.** In the source the player physically picks up dice and rolls them. In RT, the Dice Tray rolls automatically on each attack/defence — but remains visible, so the *feeling* of dice-rolling survives. The tactile act itself dissolves.
2. **Pen-and-paper tracking.** The source has the player draw paths, mark squares, track stats on paper. Dissolved — the game auto-tracks everything. Compensated by the map overlay and ink-trail visualization.
3. **The strict 7-step turn sequence.** In the source these are literal numbered steps. In RT these become concurrent systems that co-exist continuously. Step 1 is locomotion, step 2 is auto-reveal, steps 3-4 are auto-pickup, step 5 is combat, step 6 is potion hotkey, step 7 is Shop Shrine interaction.
4. **Literal die-roll for room content.** In the source, each room's contents are rolled on entry. In RT, contents are pre-seeded at run start. (The randomness still feels stochastic to the player because the seed is hidden.)
5. **"Pick-up-dice" agency.** In the source rolling dice is an action. In RT it's automatic because pausing for every roll would destroy combat flow.
6. **The 4-step combat sub-loop.** In the source, combat explicitly has Monster Attack → Player Defend → Player Attack → Monster Defend. In RT these become beats inside a continuous parry-riposte cycle — monsters attack first (preserving initiative), players parry or take damage (player defence), parries create openings for player attack (player attack), and monsters block via 9 DD only during the Dracular fight (monster defence). The 4 steps are still there, just compressed into beats.

---

## 13. New RT-Native Mechanics

Mechanics we had to invent for the RT version to work. Each is flagged as `RT-NATIVE`.

1. **`RT-NATIVE`: Parry timing window.** 200ms window. No analog in source.
2. **`RT-NATIVE`: Dodge roll with i-frames.** 350ms i-frames + 400ms recovery + 700ms cooldown. Source has no interrupt or evasion.
3. **`RT-NATIVE`: Monster Guard state.** Source enemies die in 1 unblocked hit immediately; in RT they have a Guard that blocks all hits until an Opening is created via parry.
4. **`RT-NATIVE`: Opening window.** 1 second of vulnerability after a successful parry. Replaces the "player attack step" of source combat.
5. **`RT-NATIVE`: Dice Tray UI.** A visible animated dice panel. Source has dice; this makes them *always visible and always meaningful*.
6. **`RT-NATIVE`: Monster telegraph animations.** Every enemy has a ~600ms wind-up with a red flash at the parry window. Source has no telegraphs.
7. **`RT-NATIVE`: Hit-stop and juice.** 80ms hit-stop on landed blows, particle VFX, screen shake. Source has none.
8. **`RT-NATIVE`: Doorway seals.** Physical enforcement of no-revisit rule via collapsing ink chains. Source uses pencil marks.
9. **`RT-NATIVE`: Reachability warning HUD.** Live BFS warns before a trapping move. Source relies on player awareness.
10. **`RT-NATIVE`: Room camera transitions.** ~300ms pan between rooms. Source has no camera.
11. **`RT-NATIVE`: Shop Shrine rooms.** Diegetic shop as a dedicated room type, not a menu pause. Source has a between-turn menu.
12. **`RT-NATIVE`: Dracular Nine Dice Duel.** Three-phase boss fight. Source has a single dice duel.
13. **`RT-NATIVE`: Bestiary and cosmetic metaprogression.** Source has no between-run progression.
14. **`RT-NATIVE`: Map overlay toggle (V).** Source's "map" is just the paper sheet; in RT we bring it back on demand.
15. **`RT-NATIVE`: Daily seed challenge.** Source has no seeding (random per run).

All RT-NATIVE mechanics are justified by: "without this, RT combat or pacing would be broken." None are added for their own sake.

---

## 14. Minimum Playable Prototype

The smallest version of the game that could be built in a 72-hour game jam to prove the core loop works:

### Scope
- **One room type.** Plain octagonal chamber.
- **Three monster types.** Orc (1 AD), Wolf (2 AD), Cyclops (6 AD).
- **No shop.** Starting inventory only.
- **No dungeon grid.** Linear sequence of 5 rooms → boss.
- **No bestiary, no cosmetics.** Just the core loop.
- **Placeholder art.** Circles, rectangles, colored shapes. No hand-drawn style yet.
- **SFX only.** No music.

### Must work
- Character moves with WASD
- Walks through doorway → camera pans → next room appears
- Monster telegraphs attack, red flash at parry window
- Left-click attacks; right-click parries; spacebar dodges
- Dice Tray rolls on attack and visibly shows 6s
- Monsters die when a 6 is rolled during Opening
- Player loses hearts on unblocked hits
- Reaching room 5 triggers a simplified Dracular fight
- Death and victory screens restart to room 1

### Must NOT have (out of MPP scope)
- Shop / items / progression
- Procedural dungeon
- Hand-drawn art
- Full bestiary
- Metaprogression
- Audio beyond basic feedback
- Save/load
- Menus beyond start and end

### Success criteria
A friend walks up to the laptop, plays one run (win or loss), and their takeaway is *"I want to play that again."* If they don't say that, the core loop is broken and we iterate before scaling up.

### Estimated effort
72 hours of work for one full-stack developer in Godot 4 or Phaser. Realistic scope. This IS the Wave 6 "core loop prototype" and maps directly to `prototypes/` Prompt 1.

---

## 15. What's Next in the Pipeline

After this RTGDD:
- **`design/FaithfulnessAudit.md`** — formal side-by-side of source vs RT mechanics, with a % score.
- **`design/PrototypePrompts.md`** — three paste-ready prompts for AI prototyping tools.
- **`revised/`** — eight revised RuleForge documents adapted for the RT context.
- **`architecture/`** — full technical system design, multiplayer design (skipped here since solo), engine choice.
- **`balance/`** — simulation framework, tuning knobs, playtest scripts.
- **`assets/`** — art and audio pipeline specification.
- **`prototypes/`** — extended prototype prompts including networking (N/A) and AI/NPC.
- **`deployment/`** — roadmap and risk register.

---

## Appendix A — Dependencies on Wave 1 Analyses

| Section | Source Analysis |
|---|---|
| Section 5 (Temporal) | `analysis/TemporalMap.md` |
| Section 6 (Spatial) | `analysis/SpatialModel.md` |
| Section 7 (Actions) | `analysis/AgencyModel.md` |
| Section 8 (Info/UI) | `analysis/InfoArchitecture.md` |
| Section 9 (Combat) | `analysis/ConflictModel.md` |
| Section 10 (Economy) | `analysis/EconomyModel.md` |
| Section 2/3/4/11 | `analysis/GenreCrystallization.md` |

## Appendix B — Glossary

| Term | Meaning |
|---|---|
| **Roll-and-Parry** | The core combat system of Solo Dungeon Dash. Parry creates an Opening, Opening lets you attack, attack rolls the Dice Tray. |
| **Dice Tray** | The visible on-screen panel showing the player's current Attack Dice and Defence Dice count as animated d6s. |
| **Opening** | A 1-second vulnerability window on an enemy, created by a successful parry. |
| **Guard** | The default monster state in which hits do nothing. |
| **Telegraph** | Visible wind-up animation on an enemy before an attack, marking the parry window. |
| **Shop Shrine** | A dedicated safe-room with a diegetic altar for spending treasure. |
| **Seal** | The animation that closes a doorway behind you, preventing revisit. |
| **Reachability warning** | The HUD indicator that fires when your next move would strand you from End. |
| **Ink trail** | The visual record of your path on the map overlay. |
| **RT-NATIVE** | A mechanic added for the RT version that has no analog in the source board game. |
