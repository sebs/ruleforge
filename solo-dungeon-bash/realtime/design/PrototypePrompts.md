# Prototype Prompts — Solo Dungeon Dash (RT)

> RealTimeForge Stage RT-9
> Three paste-ready prompts for AI coding/prototyping tools. Each prompt is self-contained and specifies engine, dimension, input, and scope.
>
> Tooling targets: **Claude / Cursor / Bolt / Lovable / Rosebud.ai / v0.** All three prompts target **Godot 4 (GDScript)** as the primary engine, with a web-prototype fallback in HTML + Canvas + vanilla TypeScript where the tool doesn't support Godot.

---

## Prompt 1 — Core Loop Prototype

> **Goal:** A single-player playable prototype proving that the 30-second core loop of Solo Dungeon Dash feels good. One room at a time, one enemy at a time, parry-and-attack with a visible dice tray. No polish, no procedural dungeon, no shop. Just the core verb loop.
>
> **Engine:** Godot 4.x with GDScript (or Godot 4.x C# if preferred).  
> **Fallback:** HTML5 + Canvas + TypeScript single-file prototype.  
> **Dimension:** **2D top-down.**  
> **Camera:** fixed, framing one room at a time.  
> **Input:** mouse + keyboard (WASD movement, left-click attack, right-click parry, spacebar dodge).  
> **Scope:** ~24 developer-hours (a weekend). Placeholder art only.
>
> ---
>
> **Scene structure:**
> - **Main Scene** loads a single `Room` node with the player and one enemy.
> - **Player Scene** — a Sprite2D (colored square placeholder), CharacterBody2D physics body, attached script for movement, attack, parry, dodge state machine.
> - **Enemy Scene** — an orc enemy (colored square placeholder), state machine: `Idle → Telegraphing → Attacking → Guard → Opening → Recovering`.
> - **HUD CanvasLayer** — displays hearts (top-left), dice tray (bottom-center), tooltip text (top-right).
> - **DiceTray Scene** — UI element that displays N d6 dice (where N = player Attack Dice or Defence Dice). On `roll()` call: each die sprite animates for 400ms, lands on a random 1–6 face, 6s pulse gold.
>
> ---
>
> **Gameplay spec — implement exactly this:**
>
> 1. **Player state:** starts with 17 hearts, 3 Attack Dice, 2 Defence Dice (pre-upgraded for demo interest). Position: middle of room.
>
> 2. **Enemy state:** starts with 1 HP (always), 4 Attack Dice (Orc Warrior). Position: 6 meters from the player.
>
> 3. **Movement:** player WASD moves in 4 directions at 4 m/s (roughly 240 pixels/sec at 60-pixels-per-meter scale). No acceleration. Hit detection: AABB collision between player square and enemy square.
>
> 4. **Enemy AI loop:**
>    - Enemy starts in `Idle` state for 500ms after spawn.
>    - Enters `Telegraphing` state — enemy sprite flashes red, a red glow appears around the enemy over 600ms.
>    - Between 400ms–600ms into the telegraph, the enemy is in **parry window** (yellow ring appears on enemy).
>    - At 600ms, enemy enters `Attacking` state: rolls its 4 Attack Dice, counts 6s as hits.
>    - If player is in the parry window state when `Attacking` begins → enemy enters `Opening` state (stays still, visibly staggered, blue glow, 1000ms).
>    - If player is not in parry window → damage applied (see step 5), enemy enters `Guard` state (300ms), then `Telegraphing` again.
>    - Enemy loops forever until killed.
>
> 5. **Damage application:**
>    - Enemy rolls 4 Attack Dice → count 6s → `monster_hits`
>    - Player rolls `player.defence_dice` → count 6s → `player_blocks`
>    - `unblocked = max(0, monster_hits - player_blocks)`
>    - `player.hearts -= unblocked`
>    - Red flash on player + small screen shake
>    - If `player.hearts <= 0` → show "DEFEAT" text + pause
>
> 6. **Player attack:**
>    - Left-click triggers attack animation (150ms swing).
>    - Attack only registers hit if enemy is in `Opening` state AND player is within 2 meters.
>    - If hit registers → roll player's Attack Dice on the Dice Tray → if any 6 → enemy dies → show "VICTORY!" text + pause → ~300ms later respawn a new enemy at the original enemy position.
>    - If no 6 on the roll → enemy stays in Opening (if time remains) or returns to Guard.
>
> 7. **Player parry:**
>    - Right-click triggers parry animation (100ms held parry pose, 200ms window).
>    - If parry is active during the enemy's parry window → successful parry, enemy enters Opening, cyan flash on player + hit-stop 80ms.
>    - If parry is active outside the window → miss, small "whiff" sound.
>
> 8. **Player dodge:**
>    - Spacebar triggers dodge in the direction of current movement input.
>    - Dodge lasts 350ms (i-frames: player takes no damage during dodge).
>    - 400ms recovery after dodge ends (player cannot act).
>    - 700ms cooldown before next dodge.
>
> 9. **Dice Tray UI:**
>    - Persistent at the bottom of the screen.
>    - Shows player's current Attack Dice as d6 slots on the left, Defence Dice on the right, in different colors.
>    - On `roll()` call: all dice in that tray animate (spin + bounce over 350ms), then land on face values. Each die that lands on 6 pulses gold for 500ms.
>    - Dice faces can be drawn as simple rectangles with 1–6 pips.
>
> 10. **HUD hearts:**
>     - 17 heart icons (simple red squares) across the top.
>     - When player takes damage, the corresponding hearts go grey with a shake animation.
>
> 11. **Tutorial tooltips:**
>     - On game start: "WASD to move. Right-click to parry when enemy glows yellow. Spacebar to dodge."
>     - After first successful parry: "Now attack during the blue glow (Opening)!"
>     - Remove after ~10 seconds of player playing.
>
> 12. **Fail states:**
>     - Player hearts ≤ 0 → "DEFEAT" screen. Press R to restart.
>     - On restart: reset player hearts, spawn new enemy.
>
> ---
>
> **Art spec:**
> - Placeholder squares: player = green square, enemy = red square, dice tray background = dark parchment-colored rectangle, dice = tan squares with black pips.
> - NO actual art assets needed. Placeholder shapes only.
> - Hearts: small red square glyphs.
>
> **Audio spec:**
> - One SFX per action: swing-whoosh (attack), clang (parry success), swish (dodge), thud (player hit), dice-clatter (dice tray roll), enemy-death-squelch.
> - Can use free assets from [freesound.org](https://freesound.org) or placeholder beeps.
>
> ---
>
> **Acceptance criteria:**
> - [ ] Player can move, attack, parry, dodge with responsive input (< 50ms input latency)
> - [ ] Enemy telegraph is readable — a first-time player can see the red glow and know something's coming
> - [ ] Parry window is landable — a player who understands the rule can consistently parry 3 out of 5 attempts
> - [ ] Killing an enemy feels satisfying — dice tray rolls, sixes pulse gold, enemy visibly dies
> - [ ] Losing all hearts ends the run and allows restart
> - [ ] The full loop from "enemy spawns → parry → opening → attack → dice roll → kill" can complete in under 5 seconds when played well
> - [ ] No crashes, no stuck states
>
> ---
>
> **File structure (Godot 4):**
> ```
> project.godot
> main.tscn          — Main scene with Room + HUD
> main.gd            — Main script; respawns enemies
> scenes/
>   player.tscn
>   player.gd
>   enemy.tscn
>   enemy.gd
>   dice_tray.tscn
>   dice_tray.gd
>   hud.tscn
>   hud.gd
> assets/
>   sfx/*.wav
> ```
>
> **Do not implement:** multiple enemies, shop, multiple rooms, procedural dungeon, hand-drawn art, music, save/load, menus beyond start/restart, bestiary, cosmetics. Those belong to later prototypes.

---

## Prompt 2 — Conflict Prototype (Multi-Enemy Parry Arena)

> **Goal:** Test the Roll-and-Parry combat system under more varied stress. Three enemy types with different telegraph timings. Combat feel polish. NO economy, NO shop, NO dungeon. Just "this enemy feels unique from that enemy" and "the combat system scales."
>
> **Engine:** Godot 4 GDScript (preferred) or Unity 2D (alternative).  
> **Fallback:** Phaser 3 + TypeScript.  
> **Dimension:** 2D top-down.  
> **Camera:** fixed, framing an arena (larger than Prompt 1's room, about 14×14 meters).  
> **Input:** mouse + keyboard PRIMARY, gamepad SECONDARY (Godot has native gamepad support — wire it up).  
> **Scope:** ~40 developer-hours (one week).
>
> ---
>
> **Scene structure:**
> - **Arena Scene:** A rectangular arena with visible walls. One player, an enemy spawner.
> - **Enemy spawner:** waves of enemies, 1–3 at a time, chosen from 3 types. Next wave spawns when the current wave is dead.
> - **Player Scene:** as Prompt 1, with all combat verbs functional and juicy.
> - **Three enemy types:**
>   - **Orc (slow, simple):** 1 AD, 1200ms telegraph, 300ms parry window. Overhead chop animation (simple sprite rotation).
>   - **Wolf (fast, aggressive):** 2 AD, 500ms telegraph, 150ms parry window. Lunge animation (quick dash at player).
>   - **Cyclops (heavy):** 6 AD, 1500ms telegraph, 250ms parry window. Ground-slam animation (particle burst radius).
> - **Dice Tray:** same as Prompt 1.
> - **HUD:** 17 hearts + dice tray + wave counter + enemy count.
>
> ---
>
> **Combat feel spec:**
>
> 1. **Hit-stop:** On landed player hit, freeze the game for 80ms (not just the enemy — the whole scene). This sells the impact.
>
> 2. **Screen shake:** On player hit (damage taken) → shake camera 4 pixels for 200ms. On killed enemy → shake camera 6 pixels for 150ms. On boss/heavy enemy hit → shake camera 10 pixels for 300ms.
>
> 3. **Particle effects:** On parry success → cyan sparks burst. On enemy death → ink-splash particles. On dodge → dust particles.
>
> 4. **Dice tray juice:** Each die that rolls a 6 should pulse gold + grow 20% larger + play a chime. All 6s count and stack visually.
>
> 5. **Time scale on critical moments:** When the player lands a 6-filled kill roll on a high-AD enemy, slow time to 0.3× for 300ms. It's cheap and sells the moment.
>
> 6. **Audio layering:**
>    - Base SFX layer: swings, hits, steps
>    - Impact layer: hit-stop stinger, kill stinger
>    - Ambient layer: distant dungeon reverb
>    - Dice layer: physical dice clatter
>    - (5 layers max; don't over-engineer)
>
> 7. **Gamepad support:**
>    - Left stick: movement
>    - Right trigger: attack
>    - Left trigger: parry
>    - A / South button: dodge
>    - B / East button: heal potion (hotkey - prototype has 3 starting potions)
>    - Remappable via a simple config file
>
> 8. **Parry window scaling:**
>    - Add a difficulty slider in the main scene (or a key toggle: 1/2/3 for Easy/Normal/Hard) that scales the parry window 1.5× / 1.0× / 0.7×.
>    - This is for prototype testing to find the "right" window size.
>
> ---
>
> **Enemy AI spec:**
>
> Each enemy type has slightly different AI:
>
> - **Orc:** walks toward player at 2 m/s. When within 2 meters, begins Telegraphing.
> - **Wolf:** walks toward player at 3.5 m/s. When within 5 meters, LUNGES 3 meters toward the player over 300ms, then Telegraphs immediately upon arrival.
> - **Cyclops:** walks toward player at 1.5 m/s. Never backs off. Telegraphs at 3-meter range. Telegraphs are radial (the ground-slam creates a 4-meter damage radius, not a directional attack).
>
> All enemies respect the parry window / Opening state machine from Prompt 1.
>
> ---
>
> **Wave structure:**
>
> - Wave 1: 1 Orc
> - Wave 2: 2 Orcs
> - Wave 3: 1 Orc + 1 Wolf
> - Wave 4: 1 Cyclops
> - Wave 5: 2 Wolves + 1 Orc
> - Wave 6: 2 Cyclops
> - Wave 7+: random mix, scaling enemy count
>
> Between waves: 2-second pause, "Wave N cleared" text.
>
> ---
>
> **Potions (for prototype only):**
>
> - Player starts with 3 potions.
> - B / Q key drinks a potion → +1 heart, -1 potion, 500ms animation where player is briefly invulnerable.
> - Potions do not regenerate between waves.
>
> ---
>
> **Acceptance criteria:**
>
> - [ ] All three enemy types feel mechanically distinct in ≤ 2 minutes of play
> - [ ] A player can make "great parry" moments on all three enemy types
> - [ ] Gamepad input feels as responsive as mouse+keyboard
> - [ ] Hit-stop and screen shake combined produce "this feels good to play"
> - [ ] Dice Tray cascades on multi-six rolls produce a "WHOA" moment at least once per 10 minutes
> - [ ] Parry window slider allows tuning the difficulty experimentally
> - [ ] No crashes through 7 waves of play
> - [ ] Average fight duration (1 enemy, player competent): 3–6 seconds
> - [ ] Average fight duration (1 Cyclops, player competent): 8–12 seconds
>
> ---
>
> **Do not implement:** procedural dungeon, shop, doorways between rooms, boss, metaprogression, bestiary, real hand-drawn art. Placeholder art (colored squares / simple sprites) is fine.
>
> ---
>
> **File structure:**
> ```
> scenes/
>   arena.tscn
>   enemies/orc.tscn
>   enemies/wolf.tscn
>   enemies/cyclops.tscn
>   vfx/hit_spark.tscn
>   vfx/ink_splash.tscn
>   vfx/dust.tscn
> scripts/
>   enemy_base.gd
>   orc.gd
>   wolf.gd
>   cyclops.gd
>   wave_spawner.gd
>   combat_fx.gd
>   gamepad_config.gd
> ```

---

## Prompt 3 — Full Vertical Slice (Pitch Demo)

> **Goal:** A complete vertical slice that demonstrates the full Solo Dungeon Dash pitch in one 5-to-10-minute playthrough. One small dungeon (3 rooms + Shop Shrine + boss), full combat loop, real hand-drawn visual style, working shop, working boss fight, working win/loss screens. This is the thing you show publishers and playtesters.
>
> **Engine:** Godot 4.x GDScript (primary).  
> **Fallback:** Unity 2D (if the team is already Unity-native).  
> **Dimension:** 2D top-down.  
> **Camera:** per-room framing with ~300ms pan transitions on doorway crossings.  
> **Input:** mouse + keyboard + gamepad, remappable, Steam Deck verified.  
> **Scope:** ~160 developer-hours (one month) for 1 programmer + 1 artist part-time.
>
> ---
>
> **Scene list:**
>
> 1. **MainMenu** — Title "Solo Dungeon Dash", Play / Tutorial / Settings / Credits buttons
> 2. **TutorialRoom** — scripted micro-run teaching movement, parry, dodge, dice tray, shop. Takes 3-5 minutes to complete.
> 3. **Dungeon** — 3 connected rooms + 1 Shop Shrine room + 1 Boss arena. Player can choose path through the small grid.
> 4. **EndScreen** — Victory or Defeat with run summary, "Play Again" button
>
> ---
>
> **Scenes 1: Main Menu**
>
> - Background: hand-drawn parchment with ink sketches (dungeon silhouette, dice, sword)
> - Title: "Solo Dungeon Dash" in handwritten ink-brush font
> - Buttons: PLAY, TUTORIAL, SETTINGS, CREDITS
> - Music: ambient dungeon loop with occasional dice-clatter SFX
> - Transition: ink-fade when entering dungeon
>
> ---
>
> **Scene 2: Tutorial Room**
>
> Guided 5-minute experience. See `RT-OnboardingDesign.md` for detailed script.
>
> Key beats:
> - Room 1: learn WASD, walk to door
> - Room 2: learn parry on scripted training dummy Orc
> - Room 3: learn Dice Tray through a successful attack
> - Room 4: learn Shop Shrine with an affordable Buckler
> - Room 5: two-enemy challenge (combines all learned mechanics)
> - Exit: dramatic reveal of the full dungeon map
>
> All UI copy painted on the floor in hand-drawn ink style.
>
> ---
>
> **Scene 3: Dungeon (the actual slice)**
>
> A 3×3 miniature dungeon + Start + End:
>
> ```
> +---+---+---+
> |   |   |   |
> +---+---+---+
> |   | ⛧ |   |   ⛧ = Shop Shrine
> +---+---+---+
> |   |   |   |
> +---+---+---+
>        |
>      START
> ```
>
> - 9 rooms arranged 3×3 + Start below + End above.
> - Start contains no enemies.
> - End contains Dracular.
> - 8 rooms can each be: Empty, Treasure, Potion, Monster, Shop Shrine. Pre-seeded from a hardcoded seed for reproducibility.
> - Player must navigate from Start to End using king-adjacency.
> - Rooms are octagonal chambers with up to 8 doors (here 4–6 depending on grid position).
> - Doorway transitions: player walks into a door, camera pans to new room over 300ms, door closes behind with animation.
> - The Shop Shrine has one item for sale: Big Sword (+1 AD, 3 Treasure).
> - The player should accrue enough treasure to afford the sword by the time they reach the Shrine.
>
> **Monsters in the slice:** Orc, Wolf, Skeleton (each from the source level tables).  
> **Boss:** Dracular (simplified Nine Dice Duel — only 1 phase for the slice, 30 seconds instead of 90 seconds).
>
> ---
>
> **Scene 4: End Screen**
>
> - Victory: "You defeated Dracular!" + ink splash animation + run stats (rooms visited, parries landed, treasure earned, time elapsed).
> - Defeat: "You Died" or "Blocked!" + ink fade-out + run stats.
> - Buttons: Play Again (new seed), Back to Menu.
>
> ---
>
> **Visual style:**
>
> - Hand-drawn ink-on-parchment throughout. All sprites should LOOK hand-drawn (wobbly lines, crosshatch shading, ink blots).
> - Color palette: beige/cream background, black ink, gold treasure, blue potions, red danger, green current cell indicator.
> - Typography: Caveat or Homemade Apple (Google Fonts, free) for labels; Inter for numbers.
> - UI chrome: bordered with wobbly ink lines, dog-eared corners, handwritten "Health", "Treasure", "Potions" labels.
> - Camera: slight tilt/wobble (~1px perlin noise) to simulate a handheld sketch feel.
>
> **Art assets needed:**
> - Player character (1 idle, 1 walk, 1 attack, 1 parry, 1 dodge, 1 hurt)
> - 3 monster types (Orc, Wolf, Skeleton) × (1 idle, 1 telegraph, 1 attack, 1 death) each
> - Dracular (1 idle, 1 telegraph, 3 attack variants, 1 death) — higher detail than mobs
> - Room tileset (octagonal chamber + 8 door variants)
> - UI: 17 heart icons, dice tray background, dice faces (6 sprites), buttons, shop altar
> - Particles: ink splash, dice sparkle, parry flash, dust
> - Main menu parchment scene
>
> Total art asset count: ~80 sprites. Achievable by one artist in 2 weeks part-time.
>
> ---
>
> **Audio:**
>
> - Main menu music: calm ambient dungeon theme (60 seconds, loopable)
> - In-dungeon ambient: quieter version of menu theme, crossfades between rooms
> - Boss music: intensified ambient with percussion, 90-second loop
> - SFX: parry-clang, hit-thud, dodge-swish, dice-clatter, coin-shower, potion-glug, footstep-stone, door-open, door-seal, ink-splash, victory-fanfare, defeat-dirge
> - Total: ~20 SFX + 3 music tracks. One-week audio pass by one sound designer.
>
> ---
>
> **Shop Shrine scene detail:**
>
> - Player walks into the Shrine room
> - Background music fades to a quieter "shrine" variant with chimes
> - An ink-drawn altar is at the center of the room with a glowing book
> - Player interacts (E / Y) → radial menu pops up showing items
> - Items: Big Sword (3 T) highlighted, others greyed and disabled for the slice
> - Hover over item → tooltip with description
> - Click to buy → -3 Treasure, +1 AD (visible as a new die in the Dice Tray), sparkle VFX
> - Exit the Shrine by walking out the back door
>
> ---
>
> **Dracular fight (simplified one-phase for the slice):**
>
> - Player enters the End room
> - Confirmation dialog: "Enter End square? You cannot return."
> - On confirm: door seals, music swells, camera zooms out slightly
> - Dracular walks in from the top of the arena
> - Dracular telegraphs are 800ms (tighter than normal mobs)
> - Dracular has 9 Defence Dice (visible as a dice cluster on his side)
> - Each successful player hit that rolls a 6 reduces Dracular's defence cluster by 1
> - Each parry failure rolls Dracular's 9 Attack Dice against the player's Defence tray
> - Fight lasts ~30 seconds for a competent player
> - On victory: ink splash, victory music, hearts text, cut to end screen
> - On defeat: standard death, cut to end screen
>
> ---
>
> **Acceptance criteria:**
>
> - [ ] A friend can play through the tutorial in ≤ 5 minutes without developer intervention
> - [ ] A friend can then play a full dungeon run (including boss) in ≤ 10 minutes
> - [ ] Average first-run win rate (3 friends, no prior play): 0–33% (should feel hard but fair)
> - [ ] Average third-run win rate (after 2 losses): 50–75% (should feel learnable)
> - [ ] All art assets render at hand-drawn style, not placeholder
> - [ ] All audio cues are present and non-broken
> - [ ] End screen displays accurate run statistics
> - [ ] Keyboard AND gamepad input both work without reconfiguration
> - [ ] Steam Deck verified profile exists
> - [ ] The demo fits in a 200MB download (including art + audio)
> - [ ] Zero crashes in 20 consecutive playthroughs
>
> ---
>
> **What this slice proves to a publisher:**
>
> 1. The core loop is fun (proved in Prompt 1 → here in a full dungeon)
> 2. The combat system has depth and distinct enemy feel (proved in Prompt 2 → here with 3 types)
> 3. The hand-drawn aesthetic is cohesive and distinctive (new to this prompt)
> 4. The shop system integrates naturally into room flow (new to this prompt)
> 5. The boss fight delivers a climactic moment (new to this prompt)
> 6. The complete player loop from menu to victory/defeat works (new to this prompt)
> 7. The game is ready for content scaling — add more enemies, more rooms, more polish — without architectural changes
>
> ---
>
> **Do not implement:**
> - Full 9×10 dungeon (use 3×3 for the slice)
> - All 11 monster types (use 3)
> - All 9 shop items (use 1)
> - Multi-phase Dracular (use 1 phase)
> - Procedural daily seed (use hardcoded seed)
> - Bestiary, achievements, cosmetics
> - Save / resume mid-run
> - Localization
> - Mobile touch input (PC + Steam Deck only for this slice)
> - Settings beyond volume and fullscreen toggle
>
> These all belong to the Alpha phase, not the Vertical Slice.

---

## Notes on Using These Prompts

- **Prompt 1** is a proof-of-concept weekend hack. Build it first, prove the combat feels good, then escalate.
- **Prompt 2** layers on multi-enemy variety and feel polish. Build it second.
- **Prompt 3** is the pitch demo. Do not build Prompt 3 until Prompt 2 is done.
- All three prompts share the same core architecture — player, enemy state machines, dice tray, HUD — so code written for Prompt 1 is reused in 2 and 3.

### Engine decision rationale

**Godot 4** is the recommended engine because:
- Free, open source, no royalties
- Native 2D support (not a 3D engine doing 2D)
- Small export binaries (~50 MB typical)
- Cross-platform export (Windows, macOS, Linux, web, Android, iOS)
- Steam Deck friendly
- GDScript is easy for small teams, C# option exists for larger teams
- Active community for 2D indie roguelites

Unity 2D is a close second and recommended if the team is already Unity-native. Phaser + TypeScript is the web-first fallback for teams who want HTML5 distribution.

### Where these prompts pair with other files

- `analysis/AgencyModel.md` — detailed parry/dodge/attack timing rationale
- `analysis/ConflictModel.md` — detailed combat FSM rationale
- `revised/RT-BalanceSheet.md` — detailed tunable parameter list with safe ranges
- `architecture/SystemArchitecture.md` — the full technical spec Prompt 3 should match
- `assets/AssetPipeline.md` — the full art/audio spec Prompt 3 should match
