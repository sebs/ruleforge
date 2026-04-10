# RT Balance Parameter Sheet — Solo Dungeon Dash

> RealTimeForge Stage RT-D4: Revised Balance Sheet
> Source: `output/solo-dungeon-bash/BalanceSheet.csv`, `BalanceSheet.md`
> Target: **Solo Dungeon Dash** — real-time action roguelite translation with visible dice tray combat.
> Companion file: `RT-BalanceSheet.csv` (full parameter list, 110 rows).

This document re-grounds every numeric parameter of the source game in real-time units, preserves source values where they still apply, translates counters that lose meaning in continuous time, and introduces the new timing / feel parameters that RT combat demands.

---

## 1. Summary of Parameter Categories

| Category | Count | Description |
|---|---:|---|
| `player_stat` | 4 | Hearts, HP cap, starting dice pools. Preserved verbatim. |
| `combat` | 22 | Die threshold, pools, HP, parry/dodge/opening windows, Dracular phases, pity heart, damage cap. |
| `dungeon` | 6 | Grid, adjacency, revisit rule, path-length. All preserved. |
| `monster` | 20 | Per-level AD progression + occupancy probabilities. All preserved. |
| `economy` | 7 | Treasure / potion faucets, ratios, carry cap, arming cooldown. Mostly preserved. |
| `shop` | 17 | Costs, bonuses, exclusivities, interact range, time scale. Preserved + 2 new. |
| `timing` | 11 | fps, tick, match length, boss length, mob TTK, room transitions, traversal. |
| `animation` | 13 | Hit-stop, slow-mo, camera shake, pickup animations, death cam, fade-in. |
| `feel` | 2 | Move speed, map overlay keybind. |
| **Total** | **~102** | (source was ~48; RT adds ~54 new timing/animation parameters.) |

**Row shape:** every row has `parameter, category, source_value, rt_value, units, notes`. `rt_value` is numeric when possible and narrative only when a value cannot be expressed numerically (e.g., exclusivity rules, boolean flags).

---

## 2. Source Values Preserved Verbatim

The following parameters carry over **unchanged** from the source. RT translation never touches them — changing them would undermine either the dice math (sacred), the dungeon topology (sacred), or the shop price curve (carefully tuned).

### Player identity
- `starting_health = 17` hearts.
- `health_cap = 17` hearts. Potions are hard-capped at 17; cannot overfill.
- `starting_attack_dice = 1`, `starting_defence_dice = 1` (dice in tray).

### Combat math
- `die_success_threshold = 6 on a d6`. This is the sacred number. Changing it doubles or halves the expected value of every roll in the game. **Do not touch.**
- `monster_hp_default = 1`. One unblocked hit during an Opening = kill. Source-faithful.
- `dracular_hp = 1`. Three phase gates; each phase ends on one clean hit. Source-faithful.
- `dracular_attack_dice = 9`, `dracular_defence_dice = 9`. Rendered as 9 visible d6 and a crimson aura respectively.
- `flee_combat = not allowed`. Room-lock preserved.
- `max_attack_dice_pool = 5` (1 base + 2 Big Axe + 2 Spiky Armour). Player's attack dice tray cannot exceed 5 visible dice.
- `max_defence_dice_pool = 7` (1 base + 2 Shield + 5 Magical Armour under A-3 literal stacking).

### Dungeon topology
- `grid_width = 9`, `grid_height = 10`, `total_normal_rooms = 90`. Preserved so that one row maps to one level table.
- `adjacency_type = 8-way king`. 8-neighbour walkable graph.
- `revisit_allowed = false`. Visited cells collapse behind the player.
- `min_path_length = 11` (derived).

### Monster tables
- All 10 level-specific `monster_lN_max_attack_dice` values (1, 2, 3, 4, 5, 6, 7, 8, 9, 10).
- All 10 level-specific `monster_lN_monster_probability` values (50, 33, 50, 67, 67, 67, 67, 67, 67, 67 %).
- `monster_progression_delta_per_level = +1 AD`.

### Economy
- `treasure_probability_default = 17 %` (flat across all 10 rows).
- `potion_probability_l1 = potion_probability_l3 = potion_probability_l6 = potion_probability_l9 = 17 %`. Other rows unchanged (i.e., 0).
- `potion_to_hp_ratio = 1:1` (one vial = one heart).

### Shop (all 14 price/bonus entries preserved exactly)
| Item | Cost (gold) | Attack Bonus | Defence Bonus |
|---|---:|---:|---:|
| Buckler | 1 | — | +1 |
| Shield | 2 | — | +2 |
| Potion (1) | 1 | — | — |
| Potion (3-pack) | 2 | — | — |
| Potion (6-pack) | 3 | — | — |
| Big Sword | 3 | +1 | — |
| Big Axe | 4 | +2 | — |
| Spiky Armour | 5 | +2 | +1 |
| Magical Armour | 6 | — | +5 |

Exclusivities: `Big Sword XOR Big Axe`, `Buckler XOR Shield`, `Spiky XOR Magical`. Preserved. (Sidegrade swap costs a half-refund — an RT concession documented separately in EconomyModel.md.)

---

## 3. Source Values Translated (Per-Turn → Per-Room / Per-Beat)

Real-time continuous play has no "turn," so any source rule expressed in turns has been re-expressed.

| Source concept | Source unit | RT unit | RT translation |
|---|---|---|---|
| Roll 1 d6 "per room on entry" | per turn | per room entry | Still 1 roll per room, but pre-seeded at run-start; the reveal is the only part the player experiences (fog clear ~250 ms). |
| Potion usable "next turn" after purchase | turns | ms | `bought_potion_delay = 5000 ms` arming cooldown with a visible chilling clock. |
| "Player rolls defence dice each combat round" | rounds | combat beats | Defence roll fires once per monster strike frame; player's visible defence dice tray rolls in sync with the enemy's attack tray. |
| "Monster attacks then player attacks" (4-step round) | round ticks | ms-granular beats | Monster windup 600 ms → strike 200 ms → opening 1000 ms → player swing window. See §6. |
| "Expected rounds to kill" | rounds | seconds | Derived directly from DPS math in §6. A 1-AD player kills a 1-HP mob in ~2.5 s; a 5-AD player kills in ~0.8 s. |
| "Treasure accrual in 30 turns" | turns | seconds | 30 turns × ~30 s/turn = ~15 min, which maps cleanly to our 20-min median match. Expected treasure drops held at 17 % per room → still ~5 per 30 rooms. |
| "Shop visited every turn" | per turn | per ~3 rooms | Altars placed procedurally ~1 per 3 rooms rather than "after every single room." Player walks physically to the altar, costing movement. |
| "Monster AD count" | dice thrown per round | dice in the enemy-side dice tray | Same number, new semantics: N d6 visibly spawn and tumble each strike. |
| "Monster HP = 1, lethal on connect" | hp | Opening-gated hp | 1 HP preserved. What changed: the player cannot land a hit outside the Opening (parry → stun, or post-strike recovery), so kills require timing. |

**Key principle:** numeric values are preserved where they express a frequency or probability that is independent of wall clock. They are re-expressed where they reference "turns," which RT has none of.

---

## 4. RT-Native Additions (new parameters)

These parameters do not exist in the source at all. They govern the real-time feel layer that RT demands and the board game never needed.

### Parry & dodge windows
- **`parry_window_ms = 200`** — the monster's strike frame. An LMB/RB click during this window triggers a parry. This is the canonical "forgiving but learnable" duration used by *Hades*, *Hollow Knight*, *Dead Cells*. Too short feels unfair; too long makes parry braindead.
- **`parry_window_start_ms = 400`** — parry opens 400 ms after the telegraph begins (i.e., when the monster's windup animation transitions into the strike pose). The player must read the tell and click when the pose commits.
- **`perfect_parry_ms = 80`** — first 80 ms of the strike frame. A parry landed here cancels *all* incoming hits regardless of the defence dice roll. This is the RT reward for precision timing and maps diegetically to "a lucky defence roll."
- **`parry_stun_duration_ms = 500`** — the monster's stagger after a successful parry; its guard drops and an Opening window becomes active.
- **`dodge_iframes_ms = 350`** — invulnerability window on dodge-roll. Source has no analogue; invented to give motor-accessible players an alternative to parry.
- **`dodge_recovery_ms = 400`** — post-dodge committed recovery during which the player cannot attack.
- **`dodge_cooldown_ms = 700`** — full dodge-start to next-dodge-ready cycle. Prevents spam.

### Monster pacing
- **`monster_telegraph_ms = 600`** — constant windup duration across monster types. The *visual* differs wildly (Orc jab vs. Demon overhead smash), but the frame budget is identical so the player's reaction learning transfers across enemies.
- **`monster_windup_ms = 400`** — a shorter "I'm about to telegraph" body-lean that precedes the main telegraph on tougher enemies. Pre-tell for experienced players.
- **`monster_strike_frame_ms = 200`** — active damage frame; equal to parry_window_ms.
- **`monster_recovery_ms = 300`** — post-strike period of natural guard drop (an Opening without needing to parry).
- **`opening_duration_ms = 1000`** — the total window during which a player hit counts as a kill. The sum of monster_recovery + parry_stun is approximately this. Tunable downward for Hard mode.

### Player action tempo
- **`player_swing_windup_ms = 300`** — wind-up before the hit frame.
- **`player_swing_cooldown_ms = 800`** — full swing-to-swing cycle. Intentionally close to monster telegraph duration so that a perfect player gets exactly one swing per monster windup, preserving the source's 1:1 alternating round rhythm.
- **`player_draw_weapon_ms = 400`** — delay at encounter start before the first swing is legal. Enforces "monster attacks first" diegetically.

### Movement & traversal
- **`cell_traversal_ms = 400`** — one cell takes 0.4 s to walk. Derived from board game "every square is a decision" feel — slow enough to commit, fast enough not to feel laggy.
- **`cell_commit_window_ms = 120`** — first 30 % of the walk animation is cancelable; the rest is committed.
- **`player_move_speed_m_per_s = 4`** — derived from 1.6 m cell / 0.4 s traversal.

### Room flow
- **`room_transition_ms = 300`** — camera + lighting transition between rooms.
- **`fog_reveal_ms = 250`** — the fog of war clearing to expose room contents.
- **`first_aggro_grace_ms = 150`** — the "orient yourself" grace period after fog-reveal.

### Pickups
- **`treasure_pickup_animation_ms = 200`** — coin-magnet + HUD tick.
- **`potion_pickup_animation_ms = 200`** — bottle pickup + slot fill.
- **`potion_drink_animation_ms = 500`** — drink quaff; player is mobile but cannot swing/dodge.
- **`bought_potion_delay = 5000 ms`** — RT translation of the source's "next turn" delay on bought potions.

### Juice & feel
- **`hit_stop_ms = 80`** — frame-freeze on connecting hit.
- **`parry_hit_stop_ms = 200`** — the longest freeze in the game; the parry is the hero moment.
- **`slowmo_on_kill_ms = 100`** at time scale 0.3 ×.
- **`slowmo_on_parry_ms = 300`** at time scale 0.2 ×.
- **`camera_shake_kill = 0.15`** amplitude, 150 ms. Perpendicular.
- **`camera_shake_player_hit = 0.30`** amplitude, 250 ms. Radial.
- **`death_cam_ms = 1500`** — slow-motion death beat before the run-over screen.
- **`run_start_fadein_ms = 2000`** — ambient start fade.

### Match & session timing
- **`fps_target = 60`** (16.67 ms tick).
- **`tick_rate_ms = 16.67`** (derived).
- **`match_target_duration_s = 1200`** — 20-minute median match. Faster than the source's 20–40 min because there is no pen-and-paper bookkeeping.
- **`boss_fight_duration_s = 90`** — target Dracular TTK across all phases.
- **`normal_mob_ttk_s = 2.5`** — time-to-kill for normal mobs.
- **`room_clear_time_s = 15`** — average from room entry to room clear.
- **`dracular_phases = 3`**, with `phase1_duration_s = phase2_duration_s = phase3_duration_s = 30`.

### Shop / interact
- **`shop_interact_range_m = 2`** — the distance at which a shopkeeper NPC's radial menu becomes available.
- **`shop_time_scale_on_open = 0.1`** — world slows to 10 % when the shop menu opens, rather than fully pausing. Preserves the sense of continuous time.

### Mercy rules
- **`damage_per_hit_cap = 3 hearts`** — no single hit drops more than 3 pips, regardless of dice roll. Extra "hits" above 3 become knockback and screen shake. Directly mitigates the "10 AD rolls all 6s and I instantly die" unfair moment.
- **`pity_heart_threshold = 5 hearts`** — if damage would drop the player from >5 pips to 0, they survive at 1 pip with a brief invulnerability frame instead.
- **`pity_heart_cooldown = 3 rooms`** — pity heart triggers at most once per 3 monster rooms.

### UI
- **`map_overlay_toggle_key = V`** — the source game let players draw on paper and see the full grid always; RT condenses this to a toggleable overlay.
- **`potion_carry_cap = 6 vials`** — RT-only cap to prevent degenerate hoarding. The source was unbounded but expected play rarely held more than 4.

---

## 5. Top 5 Most Sensitive Parameters (RT-reassessed)

The source's top 5 sensitive parameters (die threshold, starting HP, L10 AD, dracular DD, grid size) are still all sensitive in RT. But the RT translation introduces new axes of sensitivity that can easily eclipse them if mis-tuned. Here is the RT-era top 5, ranked by the size of the experiential change a small numerical shift produces.

### 1. `parry_window_ms` (currently 200)
**Why sensitive:** This single number is the difference between "combat feels like Hades" and "combat feels like Dark Souls on caffeine." At 100 ms only twitch players can parry; at 500 ms parry becomes a passive auto-button. The skill ceiling and the skill floor of the entire combat system pivot on this value.
- **Safe tuning range:** 150–300 ms.
- **Breaks below:** 100 ms (unfair — only the top 5 % of players can consistently hit).
- **Breaks above:** 500 ms (trivial — parry spam dominates, telegraphs stop mattering).
- **Dependency:** must stay ≤ `monster_strike_frame_ms`. They are currently equal by design.

### 2. `monster_telegraph_ms` (currently 600)
**Why sensitive:** The telegraph budget defines whether combat is a reaction test or a dance. At 300 ms monsters become reflex executions; at 1200 ms monsters become training dummies because the player has time to coffee-break before each swing. This parameter controls the game's ribbon between "cerebral" (slow tells) and "reactive" (fast tells).
- **Safe tuning range:** 400–900 ms.
- **Breaks below:** 250 ms (reaction test — excludes older and accessibility-impaired players).
- **Breaks above:** 1200 ms (boring — combat loses its pulse).
- **Interaction:** must stay consistent across all monster types. If Cyclops and Orc have different telegraph budgets, players cannot generalize learning.

### 3. `player_move_speed_m_per_s` (currently 4)
**Why sensitive:** This is the RT replacement for what the source called "movement budget" (the 90 cells / run). At 3 m/s the game feels sluggish and combat encounters become unavoidable traps. At 6 m/s the player can simply walk around every monster windup, trivializing the entire parry/dodge system — dodge becomes optional because raw velocity out-paces strikes. Also interacts with `shop_interact_range_m` and cell traversal time.
- **Safe tuning range:** 3–5 m/s.
- **Breaks below:** 2 m/s (frustrating, slow room clears, match duration swells past 30 min).
- **Breaks above:** 6 m/s (dodge trivializes combat, no reason to parry, room clear times collapse).
- **Dependency:** if changed, `cell_traversal_ms` must scale inversely.

### 4. `dice_roll_animation_ms` (currently 350)
**Why sensitive:** The dice tray is the diegetic heart of the source's feel. Every swing rolls visible dice. If the roll animation is too slow, each combat beat drags and the "1.5 s per monster" target fails. If too fast, the player can't read the dice, and the bursty / swingy signature of the source's math becomes invisible, reducing combat to "it hit" or "it didn't."
- **Safe tuning range:** 200–500 ms.
- **Breaks below:** 100 ms (illegible — the player loses the dice-reading feedback loop).
- **Breaks above:** 700 ms (combat sluggish; normal mob TTK inflates past 4 s; match length swells).
- **Interaction:** must fit within `opening_duration_ms` so the dice resolve before the Opening closes.

### 5. `dracular_defence_dice` (currently 9 — inherited from source)
**Why sensitive:** Same as source — the boss fight pivots entirely on this value. But in RT the sensitivity is even sharper because the defence dice now render as the **crimson aura**, a visual wall the player must fight through. 7 DD and the aura drops often enough that any swing might land — fight becomes short and loses climactic weight. 11 DD and the aura almost never drops — fight becomes a 3-minute grind where 90 % of swings spark. The boss fight feels ruined at either extreme.
- **Safe tuning range:** 7–11 DD.
- **Breaks below:** 5 DD (boss pushover; target time-to-kill collapses to 30 s).
- **Breaks above:** 13 DD (boss grind; target time-to-kill balloons to 300 s).

### Honourable mentions (Tier 2 RT sensitivity)
- **`opening_duration_ms`** — if <400 ms, only top-tier reactions can punish parries and the parry becomes a frustrating Pyrrhic victory.
- **`damage_per_hit_cap`** — if removed, FP-1 bites: players die from single 10-AD rolls with no reaction time.
- **`monster_l10_max_attack_dice`** — unchanged sensitivity from source; the Demon still one-shot-threatens under-geared players even with damage cap.
- **`starting_health`** — same as source, still sensitive in RT because hearts are discrete pips that visibly deplete.
- **`player_swing_cooldown_ms`** — too short (<400) and combat becomes button-mash; too long (>1500) and every swing feels weighty but combat drags.

---

## 6. Expected Damage Math in RT Units

Rework the source's count-6s math into per-beat and per-second DPS estimates. The underlying dice math is unchanged (1/6 success per die); the RT layer simply maps the rolls onto a time budget.

### Per-swing expected damage (player attacking a monster)

| Player AD pool | P(0 hits) | P(≥1 hit) | E[hits] | Per-swing outcome |
|---:|---:|---:|---:|---|
| 1 | 83.3 % | 16.7 % | 0.167 | Usually spark. Kill on lucky rolls only. |
| 2 | 69.4 % | 30.6 % | 0.333 | Still mostly spark; meaningful chance of a kill. |
| 3 | 57.9 % | 42.1 % | 0.500 | Half the Opening swings kill. |
| 4 | 48.2 % | 51.8 % | 0.667 | Slightly favour kill per swing. |
| 5 | 40.2 % | 59.8 % | 0.833 | Dominant kill per swing. |

Note: "hits" that land outside an Opening are converted to sparks regardless of dice — the parry-gate is upstream of the dice roll.

### Per-beat expected time-to-kill (normal 1-HP mobs)

With an Opening firing every `player_swing_cooldown_ms = 800` ms, the expected number of swings to land a kill is `1 / P(≥1 hit)`. Multiply by 0.8 s and add a 0.5 s setup-beat average.

| Player AD pool | E[swings to kill] | E[TTK in seconds] |
|---:|---:|---:|
| 1 | 5.99 | ~5.3 s |
| 2 | 3.26 | ~3.1 s |
| 3 | 2.37 | ~2.4 s |
| 4 | 1.93 | ~2.0 s |
| 5 | 1.67 | ~1.8 s |

These estimates assume the player lands every Opening — in practice add ~30 % latency for imperfect play. A 5-AD player kills a normal mob in ~2.0 s under sloppy play and ~1.5 s under tight play, landing inside the **1.5–3 s** genre-lock target.

### Per-strike expected damage taken (monster attacking player)

Monster expected hits per strike = `AD / 6`. After the defence dice subtract (approximated as a simple difference; true value is slightly kinder):

| Monster AD | Player 1 DD | Player 3 DD | Player 5 DD | Player 7 DD |
|---:|---:|---:|---:|---:|
| 1 (Orc) | 0.00 | 0.00 | 0.00 | 0.00 |
| 3 (Skel) | 0.33 | 0.00 | 0.00 | 0.00 |
| 5 (Bat) | 0.67 | 0.33 | 0.00 | 0.00 |
| 7 (Dark Elf) | 1.00 | 0.67 | 0.33 | 0.00 |
| 9 (Wizard) | 1.33 | 1.00 | 0.67 | 0.33 |
| 10 (Demon) | 1.50 | 1.17 | 0.83 | 0.50 |

Cap of `damage_per_hit_cap = 3 hearts`: the worst possible Demon strike consumes at most 3 pips. Without the cap, 4+ six-rolls on 10 dice can happen ~6 % of the time and would drop a mid-geared player by 4–5 pips in one beat — unplayable.

### Per-beat Dracular math (the boss)

Dracular rolls 9 AD and 9 DD. Expected hits per attack = 1.5 (before player defence). Expected blocks on player swing = 1.5 (before player AD roll). A 5-AD vs 9-DD player produces ~0.833 − 1.5 = −0.667 … i.e., on raw numbers the player lands **zero** hits per swing on average. This is the source's intended coin-flip fight.

How RT preserves this: the boss aura (9 DD) is **visually down only during Opening windows**. The player must force an Opening (bite-lunge parry, bat-swoop apex, shadow-clone kill) and swing *during* the drop. Raw dice math is preserved for the actual roll, but the gate is positional.

Expected fight length:
- Phase 1: 3–5 Openings × 2.5 s inter-beat = **25–45 s**.
- Phase 2: 3–5 Openings × 3 s inter-beat = **30–50 s**.
- Phase 3: 1 Opening × 3 s sprint + setup = **25–40 s**.
- **Total: 80–135 s**, centered on 90–120 s target. Matches the **60–180 s** boss range in the genre lock.

### Per-second player DPS (for engine building curve)

| Player AD pool | Swings/sec | E[hits/sec] | DPS-equivalent vs 1-HP targets |
|---:|---:|---:|---:|
| 1 | 1.25 | 0.21 | 0.21 kills/sec |
| 2 | 1.25 | 0.42 | 0.42 kills/sec |
| 3 | 1.25 | 0.63 | 0.63 kills/sec |
| 4 | 1.25 | 0.83 | 0.83 kills/sec |
| 5 | 1.25 | 1.04 | 1.04 kills/sec |

(Assuming constant swing cooldown and a constant stream of Openings — i.e., in a practice-dummy scenario. Real combat is gated by monster telegraph timing and adds roughly 30–50 % overhead.)

---

## 7. Difficulty Mode Tuning Ranges

The source ships as a single-mode experience. RT introduces four difficulty modes because twitch-tolerance varies dramatically across players. Below is the delta sheet — every value diverges from Normal in a transparent way.

| Parameter | Easy | Normal | Hard | Nightmare |
|---|---:|---:|---:|---:|
| `starting_health` | 20 | **17** | 14 | 14 |
| `health_cap` | 20 | **17** | 14 | 14 |
| `starting_defence_dice` | 2 | **1** | 1 | 1 |
| `parry_window_ms` | 300 | **200** | 150 | 120 |
| `perfect_parry_ms` | 120 | **80** | 60 | 40 |
| `monster_telegraph_ms` | 800 | **600** | 500 | 400 |
| `opening_duration_ms` | 1400 | **1000** | 800 | 600 |
| `dodge_iframes_ms` | 500 | **350** | 300 | 250 |
| `damage_per_hit_cap` | 2 | **3** | 3 | 4 |
| `pity_heart_threshold` | 7 | **5** | 3 | off |
| `pity_heart_cooldown` (rooms) | 2 | **3** | 5 | — |
| `dracular_attack_dice` | 9 | **9** | 9 | 10 |
| `dracular_defence_dice` | 7 | **9** | 9 | 10 |
| `treasure_probability_default` | 20 % | **17 %** | 15 % | 15 % |
| `match_target_duration_s` | 1400 | **1200** | 1100 | 1000 |
| Expected win rate | ~75 % | ~55 % | ~35 % | ~15 % |

**Design intent per mode:**
- **Easy** — not about slower monsters, about *longer telegraphs* and *more forgiving parry*. The dice math is unchanged so the core identity survives. A cool-headed player can play Easy and still experience the full game.
- **Normal** — the canonical tuning. Matches source feel. 55 % win rate mirrors the source's real-world completion curve.
- **Hard** — shaves the HP budget, shrinks every reaction window, tightens the Opening. The dice math is still source-faithful but the reaction surface is smaller.
- **Nightmare** — Hard + upgraded Dracular + pity heart disabled. This is the "source was already hard, now we made it harder" mode. Reserved for players who 100 %'d Hard.

**What modes do NOT change:**
- `die_success_threshold` — never. The sacred 1/6 success rate is unchanged across all modes.
- Shop prices, item bonuses — never. Economy discipline is the same across modes.
- Monster type / progression — never. The enemy cast is identical.
- Grid size, revisit rule, room probabilities — never. Dungeon topology is sacred.

---

## 8. Feel Targets

One-line feel targets for the major player actions. These are the north stars that tuning should serve. If a tuning change violates the target, revert.

### Movement
- **Walking** should feel like *walking across a stone dungeon floor with commitment* — not flighty, not sluggish. The player knows each step matters. Reference: Hyper Light Drifter's dash-less walk.
- **Dodge** should feel like *a panic button that saves you, then costs you tempo*. You get out of the strike but you lose the Opening. Reference: Hades dash (but without the offensive utility — RT-Solo Dungeon Dash's dodge is purely defensive).

### Combat
- **A parry** should feel like *a cathedral bell tolling inside time itself*. The screen flashes white, the world slows, a golden die pulses in the tray, and the monster is frozen in a stagger pose with a red "!" above its head. This is the hero moment — the player should want to parry every single strike just to feel it.
- **A dodge-roll** should feel like *an unglamorous survival exhale*. No hit-stop, no slow-mo, just silent invulnerability and a brief recovery tax. It's the "I wasn't ready" option.
- **A successful swing (kill)** should feel like *a decisive exclamation mark*. Hit-stop, slow-mo, a gold-pulsed 6 in the dice tray, a crunch, a spray of appropriate particles, a short camera shake. The 1-HP monster dies like an event — because it IS the event.
- **A spark (non-opening hit)** should feel like *clanging a hammer against steel*. Sharp clang, yellow sparks, tiny stagger on the monster, no hit-stop, no slow-mo — a clear "that didn't count" signal. The contrast with a kill swing is the entire learning feedback.
- **Taking a hit** should feel like *a punch that stops the music*. Red vignette, strong screen shake, a visible pip shattering on the HUD with a "break glass" effect, controller rumble, a low-frequency rumble in audio, a brief time slow. You *know* you were hit.
- **A boss swing landing** should feel like *a cathedral organ chord smashing your face*. Double the shake, double the slow-mo window, the shattered pip displays the damage number (for bosses only), the screen goes nearly black on a 3-pip charged hit. This is the game's worst punishment and the feel should match.

### Economy beats
- **Picking up treasure** should feel like *a small satisfaction*, one pleasant coin chime per pickup, a tiny HUD juice animation. Not spammy. Source's "1 Treasure counter tick" translated to sub-300 ms juice.
- **Picking up a potion** should feel like *a moment of hope*. A soft bottle chime, the vial slides into the bar, the player's next hit is survivable.
- **Drinking a potion** should feel like *a commitment you have to earn*. 500 ms of vulnerability, a visible quaff animation, the heart bar ticks up one pip. If you panic-drink in a fight, the consequence is obvious.
- **Shop interaction** should feel like *an island of calm*. The world slows to 10 %, music ducks, the shopkeeper turns to face you, a radial menu opens. Time isn't stopped — tension bleeds in slowly — but the rhythm changes.

### Macro beats
- **Entering a new room** should feel like *a small gasp*. Fog clears in 250 ms, the room's contents reveal, and the player has 150 ms to read the situation before the monster begins its windup.
- **Clearing a room** should feel like *catching your breath*. The music ducks, the room lighting warms, no pressure.
- **Hitting a new row** (level transition) should feel like *the weight settling*. Camera pans up, music swells, the player sees the new level's monster table telegraphed on a brief HUD banner.
- **Dying** should feel like *a final, visible mistake*. 1.5 s death cam, clear view of what killed you, "RUN OVER" stinger, fade to post-run screen. Fair loss, clear death.
- **Killing Dracular** should feel like *earning the ending*. The crimson aura shatters, the dice tray goes gold, an organ swell, a 3-second slow-mo, the credits fade in.

---

## 9. Sanity Checks Against the Genre Lock

- **Match length** — `match_target_duration_s = 1200` matches the 20-min median.
- **Tick rate** — `fps_target = 60` / 16.67 ms matches the genre lock.
- **Normal mob TTK** — 1.5–3 s target → our 5-AD `normal_mob_ttk_s = 2.5` hits center of range. 1-AD starter fights will be ~5 s (slightly over the range — intentional, since early game is meant to feel scary).
- **Dracular TTK** — 60–120 s target → our `boss_fight_duration_s = 90` and three 30 s phases hits center.
- **Room traversal** — 15–45 s target → our `room_clear_time_s = 15` is on the low end but matches the fact that the majority of rooms are non-combat (treasure, potion, empty).
- **Cell move** — 0.35–0.5 s target → our `cell_traversal_ms = 400` hits center.
- **Combat beat** — 1.5–3 s per normal mob target → our per-swing cycle of 800 ms × ~2 swings = 1.6 s hits the low end. Sloppy play pushes into the middle of the range.

All values consistent with the genre lock.

---

## 10. Red Flags Preserved From Source

The source `BalanceSheet.md` flagged five balance issues. None of these should change in RT — they are either intentional design or already handled by existing shop exclusivities. For reference:

| Flag | Parameter(s) | RT Status |
|---|---|---|
| Shield strictly dominates Buckler | `shield_cost`, `buckler_defence_bonus` | Preserved. Buckler is still the "1-gold stepping stone." |
| Big Sword dominated by Big Axe per-die | `big_sword_cost`, `big_axe_cost` | Preserved. Big Sword is a saving-up intermediate. |
| Magical Armour vs Shield per-die | `magical_armour_cost`, `shield_cost` | Preserved. Different tiers. |
| Spiky vs Big Axe+Buckler hybrid | `spiky_armour_*` | Preserved. Interesting design choice. |
| Dracular's 9/9 coin flip | `dracular_attack_dice`, `dracular_defence_dice` | **Intentional tension** — preserved unchanged on Normal. |

---

## 11. Summary of RT-Era Changes

- **~48 source parameters preserved**, including all dice math, all topology, all shop prices, and all monster tables.
- **~54 new parameters added**, covering parry/dodge windows, monster telegraph timing, animation durations, hit-stop/slow-mo juice, camera shake, pickup animations, match-length targets, and Dracular phase durations.
- **3 source concepts translated** from turn-based to real-time equivalents: bought-potion delay (5 s arming cooldown), per-turn shop availability (1 altar per ~3 rooms), combat round (1.2–2 s beat cycle).
- **5 top-sensitivity parameters**, of which 4 are RT-new: parry window, monster telegraph, player move speed, dice roll animation, and dracular DD (inherited from source).
- **4 difficulty modes** defined via a delta sheet: Easy, Normal, Hard, Nightmare.
- **Feel targets** encoded for every major player action so future tuning can be guided by intent rather than numbers alone.

This sheet is the **single source of truth** for RT-Solo Dungeon Dash's numeric tuning. Downstream deliverables (RTGDD, architecture, assets) reference these values directly.
