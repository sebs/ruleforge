# Temporal Map — Solo Dungeon Bash → Real-Time

> RealTimeForge Stage RT-0: Temporal Deconstruction
> Source: `output/solo-dungeon-bash/` (RulesExtraction, GameLoop, Mechanics, GDD)
> Target: Real-time interactive translation (2D/3D) of Felbrigg Herriot's 2007 print-and-play roll-and-write.

---

## 1. Overview

### What time units exist in the source game?

Solo Dungeon Bash is a **strictly sequential, player-paced, discrete-time** board game. There is not a single second, pulse, cooldown, timer, or parallel actor anywhere in the rulebook. Time exists only as an *ordering relation* over discrete events. Concretely, the game recognizes exactly these time units — and nothing else:

| # | Time unit | Scope | Triggered by | Observable effect |
|---|---|---|---|---|
| T0 | **Micro-step** | one die-roll | player picks up dice, rolls, counts sixes | updates counters (hits, blocks, content) |
| T1 | **Combat tick** (1 of 4) | inner combat loop | each of the 4 ordered combat sub-steps | flips "whose action" bit |
| T2 | **Combat round** | full atk-def-atk-def cycle | completion of sub-step 4 | HP may change; monster may die |
| T3 | **Turn step** (1 of 7) | primary loop segment | player voluntarily advances | updates grid/resources/inventory |
| T4 | **Turn** | full primary loop | completion of step 7 | cursor "advances" to next turn |
| T5 | **Level transition** | implicit, position-driven | player moves into a new row | swaps active encounter table |
| T6 | **Run** | full tertiary loop | reset at Start | win/loss resolution |

There is **no** T-unit smaller than "one die roll," and none of these units has an enforced wall-clock duration. A player can think for 30 seconds or 30 minutes between squares without penalty.

### Temporal nature of the source game

- **Strictly sequential.** No parallelism whatsoever — the player is the only actor, the monster acts only via deterministic dice rolls during combat, and even within combat the monster and player alternate.
- **Monotonically progressing.** Time never flows backward. Once a room is entered and rolled, its contents are fixed. Path cannot be un-drawn.
- **Player-paced.** Zero pressure from the clock. The game tolerates arbitrarily long deliberation.
- **Atomic decisions.** Each decision (pick square, use potion, buy item) is a single committed act. There is no "partial" state between decisions.
- **Deterministic structure with stochastic content.** The *skeleton* (7-step sequence, 4-step combat) is fixed; the *contents* (what's in the room, who hits) are dice-driven.
- **Single initiative holder.** The player owns 100% of agency outside combat. Inside combat, initiative is fixed (monster first), mechanical, and uncontested.

### The translation problem in one sentence

We must take a game whose entire sense of "time" is "whenever the player clicks next," and invent a **continuous, pressured, twitch-tolerant** version of every one of those moments without destroying the three design pillars: "every step is a gamble," "permanent progression within the run," and "fair loss, clear death."

### Direction chosen (stated up-front so the rest of the doc can be opinionated)

RealTimeForge-SDB is **slow-tactical action-RPG with dungeon-crawler flow**: think *Diablo* or *Hades* frame-rate for movement and combat, but with the **punctuated rhythm** of *Into the Breach* — i.e., real-time locomotion and melee, but with a **beat-based combat cadence** inside rooms so that the board game's "alternating dice rolls" translates cleanly to "monster windup → parry window → player swing window → monster defence/block → repeat." Not fast-twitch (*Devil May Cry*). Not slow-paced turns-with-timers (*FTL*). Somewhere in between. Target TTK for normal monsters: **1.5–3 seconds**. Target boss fight: **60–120 seconds**.

---

## 2. The 7-step turn sequence — per-step translation

In the source game each of these 7 steps is a discrete, fully-committed, atomic event. In RT they must all co-exist as **concurrent systems** that a player experiences continuously. Here is the translation for each.

### Step 1 — Pick next square and move into it

**Board game behaviour.** Player looks at up to 8 king-adjacent unvisited squares, picks one, draws a line. Zero time pressure. Full lookahead.

**RT translation.** Continuous **WASD / analog-stick locomotion** on the 2.5D grid, with the grid expressed as **cell-snapped tile movement** (~0.35–0.5 seconds per cell traversal at walk speed). Movement is not free-form; the player is always inside exactly one cell, and moving from A→B is an animated transition with a cancelable first ~30% of the animation. Adjacency becomes "you can only walk into a cell that is king-adjacent to the one you are in."

Key decisions:

1. **Cell snapping, not free roaming.** This preserves the board game's "every square is a decision" feel. A free-roam character trivializes the pathing puzzle.
2. **Revisit prohibition becomes a world-state lock.** Visited cells *close behind you* — the floor literally collapses, chains, or bricks over, so the player can see the topological constraint build up. The visited cell is no longer walkable.
3. **Block-detection → persistent "Path to End" mini-HUD** that continuously computes reachability and warns when the player is two moves from losing.
4. **Path hovering.** The next cell the player is *about to* enter highlights with a ~200ms "commit window" — if they reverse direction within that window, they don't commit. This is how we preserve the board game's "I can change my mind" moment without pausing the world.

**RT concept:** Continuous cell-snapped locomotion + world-state wall-closure behind you + persistent reachability check + short commit window.

### Step 2 — Roll to determine room contents

**Board game behaviour.** On entry, player rolls 1d6 and reads the row's level table. This is the single stochastic reveal in the turn.

**RT translation.** The cell's contents are **pre-seeded at run start** (deterministic per-run from a seed; see Stage RT-5 balance) but revealed **on proximity**. The reveal is a **"room materializes"** effect as the player crosses the cell threshold — fog clears, ambient lighting shifts, and the contents (monster model, treasure pile, potion bottle, or empty dust) appear in-place within ~250ms. There is no "rolling animation" in the player's face because rolling-in-RT would either (a) feel like a slot machine, which breaks immersion, or (b) force a pause, which breaks continuity.

Two important choices:

1. **Pre-seeded, not rolled on entry.** This is a deliberate break from literal fidelity. If every cell was rolled on entry in real-time, the fog of war would make the game nondeterministic in a way that cannot be showcased to a second player, replayed, or speedrun. Pre-seeding preserves reproducibility. The stochasticity happens at **world generation time**, not at **player encounter time**.
2. **The reveal still feels stochastic.** The player does not see the seed; they don't know what's in the next cell; they feel exactly as much surprise as the board game player feels on a die roll. From the player's POV, RT-SDB is just as dice-driven as board-SDB.

**RT concept:** Per-run pre-seeded content + on-proximity fog-of-war reveal with ~250ms materialization beat.

### Step 3 — Apply Treasure

**Board game behaviour.** +1 to Treasure counter. Zero player involvement beyond crossing the threshold.

**RT translation.** **Automatic pickup** on entering the cell. Treasure is a visible sparkling pile. On pickup: coin-collect SFX, particle burst, `Treasure` counter in HUD ticks by +1 with a small juice animation. No modal, no menu, no pause. ~300ms total feedback.

**RT concept:** Passive on-touch pickup with juice feedback.

### Step 4 — Apply Potion

**Board game behaviour.** +1 to Potion counter. Identical structure to Treasure.

**RT translation.** Same as Step 3 — automatic pickup, visible bottle on the floor, pick-up SFX, HUD tick, ~300ms feedback. **Important:** potions are *not* auto-drunk — they go into stockpile and the player decides when to use them. Picking up the potion is passive; *drinking* the potion is active (see Step 6).

**RT concept:** Passive on-touch pickup into stockpile, separate consumption action.

### Step 5 — Combat (fight to the death)

**Board game behaviour.** Automatic alternating-round combat against the rolled monster until one side dies. Player makes zero decisions during combat in the source rules (combat is deterministic-once-rolled, per Mechanics.md).

**RT translation.** The monster is **spawned in-cell on reveal** (from step 2) and **immediately aggros**. Combat is **real-time melee** with a beat-based cadence:

- Monster performs a **telegraphed windup attack** (~600ms windup, ~200ms strike frame).
- Player has a **parry window** during the strike frame — a timed LMB-click defends, converting the "defence dice" roll into a skill-expressed input.
- After the monster's strike, the player can **swing** (~300ms windup, instant hit) — this is the "player attacks" step of the source combat round.
- Monsters with defence dice (i.e., Dracular) get a **block frame** during the player's strike — the player sees a shield/shimmer and must time the swing between block frames.
- Combat ends when monster HP hits 0 (one unblocked hit, per source) OR player HP hits 0.

This is the most significant reinterpretation in the document and it deserves its own section (see §3).

Critically: **the player cannot walk past an unresolved monster.** Entering a cell with a monster commits the player to combat. The room "locks" (walls glow, exits shimmer) until combat resolves. This preserves the board game's "fight to the death" rule.

**RT concept:** Room-locked real-time melee with beat-cadenced windup/parry/swing/block cycles.

### Step 6 — Use Potions (recovery phase)

**Board game behaviour.** Between combat and shop, player may drink any number of stockpiled potions (1 per +1 HP, capped at 17). Strictly outside combat.

**RT translation.** **Persistent hotkey-bound action** (e.g., `Q` or a radial), usable **at any time the player is not mid-swing or mid-parry**. Ambiguity A-2 from the source rules said "no mid-combat drinking," and we will **honour that** by disabling the hotkey during the combat-locked state. Outside combat, the hotkey is a snappy ~150ms quaff animation with an HP bar tick-up and a short vulnerability frame (can't swing mid-drink).

Two sub-choices:

1. **One potion per quaff.** Holding the button quaffs repeatedly at ~0.5s/potion. This prevents the "insta-heal from 1 HP to 17 HP" button-mash exploit and creates a risk window where the player is committing to healing.
2. **Bought-potion delay preserved.** The source rule that newly-purchased potions are only usable "next turn" translates to a **cooldown on bought potions** — they appear in the stockpile with a ~5 second "chilling" indicator before becoming drinkable. This keeps the board game's tempo cost of shop-healing without requiring an explicit "turn" concept.

**RT concept:** Persistent hotkey action, disabled in combat-lock, one-quaff-per-press, cooldown on freshly-bought potions.

### Step 7 — Exchange Treasure at the shop

**Board game behaviour.** End-of-turn shop visit. Player spends Treasure on up to 6 items across 3 mutually exclusive slots + consumable potion packs.

**RT translation.** There is **no end-of-turn** in RT, so "shop" cannot be "after every cell." Two options:

- **Option A — Per-cell radial menu.** Opens on hotkey, anywhere, any time (outside combat). Pros: no backtracking. Cons: breaks the board game's "visit a shop" physicality.
- **Option B — Physical shop rooms.** One or more NPC shopkeepers spawn at fixed grid positions (e.g., row 1, row 4, row 7). The player must physically walk to them to buy. Pros: preserves pacing, rewards exploration, creates natural "push-your-luck" decision about when to shop-run. Cons: more UI states, risk of "I died two rows from the shop."

**We pick Option B.** Reasoning: the board game's shop is *always* available at zero movement cost — that's part of what makes step 7 "easy." But in RT we want the shop to **cost movement** because movement is the scarce resource (the no-revisit rule). By making the shop a destination, we force the player to allocate grid real estate to "how do I get to the shop and back without blocking myself." This is a **net-positive interpretation** of the source's intent (the board game's shop is free because it doesn't use movement; in RT, movement is the only currency aside from Treasure, so the shop *should* use it).

Additionally: **time pauses at the shop.** When the player enters shop dialogue, enemy AI freezes and ambient pressure stops. This is a deliberate island of calm, mirroring the board game's between-turns reflection moment.

**RT concept:** Physical shop NPCs at fixed grid positions, time pauses in dialogue, player allocates movement budget to reach them.

---

## 3. Combat sub-loop — 4-step inner translation

The board game's combat round is a 4-step deterministic tick cycle: *monster attacks → player defends → player attacks → monster defends*. This is the single hardest part of the translation because the board game's combat is 100% deterministic-once-rolled and the player has **zero agency** inside a round (per Mechanics.md M9 and GameLoop.md). To make RT combat fun, we must **inject agency** without breaking source-faithfulness.

### The core reinterpretation

Each of the 4 board game combat steps becomes a **combat beat** in RT. The full cycle takes ~1.2–2.0 seconds per round for normal monsters and the beats overlap into a single flowing exchange rather than 4 hard pauses.

| Source step | Board game tick | RT beat | Duration | Agency |
|---|---|---|---|---|
| 1 | Monster attacks: rolls N d6, counts 6s → M hits | **Monster windup + strike** | 600ms windup + 200ms strike frame | Player observes and positions |
| 2 | Player defends: rolls D d6, counts 6s → cancels M hits | **Parry window / auto-block** | 200ms strike frame (same as above) | **Timed LMB click = parry** |
| 3 | Player attacks: rolls A d6, counts 6s → H hits | **Player swing** | 300ms windup + instant hit | **LMB click at any time (cooldown-gated)** |
| 4 | Monster defends: rolls its DD → cancels | **Monster block frame (bosses only)** | 400ms shimmer window on strong enemies | Player times swing to miss the shimmer |

### Step-by-step with reasoning

#### Beat 1 (from source step 1) — Monster attack: telegraphed windup

- **Board game:** monster rolls `N` attack dice, where `N` = monster AD (1 for Orc, 10 for Demon, 9 for Dracular).
- **RT:** the monster plays a windup animation. The windup **duration is constant** (~600ms) but the **visual tell** scales with AD — an Orc (1 AD) does a little jab; a Demon (10 AD) telegraphs with a huge overhead smash. The animation telegraphs *both* that it's coming and *how bad* it'll be.
- **Hit computation:** behind the scenes, we **still roll N d6 and count sixes** to determine the number of incoming hits. This preserves the board game's expected-value math exactly. The rolls are computed once at the start of the strike frame.

#### Beat 2 (from source step 2) — Player defence: parry window

- **Board game:** player rolls `D` defence dice, counts sixes, subtracts from hits.
- **RT:** during the monster's 200ms strike frame, the player can **click/press parry** to convert their defence dice into a real-time roll. If the player **mistimes or ignores**, defence is still rolled automatically and the result still applies — so a passive player is no worse off than a board game player.
- **Skill expression:** if the player **parries within the first 80ms of the strike frame** (a "perfect parry"), *all* incoming hits are cancelled regardless of the defence roll. This is the RT reward for good reflexes, and it maps to a "lucky defence roll" from the board game's POV.
- **Miss consequence:** if the player *actively* clicks parry *outside* the 200ms window, no bonus — defence still rolls normally. No punishment for trying.

#### Beat 3 (from source step 3) — Player attack: swing

- **Board game:** player rolls `A` attack dice, counts sixes.
- **RT:** after the monster's strike resolves, a brief "opening" frame (~150ms) lets the player swing. The player **holds LMB to charge** or **clicks to quick-swing**. On swing, we roll `A d6` and count sixes just like the source.
- **Skill expression:** a **charged swing** (held for ~400ms) rolls the same dice pool with **advantage** (reroll one die). A **quick swing** uses the standard pool. This preserves source math while rewarding skill.
- **Movement coupling:** the player can *sidestep* during their swing window — dodge-roll out of the next monster windup. This is how the RT game bridges the board game's turn-based pace into continuous action.

#### Beat 4 (from source step 4) — Monster defence: block frame

- **Board game:** monster rolls its DD. Only Dracular has any.
- **RT:** normal monsters skip this entirely (DD = 0 → always die on one unblocked hit, same as source). **Dracular** and any other DD-bearing enemy (future content) get a **visible 400ms block frame** shimmer during the player's swing — hit during the shimmer = the DD dice decide, hit outside = clean damage.
- **Skill expression:** reading Dracular's block rhythm is the **entire boss fight**. This converts the source's "rolling 9 DD vs 9 AD" abstract dice duel into a readable dance.

### Non-normal combat modes

- **Combo combat (multi-monster room).** The source rulebook does not contemplate multi-monster rooms (1d6 always produces one content type). RT preserves this — one monster per cell, full stop.
- **Fleeing combat.** The GDD's "What we should NOT add" list explicitly forbids fleeing. We honour that: **the room is locked** until combat ends.
- **Dying mid-swing.** If player HP reaches 0 during the strike frame, the game ends immediately — no "complete the animation" grace. The source is roguelike-lethal on purpose.

### Why alternating rounds, not simultaneous action?

One could translate "combat" as simultaneous free-form hack-and-slash, but that would erase the source's defining rhythm. The alternating monster-then-player structure is **the rhythm of the board game**, and preserving it gives RT-SDB a unique flavour among action games: **every combat is a call-and-response dance**, not a blender. This is the *Sekiro*-ification rather than the *Diablo*-ification of the source.

---

## 4. Nested loop hierarchy — mapped

The source defines 4 nested loops (atomic, primary, secondary, tertiary). RT needs 4 nested time scales as well: **frame-tick, short-loop, medium-loop, match-arc**. Here is the mapping.

### 4.1 Atomic → Frame-tick

| Aspect | Board game (Atomic Loop) | RT (Frame-tick) |
|---|---|---|
| Definition | One combat round (atk/def/atk/def) | One render frame (16.67ms @ 60fps) |
| Duration | ~2–5 seconds per round | 16.67ms |
| Contains | 4 discrete ticks | Frame-level: animation pose, input poll, physics step |
| Player agency | None in source | **Full** — every frame can receive a parry/swing/dodge input |

The board game's "combat round" becomes an RT **combat beat cycle** that takes ~1.2–2.0 seconds and is made of thousands of frame-ticks. The important thing is that the *cadence* of the combat beat cycle (monster-strike → player-strike → repeat) is preserved at human-perceivable rhythm (~1 Hz of call-and-response), not at frame rate.

### 4.2 Primary → Short-loop

| Aspect | Board game (Primary Loop) | RT (Short-loop) |
|---|---|---|
| Definition | One turn (7 steps) | One room-clear: walk in → reveal → combat → pickup → move on |
| Duration | ~15–60 seconds | ~3–15 seconds per room |
| Contains | Pick / Roll / Treasure / Potion / Combat / Potion-use / Shop | Locomote / Reveal / Combat / Loot-vacuum / Locomote |
| Agency | 2 decision points (pick, shop) | Continuous micro-decisions (route, parry timing, swing timing) |

The short-loop is **denser** than the board game's primary loop — more micro-decisions, but each is smaller. The aggregate tension is similar.

Shop-visits are **removed from the short-loop** and pushed into a medium-loop activity (see §2, Step 7). This is deliberate and significant.

### 4.3 Secondary → Medium-loop

| Aspect | Board game (Secondary Loop) | RT (Medium-loop) |
|---|---|---|
| Definition | Level progression (position-driven row transitions) | Multi-cell strategic arcs: **"push to row 4 and stock up," "run back to shop," "spiral to find potions"** |
| Duration | ~3–10 turns | ~30 seconds to 3 minutes of gameplay |
| Contains | Several turns at similar risk profile | Several rooms + a shop run or strategic retreat |
| Agency | Push-your-luck decision on when to advance | Same, plus physical route planning (how to stay close to shop / how not to block self) |

**Key addition in RT:** the medium-loop gains a **spatial routing meta-game** that the board game has only implicitly. In the board game, your pen can instantly mark any unvisited square — there's no physical cost to backtracking across the map. In RT, traversal itself takes real seconds (~0.5s/cell), so the player plans **round-trips**: "can I make it to the L4 shop and back before the monsters on the path respawn?" (Note: monsters do NOT respawn in our design — see §5 — but the player's *perceived* pacing pressure is present nonetheless.)

### 4.4 Tertiary → Match-arc

| Aspect | Board game (Tertiary Loop) | RT (Match-arc) |
|---|---|---|
| Definition | Full run: Start → Dracular → Win/Loss | Full run: same structure |
| Duration | 20–40 minutes | **12–25 minutes** — faster because no pen-and-paper bookkeeping |
| Contains | Early farming → mid upgrade → deep push → boss | Same three-act structure, paced for RT |
| Exit | Win / Dead / Blocked | Same |

RT runs should be **slightly shorter** than the board game because:

1. No physical writing/erasing.
2. No dice-to-table lookup mental work.
3. Combat, while beat-based, is fast.
4. Continuous locomotion is faster than "pick next square, draw a line."

But they should NOT be much shorter, or the match-arc loses its "expedition" feel. Target **~15 minutes** per run as a sweet spot.

### Visual summary — nested loop mapping

```
Board game                                   Real-time
----------                                   ---------
Atomic (combat round, ~3s, 4 ticks)    →     Frame-tick (16.67ms) organised into
                                               combat beat cycles (~1.5s, 4 beats)
Primary (turn, ~30s, 7 steps)          →     Short-loop (room clear, ~5s)
Secondary (level progression, minutes) →     Medium-loop (strategic arc, ~1–3 min)
Tertiary (run, 20–40 min)              →     Match-arc (run, ~15 min)
```

---

## 5. Implicit timing — what the board game doesn't say, but RT must invent

Any board-to-RT translation has to invent a lot of numbers the source never mentions, because the source is indifferent to wall-clock time. Here is the exhaustive list of timing decisions RT must make that the board game doesn't answer.

### 5.1 Player locomotion speed

- **Board game:** N/A — pen instantly moves.
- **RT invented:** ~0.4s per cell walking, ~0.25s with a "hustle" hotkey (no stamina cost — the cost is committing to a cell sooner than you might want).
- **Why:** Fast enough to not feel laggy, slow enough that the player feels "committed" to the cell they just entered.

### 5.2 Cell-entry commit window

- **Board game:** N/A — pen commits atomically.
- **RT invented:** first 30% of the walk animation (~120ms) is cancellable. After that, the player is committed and the visited-cell-closure triggers.
- **Why:** preserves the board game's "I can change my mind" affordance without making movement feel sticky or rubber-banded.

### 5.3 Content reveal duration

- **Board game:** instantaneous on die roll.
- **RT invented:** 250ms fog-clear animation. The monster spawn is at 250ms; aggro begins at 400ms; the player has a 150ms "orient yourself" grace period before the first windup.
- **Why:** rewards attentive play, punishes walking in blind.

### 5.4 Monster aggro range and behaviour

- **Board game:** monsters are stationary. Each cell has one.
- **RT invented:** monsters are room-bound — they cannot leave their spawn cell. They aggro immediately on reveal. They turn to face the player and begin the windup cycle.
- **Why:** preserves "one monster per cell" and "fight to the death" exactly.

### 5.5 Monster respawn

- **Board game:** monsters don't respawn; each cell rolls exactly once.
- **RT invented:** **no respawns, ever.** A cleared room stays cleared. This is directly lifted from the source.
- **Why:** source fidelity. Adding respawns would make farming trivial.

### 5.6 Between-room "downtime"

- **Board game:** zero — the player decides immediately.
- **RT invented:** the player has as much time as they want between rooms **if they are not in a monster-locked room**. The world only exerts pressure via the persistent "blocking" check; otherwise the player can stand still and think. This is a deliberate concession to the source's player-paced nature.
- **Why:** tactical thinking is part of SDB's identity. Removing it would destroy the game.

### 5.7 Parry timing window

- **Board game:** N/A.
- **RT invented:** 200ms strike frame for standard parry, 80ms window inside that for perfect parry.
- **Why:** standard "forgiving but learnable" action-game timing. These are the *Hades*/*Hollow Knight* numbers.

### 5.8 Player swing cooldown

- **Board game:** one attack roll per combat round, no cooldown.
- **RT invented:** ~800ms cycle from swing-start to next swing-ready, roughly matching the monster's windup cadence so that a player who plays perfectly can land one swing per monster windup.
- **Why:** creates the call-and-response rhythm. Too short and combat becomes button-mash; too long and combat feels sluggish.

### 5.9 Potion quaff speed

- **Board game:** instantaneous.
- **RT invented:** 150ms per quaff, with a ~100ms "drinking frame" where the player can't swing or dodge.
- **Why:** creates a tempo cost for healing, which the board game doesn't have. Without this, potions would be infinite insta-heals.

### 5.10 Shop dialogue opening

- **Board game:** instantaneous.
- **RT invented:** ~300ms opening animation, pauses the world.
- **Why:** a clean signal that the player has entered a different mode.

### 5.11 Bought-potion cooldown

- **Board game:** "usable next turn" — i.e., next step-6 of the 7-step sequence.
- **RT invented:** ~5-second real-time cooldown on newly purchased potions, with a visible "chilling" icon in the stockpile UI.
- **Why:** preserves the source's intent (bought potions are delayed) in continuous time.

### 5.12 Blocked-path re-check cadence

- **Board game:** checked implicitly at move-time.
- **RT invented:** BFS recomputed every time a cell is committed. Persistent on-HUD indicator updates at 10 Hz.
- **Why:** the adaptation gap explicitly calls for BFS-on-every-move. 10 Hz is overkill but cheap and keeps the HUD always in sync.

### 5.13 Dracular's attack cadence

- **Board game:** standard combat round, same as any other monster but with 9 AD / 9 DD.
- **RT invented:** multi-phase boss with **three attack patterns** (fast jab ~400ms, heavy slam ~1000ms, AoE spin ~1500ms) rotated procedurally. Each pattern still rolls 9 d6 behind the scenes; the animation is just a vehicle.
- **Why:** a single monotonous pattern on a boss fight would be boring in RT, but the underlying math stays source-faithful.

### 5.14 Death animation / game-over reveal

- **Board game:** you stop. You lose.
- **RT invented:** ~1.5s death cam, then fade to "RUN OVER — Died on row 7, defeated by Wizard" screen.
- **Why:** "fair loss, clear death" is a design pillar. The player must see exactly *what* killed them.

### 5.15 Run-start fade-in

- **Board game:** N/A.
- **RT invented:** ~2s fade-in with ambient dungeon sound, player spawns on Start cell, first 3 legal moves highlight.
- **Why:** matches the onboarding spec.

### 5.16 Idle-timeout warning (controversial)

- **Board game:** N/A — player can stare at the map for hours.
- **RT invented:** **none by default**. Optional "arcade mode" could add a 60-second idle warning, but the default mode preserves full player-paced freedom outside of combat.
- **Why:** imposing an idle timer would break the source's identity. We explicitly reject it.

---

## 6. Temporal Translation Table

One row per concept. This is the master table for downstream stages (RTGDD, revised docs, balance, assets).

| Board game concept | RT translation | Duration / cadence | Justification |
|---|---|---|---|
| Turn order (player-only) | Continuous player control | n/a — always on | No other agents except monsters in combat; player owns the world clock |
| 7-step turn sequence | **Concurrent subsystems** (movement, reveal, pickup, combat, potion, shop) running at once | Each fires on demand | RT cannot serialize steps; they must overlap as persistent systems |
| Step 1 (pick square) | Cell-snapped WASD locomotion with 120ms cancel window | 0.4s per cell | Preserves cell-as-decision feel; cancel window preserves "change mind" |
| Step 2 (roll contents) | Fog-of-war reveal on proximity (pre-seeded at run start) | 250ms reveal | Determinism for replay; surprise preserved from player POV |
| Step 3 (Treasure pickup) | Passive on-touch collect | 300ms juice | Trivial; the board game step itself is passive |
| Step 4 (Potion pickup) | Passive on-touch collect into stockpile | 300ms juice | Same reasoning |
| Step 5 (Combat) | Room-locked real-time melee with 4-beat cadence | 1.5s per round, 1–3 rounds typical | Preserves alternating rhythm; lock preserves "fight to the death" |
| Step 6 (Use potions) | Persistent hotkey, disabled in combat lock | 150ms/quaff | Honours A-2 (no mid-combat drinking) |
| Step 7 (Shop) | Physical shopkeeper NPCs at fixed rows, time-pauses on dialogue | 300ms open; unbounded dialogue | Movement becomes the shop's cost; preserves "island of calm" |
| Combat step 1 (monster attack) | Telegraphed windup animation, visual scaled to AD | 600ms | Telegraph rewards reading |
| Combat step 2 (player defence) | Parry window; LMB within strike frame | 200ms parry, 80ms perfect | Standard learnable action timing |
| Combat step 3 (player attack) | Swing with charge option | 300ms windup, 800ms cooldown | Rhythm matches monster cadence |
| Combat step 4 (monster defence) | Block frame (bosses only) | 400ms shimmer | Creates readable boss dance |
| Room content determination | Deterministic per-run seed | at run start | Replay determinism while keeping in-run surprise |
| Level table (10 variants) | Spatial progression — monster strength scales with row | every row crossing | Preserves source's "sliding" level tables verbatim |
| Monster HP = 1 | Normal enemies die on one unblocked hit | instant | Direct lift |
| Dracular 9 AD / 9 DD | Multi-phase boss with 3 attack patterns; 9d6 rolled per pattern | 60–120s fight | Source math preserved; animation varied |
| Health cap 17 | Same | n/a | Direct lift |
| Health loss | Animated HP bar depletion + heartbeat SFX <5 HP | instant per hit | GDD Appendix B |
| Potion (found) | Usable same encounter, outside combat lock | 150ms quaff | Honours "same turn" for found potions |
| Potion (bought) | ~5s cooldown before usable | 5s | Translates "next turn" to real-time delay |
| Treasure counter | HUD counter with +1 animations | instant | Trivial |
| Shop exclusivity (slots) | Greyed-out items in radial menu; visible slot UI | n/a | Adaptation gap calls this out |
| No-revisit rule | Visited cells lock (floor collapse) | on cell commit | Makes topological constraint visible |
| King-adjacency | 8-way cell neighbourhood walkable | n/a | Direct lift |
| Block-detection (A-1) | BFS on every commit, persistent HUD indicator | 10 Hz | Adaptation gap spec |
| Run start | Fade-in + highlight legal moves | 2s | Onboarding spec |
| Run end (win) | Dracular death cam + "Dungeon Cleared" screen | 3s | "Fair loss clear death" inverted |
| Run end (loss-dead) | Death cam + "Run Over" screen | 1.5s | Design pillar |
| Run end (loss-blocked) | Camera pulls out to full map + "Blocked" screen | 2s | Must make the topology visible |
| Grid-level "level progression" | Implicit spatial gradient in monster spawns | continuous | Source is already position-driven |
| Push-your-luck | Player decides when to ascend vs farm | continuous | Same as source — just continuous |
| Engine-building (items) | Visible stat changes in HUD + weapon model swap | on shop purchase | Makes progression physical |
| Self-blocking trap | Full-map pan-back + replay of critical commit point | 3s | Teaches the player why they died |
| Idle thinking | Fully allowed outside combat lock | unbounded | Source is player-paced |

---

## 7. Temporal Feel

### The one-sentence pitch

**Slow-tactical dungeon-crawl action-RPG: you walk, you think, you fight in call-and-response bursts, and you breathe between fights.**

### Dial positions (where RT-SDB sits on the tension spectrum)

```
Fast twitch                                              Slow deliberate
|-------|-------|-------|-------|-------|-------|-------|
              X (RT-SDB, ~30% from slow end)

Devil   Hades   Diablo  Sekiro  Darkest  FTL     XCOM   Into the
May Cry                         Dungeon                 Breach
```

RT-SDB should feel **closer to *Darkest Dungeon* and *Sekiro*** than to *Diablo* or *Hades*. The player should feel:

- **Breathing room between rooms.** When a room is cleared, the world exhales. No spawn waves, no timers, no "hurry!" The player can stand on the cell and plan.
- **Focused tension during combat.** During combat, the world narrows to the monster and the rhythm. No camera sweeps, no side objectives, just the beat.
- **Weight on every step.** Because visited cells lock behind you, every cell-entry is a small commitment. The RT equivalent of the board game's "draw a line with your pen."
- **Calm at the shop.** Walking into a shop should feel like walking into an inn. The world-music dips, the ambient dungeon hum fades, the shopkeeper greets you.

### What RT-SDB should NOT feel like

- **Not button-mash.** If a player can beat combat by spamming click, we've failed the source.
- **Not DDR.** If a player has to hit 4+ inputs per second, we've failed the source.
- **Not a timer game.** If the player feels hurried outside combat, we've failed the source.
- **Not a walking sim.** If locomotion feels empty or decorative, we've failed the source's "every step is a gamble" pillar.

### Temporal beats per run (target)

A well-paced RT-SDB run should contain roughly:

- **40–70 rooms** entered (source has 92 cells; player visits a subset).
- **20–40 combat encounters** across a run.
- **3–6 shop visits** (constrained by physical shop placement).
- **2–5 "scary moments"** where HP is below 5 (heartbeat SFX kicks in).
- **1 boss fight** of 60–120 seconds.
- **12–18 minutes** total run length.

### Pacing diagram (qualitative)

```
Tension
 ^
 |          /\         /\            /\            /|
 |         /  \       /  \          /  \          / |
 |    /\  /    \  /\ /    \    /\  /    \    /\  /  | boss
 |   /  \/      \/  V      \  /  \/      \  /  \/   |
 |  /                       \/              \/      |
 |_/_______________________________________________\|
    ^                                                ^
   Start                                            End
        ^         ^           ^              ^
        first     L4 shop     L7 shop        approach
        combat                                boss
```

Tension spikes on combat, dips between rooms, compounds as the player descends deeper into higher-tier level tables, and crescendos at the boss fight.

---

## 8. Flagged Issues — things that resist temporal translation

Places where the board game's nature actively fights against RT translation. Each flag lists the problem, the chosen resolution, and the risk.

### F-1. "The player can stare at the map forever."

- **Problem.** The board game tolerates unbounded thinking. RT games, by convention, do not. If we preserve unbounded thinking we risk boredom; if we punish it with timers we betray the source.
- **Resolution.** **Preserve it.** Outside combat lock, the player has unlimited time. We accept the risk of boredom for players who expect twitch pacing — those players are not the target audience.
- **Residual risk.** Players trained on *Hades*/*Diablo* may find the early game slow and bounce off. Mitigation: onboarding should set expectations (e.g., "take your time — this is a thinker's dungeon crawler").
- **Severity.** Medium.

### F-2. "Combat is 100% deterministic in the source — RT adds skill expression."

- **Problem.** Source combat has zero player agency inside a round. Adding agency (parries, swing timing) changes the *feel* of combat even if the expected values match.
- **Resolution.** Accept the change. The core dice math is preserved; only the presentation adds skill. A passive player gets the same expected outcome as a board game player.
- **Residual risk.** Skilled players may trivialize content the source considered dangerous (e.g., late-game monsters). Mitigation: tune the parry window tight enough that perfect parries are rare even for experts.
- **Severity.** Medium. This is the biggest intentional departure from source and must be validated in playtest.

### F-3. "The no-revisit rule is a paper-and-pen rule."

- **Problem.** On paper, the no-revisit rule is enforced by the player's pen — once you draw a line through a cell, you can't draw through it again. In RT we must enforce it physically.
- **Resolution.** Visited cells **physically close** — floor collapses, walls seal, doorway bricks over. The player literally sees the door close behind them.
- **Residual risk.** Some players will misread the visual cue. Mitigation: the "you are about to block yourself" warning in the HUD catches misreads.
- **Severity.** Low.

### F-4. "The source has no concept of enemy aggro range."

- **Problem.** In the source, monsters are rolled on entry; they don't exist outside the cell. In RT they must be spawned, rendered, and animated, which implies *existence before entry*.
- **Resolution.** Monsters do not exist (visually) until the fog clears on reveal. They spawn instantaneously during the reveal animation. From outside the fog, there is no enemy.
- **Residual risk.** If the player is on a cell boundary, partial reveals could feel inconsistent. Mitigation: cell-snapped movement eliminates boundary ambiguity.
- **Severity.** Low.

### F-5. "Shop is 'end of turn' — but RT has no turn."

- **Problem.** Step 7 happens at the end of every turn in the source. RT has no turn boundary.
- **Resolution.** Shop is physicalized into shop rooms at fixed grid positions (see §2 Step 7). Source loses "visit every turn" but gains "movement-cost shop visits" — a clean trade.
- **Residual risk.** Some source purists will object that "the shop was always available." Mitigation: call out the design choice in the RTGDD rationale.
- **Severity.** Medium.

### F-6. "Potion-use timing has ambiguity already (A-2)."

- **Problem.** Source ambiguity: can potions be drunk mid-combat? The extraction defaults to "no." RT must pick a side.
- **Resolution.** **Honour "no."** Potion hotkey is disabled during combat lock. This is consistent with the source's default resolution and keeps combat tense.
- **Residual risk.** Some players will intuit "I should be able to heal mid-fight" from RPG convention. Mitigation: tutorial explicitly calls out "you cannot drink in combat — use your window between encounters."
- **Severity.** Low.

### F-7. "Dracular is deterministic dice in the source — RT makes him a pattern-reading boss."

- **Problem.** The source Dracular is just "9 AD vs 9 DD in standard combat rounds." Making him a multi-phase pattern-reading boss adds a lot the source doesn't authorize.
- **Resolution.** Keep the math (still rolling 9d6 per strike, 9d6 per defence) but diversify the *presentation* into 3 attack patterns. From a dice-pool POV, he's identical to the source.
- **Residual risk.** The fight may feel "gamey" vs "source-faithful." Mitigation: the underlying expected-damage-per-round matches the source exactly, so run length and lethality are preserved.
- **Severity.** Medium.

### F-8. "Found potions are usable this turn, bought potions next turn — translating 'turn' is hard."

- **Problem.** The source's potion-delay rule depends on the concept of "turn," which RT abolishes.
- **Resolution.** Found potions enter the stockpile immediately (usable in the next non-combat moment). Bought potions enter with a 5-second real-time cooldown. This preserves the *direction* of the rule (bought is slower than found) without requiring a "turn" abstraction.
- **Residual risk.** 5 seconds is arbitrary. Mitigation: make it tunable in a config and playtest.
- **Severity.** Low.

### F-9. "There's no concept of run length in the source — it's bounded only by the grid."

- **Problem.** Source runs are 20–40 minutes, paced by the player's writing speed and thinking time. RT runs will be whatever we make them.
- **Resolution.** Target 15 minutes per run. Adjust locomotion speed, combat cadence, and cell-reveal speed to land there.
- **Residual risk.** Pure fidelity fans want 30+ minute runs. Mitigation: include a "slow mode" that halves locomotion speed and doubles combat windups for a 30-min run experience.
- **Severity.** Low.

### F-10. "The board game is saveable/pausable anywhere."

- **Problem.** The source's `GameLoop.md` notes the digital loop is "trivially pausable/save-anywhere." In RT with beat-based combat, pausing mid-parry creates save-scumming risk.
- **Resolution.** **Save-anywhere outside combat lock.** Pause is always allowed (as a UX pause, no mechanical effect). Save-and-quit is allowed only outside combat lock. Mid-combat quit forfeits the run.
- **Residual risk.** "My cat walked on my keyboard and I died mid-combat" is a real support ticket. Mitigation: a ~1-second "press ESC to pause" grace period at the start of each combat, after which pause suspends rendering but does not save-scum.
- **Severity.** Medium.

### F-11. "Combat math is averaged over many rolls — RT shows each roll live."

- **Problem.** In the source, you roll all your attack dice at once and count sixes. Expected value is smooth over a long run. In RT, if we literally show 9 dice rolling for Dracular, we've turned combat into a slot machine.
- **Resolution.** The underlying roll still happens once per strike, but the *visual* is the animation (sword swing, spell cast). The resulting number of hits is communicated via damage numbers/ particles, not via rolling dice on-screen.
- **Residual risk.** Source purists will miss the dice. Mitigation: an optional "classic mode" overlay shows the dice rolls as floating d6s next to the character, for nostalgic players. Hidden by default.
- **Severity.** Low.

### F-12. "The atomic loop is 4 ticks — but RT beats into one flowing exchange."

- **Problem.** The source has 4 discrete combat ticks per round. In RT, the 4 ticks overlap into a single 1.5-second call-and-response dance. Is this still "4 ticks"?
- **Resolution.** Yes — the 4 beats are preserved in structure (windup / parry window / swing window / block frame) but compressed in wall-clock time. A player who wants to count ticks can; a player who wants to flow through combat also can.
- **Residual risk.** Complex internal state to track. Mitigation: state machine with 4 states per combat round, one tick per state.
- **Severity.** Low.

---

## Appendix — Quick-reference timing constants (draft)

These are the proposed defaults for the RT balance stage. All are tunable.

| Constant | Value | Where used |
|---|---|---|
| `LOCO_WALK_SECS_PER_CELL` | 0.40 | Cell locomotion |
| `LOCO_HUSTLE_SECS_PER_CELL` | 0.25 | Hotkey hustle |
| `LOCO_COMMIT_WINDOW_MS` | 120 | Movement cancel window |
| `REVEAL_FOG_CLEAR_MS` | 250 | Room reveal |
| `REVEAL_AGGRO_DELAY_MS` | 400 | Monster aggro after reveal |
| `REVEAL_PLAYER_GRACE_MS` | 150 | Player grace before first windup |
| `PICKUP_JUICE_MS` | 300 | Treasure/potion pickup feedback |
| `COMBAT_BEAT_CYCLE_MS` | 1500 | Full monster-player exchange |
| `MONSTER_WINDUP_MS` | 600 | Monster attack telegraph |
| `MONSTER_STRIKE_FRAME_MS` | 200 | Parry window duration |
| `PARRY_PERFECT_WINDOW_MS` | 80 | Perfect parry threshold |
| `PLAYER_SWING_WINDUP_MS` | 300 | Player attack windup |
| `PLAYER_SWING_COOLDOWN_MS` | 800 | Time until next swing |
| `PLAYER_CHARGE_THRESHOLD_MS` | 400 | Hold-to-charge threshold |
| `BOSS_BLOCK_FRAME_MS` | 400 | Dracular block shimmer |
| `POTION_QUAFF_MS` | 150 | Per-quaff duration |
| `POTION_BOUGHT_COOLDOWN_MS` | 5000 | Bought potion delay |
| `SHOP_OPEN_MS` | 300 | Shop dialogue open |
| `BLOCK_CHECK_HZ` | 10 | BFS reachability re-check rate |
| `DEATH_CAM_MS` | 1500 | Player death animation |
| `WIN_CAM_MS` | 3000 | Dracular death animation |
| `BLOCKED_LOSE_CAM_MS` | 2000 | Blocked camera pullback |
| `RUN_START_FADE_MS` | 2000 | Intro fade-in |
| `TARGET_RUN_LENGTH_MIN` | 15 | Match-arc target |

---

## End of TemporalMap.md
