# RT-GameLoop — Solo Dungeon Dash

> RealTimeForge Stage RT-D3: Revised Game Loop
> Source: `output/solo-dungeon-bash/GameLoop.md`, `GameLoop.mmd`
> Analysis inputs: `realtime/analysis/TemporalMap.md`, `GenreCrystallization.md`, `ConflictModel.md`
> Target: Solo Dungeon Dash — 2D top-down Action Roguelite / Dungeon Crawler

---

## 0. Preamble — from turn-based to beat-based

The source game *Solo Dungeon Bash* has a strictly sequential, player-paced,
discrete-time loop with four nested scopes: combat round → turn → level-row →
run. There is not a single wall-clock second anywhere in the original rulebook.

*Solo Dungeon Dash* rebuilds that hierarchy on a wall-clock foundation. The
four loops survive, but each is now measured in real milliseconds, each
contains genuine player agency at every instant, and the "turn" unit has been
dissolved into the "room." The Dice Tray — a visible 3D physics panel in the
lower-right HUD — is the single on-screen element that carries the source's
identity through every loop level, because every combat roll, attack, and
defence is visualized there in real time.

This document maps the four loops, then walks through each in increasing
detail, closing with exit conditions and a pacing diagram.

---

## 1. The Four Loops (Hierarchy)

Solo Dungeon Dash has **four nested loops** running at very different
granularities. Higher loops contain lower loops; lower loops drive higher
loops to completion.

### 1.1 Atomic loop — Frame tick (16.67ms @ 60fps)

The innermost loop is the render tick. Every frame the engine runs a fixed
sequence:

1. **Input poll** — read WASD / LMB / Space / Q / ESC.
2. **Update physics** — player position, velocity, knockback, iframes.
3. **Update AI** — advance monster beat state machines (see §2).
4. **Collision resolution** — swing arcs vs. hurtboxes, pickup sensors, cell
   walls, doorway triggers.
5. **Render** — sprites, tilemap, HUD, Dice Tray, particle VFX, ink-line map
   overlay.
6. **Audio tick** — layered music bus, positional SFX, heartbeat crossfade.

The frame tick does **not** correspond to any rule from the source game. It is
invented purely to host the higher loops. It is also where input latency lives
— a single missed parry by one frame (16.67ms) is the difference between a
clean deflect and a 2-pip hit.

Target: rock-solid 60fps on PC and Steam Deck, 30fps floor on mobile.

### 1.2 Combat beat — Per beat (~1.5s, varies 1.0–2.5s by enemy)

A **combat beat** is one full Roll-and-Parry exchange: monster telegraphs,
parry window opens and closes, strike resolves, opening appears, player swings
and rolls the Dice Tray, damage applies, monster recovers. One beat is the
real-time analogue of one "combat round" in the source game's 4-step atomic
loop. The beat has a fixed skeleton but variable wall-clock length depending
on the enemy's AD count and pattern.

- **Short beats:** Orcs, Rats (~1.0s) — small jab windups, short openings.
- **Medium beats:** Skeletons, Goblins (~1.5s) — standard cadence.
- **Long beats:** Demons, Wizards, boss phases (~2.0–2.5s) — heavy windups,
  longer parry windows, longer openings, higher stakes.

A normal mob dies in 1–3 beats. Dracular's Nine Dice Duel is ~60 beats across
3 phase patterns within the 90s target boss fight.

### 1.3 Room loop — Per room (~15–45s)

A **room** is the real-time analogue of the source game's "turn." One room
starts when the player commits to crossing a doorway into an unvisited cell;
it ends when the contents are fully resolved and the doorways reopen. A room
can hold an empty cell (~3s), a treasure pickup (~5s), a potion pickup (~5s),
a Shop Shrine (~10–60s, player-paced), or a combat encounter (~10–30s).

Median combat room: ~20s. Median clean-traversal room: ~5s. Target average
across a run: ~15–25s per room.

### 1.4 Run loop — Per match (~20 min median, 12–35 min spread)

A **run** is one full expedition from spawn on the Start cell to either
Dracular's corpse, the player's corpse, or the self-blocked camera pan-back.
Target median 20 minutes. 80th-percentile window 12–35 minutes. A run
traverses ~40–70 rooms out of the 90 cells on the 9×10 grid, fights ~20–40
monsters, visits ~3–6 Shop Shrines, and ends in the Nine Dice Duel at the End
room.

Runs never resume. Save-and-quit is allowed outside combat lock (suspends the
dungeon state), but on death or victory the seed is discarded.

---

## 2. The Combat Beat in Detail

This is the hardest translation in the whole pipeline because the source's
combat round has **zero player agency** (per `Mechanics.md` M9), yet a
real-time combat loop that asks nothing of the player is not combat at all.
The solution is **Roll-and-Parry**: the dice math of the source is preserved
exactly, but the player's input layer decides *whether* and *when* the dice
get to roll.

Below is a millisecond-level breakdown of one full combat beat against a
standard monster (e.g., a Skeleton, AD 3, DD 0, 1 HP). All numbers come from
`TemporalMap.md §5` and the `ConflictModel.md` balance appendix.

### Beat timeline — monster cycle

```
t = 0     ms  MONSTER BEAT START
              Monster turns to face player, locks onto hurtbox
              Dice Tray pulses ready
              Combat music layer bumps up

t = 0     ms  TELEGRAPH — windup animation begins
              Visual tell scales with AD count:
                AD 1  = small jab
                AD 3  = overhead chop
                AD 7  = wide sweep
                AD 10 = screen-wide slam (Demon)
              Audio: weapon hiss, armor rustle, low growl

t = 400   ms  PARRY WINDOW OPENS
              Player can press LMB to parry
              Player can press Space to dodge-roll (iframes 300ms)
              Player can do nothing (auto-defend falls through)

t = 400   –
    480   ms  PERFECT PARRY zone (80ms)
              LMB in this 80ms window:
                - ALL incoming hits cancelled, regardless of DD roll
                - Monster staggered for 1200ms opening
                - Screen flash, sharp chime, hit-stop 50ms

t = 480   –
    600   ms  NORMAL PARRY zone (120ms)
              LMB in this zone:
                - Roll DD pool in Dice Tray (count 6s)
                - Each 6 cancels one incoming hit
                - Monster enters 1000ms opening regardless
                - Medium chime, slight hit-stop

t = 600   ms  PARRY WINDOW CLOSES
              If no input: auto-defend fires
                - Roll DD pool automatically (same math as normal parry)
                - Monster does NOT enter opening
                - Player is defending but not creating a window

t = 700   ms  STRIKE RESOLVES
              Count unblocked monster hits:
                - For each unblocked hit, HP -= pip cost (1 or 2 per hit
                  depending on monster tier, see ConflictModel §3)
                - Heart pips subtract with discrete 150ms drain anim
                - If HP <= 0  => DEATH CAM 1500ms => LOSS
                - Screen edge flashes red on hit
                - If HP < 5, heartbeat SFX layer begins

t = 1000  ms  OPENING WINDOW BEGINS (if parried or perfect-parried)
              Monster guard is DOWN for 1000ms (normal) or 1200ms (perfect)
              Enemy enters "stunned" pose, visible vulnerability glow
              Dice Tray highlights, ready to receive an attack roll

t = 1000  –
    1700  ms  PLAYER SWING WINDOW
              LMB quick-swing (300ms windup, then hit frame)
                OR
              Hold LMB >= 400ms for charged swing (reroll 1 die advantage)
                OR
              Do nothing => opening expires, monster recovers

t = ~1300 ms  ON SWING CONTACT
              DICE TRAY ROLL
                - N d6 physics-tumble in lower-right tray (500ms animation)
                - Count 6s = Hits
                - 0 hits   => GUARD SPARK (clang VFX, sword glances off)
                - 1+ hit  => MOB DIES (1 HP, one clean hit kills)
                - Extra 6s => CLEAVE (hits adjacent grouped mob if any)
              Kill VFX: ink-splash, ragdoll, loot drop (if any), combo SFX

t = 2000  ms  MONSTER RECOVERS / COMBAT ENDS
              If monster dead: room unlocks, walls dim, doorways reopen,
                return to Room Loop
              If monster alive (rare — only if player missed opening):
                Return to t=0, next beat begins
```

### Boss variation — Dracular (Nine Dice Duel)

The boss fight reuses the beat skeleton but layers three timing variants:

- **Fast Jab pattern** (windup 400ms, opening 800ms, tight parry)
- **Heavy Slam pattern** (windup 1000ms, opening 1200ms, big tell)
- **AoE Spin pattern** (windup 1500ms, opening 600ms, must dodge-roll)

All three patterns still roll **9 AD** in the Dice Tray on strike and **9 DD**
on the player's swing contact. The source math is untouched — only the
presentation varies. Additionally, on every player swing contact Dracular has
a **400ms block frame shimmer**: swings that land inside the shimmer go
through the 9 DD roll (and usually bounce), swings that land outside it
bypass the shimmer (clean hit, still rolled but not defended).

Dracular has 3 HP for the purposes of the RT boss fight (one pip per phase);
each phase down changes his pattern rotation. The total expected beat count
is ~60, landing the fight near 90 seconds median.

### Why this math is source-faithful

- **Monster AD still rolls N d6 per strike** — expected damage per round is
  identical to the source.
- **Player DD still rolls N d6 per parry/auto-defend** — expected block rate
  is identical to the source.
- **Player AD still rolls N d6 per swing** — expected kill rate is identical
  to the source (one unblocked hit kills 1-HP mobs).
- **Dracular 9 AD / 9 DD preserved exactly.**
- A passive player who never parries and never dodges gets exactly the
  board-game outcome distribution. Active play adds skill expression on top of
  the same underlying RNG.

---

## 3. The Room Loop in Detail

A **room** is bounded by two doorway crossings: the one that brings the
player in, and the one that takes them out. Here is the full beat-by-beat
walk-through.

### Step A — Doorway hover (player is in previous room)

Eight king-adjacent unvisited cells are available as exits. The player
hovers each doorway to see a minimal content fog (silhouette only: empty,
monster, treasure, potion, Shop Shrine). The persistent reachability BFS
(10 Hz) highlights doorways that would block the path to End with a red
border. The player may stand here indefinitely — there is no timer outside
combat lock.

### Step B — Commit and cross (120ms cancel window)

Once the player steps into a doorway, a 120ms cancel window fires. Reverse
direction in that window = no commit, returns to hover. After 120ms the
commit is final: the previous cell seals behind the player (floor collapses,
wall bricks over — visible VFX enforcing the no-revisit rule), and the ink
line is drawn from previous cell to current cell on the map overlay.

### Step C — Fog dissolve and reveal (250ms)

The new cell's fog dissolves. Camera slides to center on the octagonal room.
Ambient light shifts to the room type's palette (gold for treasure, blue for
potion, red for monster, neutral for empty, purple for Shop Shrine). The
pre-seeded content becomes visible in place: pile, bottle, shrine, or
monster model.

### Step D — Content identification

Based on what resolved:

- **Empty** — ambient dust motes, low hum. Player may idle-think. Skip to
  Step G.
- **Treasure** — sparkling pile. Auto-pickup on walking over. 300ms juice
  feedback, Treasure counter +1. Skip to Step G.
- **Potion** — bottle on floor. Auto-pickup on walking over. 300ms juice
  feedback, Potion stockpile +1. Skip to Step G.
- **Shop Shrine** — altar with NPC keeper. Walking up triggers a 300ms open
  animation; world pauses, ambient music dims. Player spends Treasure across
  3 mutually-exclusive gear slots and consumable potions (newly bought
  potions get a 5s chilling cooldown). Player exits when ready. Skip to
  Step G.
- **Monster** — **ROOM LOCKS**. Walls glow red, doorways shimmer shut. Aggro
  begins at t+400ms (reveal fog clears at t=250ms, aggro delay 150ms grace
  on top). Enters **Combat Beat Loop** (see §2) and stays there until mob
  dies or player dies. On mob death, walls dim and doorways reopen.

### Step E — Combat resolution (if applicable)

The combat beat loop runs for 1–3 beats against a normal mob (most mobs die
on the first successful parry + swing). HP may change. Potions are
**inaccessible** during combat lock — the Q hotkey is greyed out. This
enforces source ambiguity A-2 ("no mid-combat drinking").

If player HP reaches 0 during Step E, the Run Loop exits via **LOSS — Dead**
(death cam 1500ms, summary screen).

### Step F — Room unlocks

Walls dim. Doorways reopen. Mob ragdoll remains visible. Any pickup (loot
drop from a killed mob, such as a bonus potion on rare drops) is auto-
collected.

### Step G — Post-room player phase

The player is now standing in a resolved room. They may:

- **Use Potion** — Q hotkey, 150ms quaff, HP +1 per quaff. Vulnerable frame
  100ms. Can queue multiple quaffs. Cap 17. Newly-bought potions have a 5s
  chilling cooldown before they become quaffable.
- **Check map overlay** — V key / middle-click. Shows the full 9×10 grid
  with visited cells inked in. Reachability overlay shows which doorways
  lead to End.
- **Pause** — ESC. Outside combat lock, pause is free. Inside combat lock,
  a 1-second grace at the start of combat allows emergency pause.
- **Idle-think** — no penalty. The world waits.

### Step H — Choose next doorway

Return to Step A. The loop repeats until the player enters the End room
(forced boss fight), dies, or is blocked.

---

## 4. The Run Loop in Detail

A run is one full expedition. The arc has four distinct acts plus the boss
plus the outcome.

### Act 0 — Spawn (t = 0s)

- Run seed generated (deterministic per run).
- 9×10 grid pre-seeded with content (pre-rolled from seed).
- Player spawned on Start cell (row 1, column 5) at HP 17, 1 AD, 1 DD, 0
  Treasure, 0 Potions, no gear.
- 2s fade-in animation with ambient dungeon audio.
- First 3 legal doorways highlighted for onboarding.
- Music: calm dungeon ambience.

### Act 1 — Early arc (rows 1–3, ~0:00–4:00 wall-clock)

- Mobs: Rats (AD 1), Orcs (AD 2), Wolves (AD 2).
- Combat: 1-beat kills are common. Risk is low.
- Goal: orient, find the first Shop Shrine (usually row 2–3), start a Treasure
  stockpile, grab any early potions.
- Player is **learning the route**. Most deaths here are self-inflicted
  mis-parries, not strategic failures.
- Pacing: ~8–15 rooms traversed. ~6 combats, all easy.

### Act 2 — Mid arc (rows 4–7, ~4:00–12:00 wall-clock)

- Mobs: Skeletons (AD 3), Goblins (AD 4), Ogres (AD 6), Wizards (AD 7).
- Combat: 1–3 beat fights. Multiple hits per fight now matter.
- Shop visit: second Shop Shrine around row 4 or 5. This is the critical
  upgrade point — most runs buy their first real AD pool upgrade here.
- Goal: upgrade to 3+ AD / 3+ DD, stockpile 4–6 potions, reach row 7
  without major HP loss.
- Player is **routing carefully**. Wrong commits now cost real HP.
- Pacing: ~20–30 rooms traversed. ~15 combats, mix of easy and scary.

### Act 3 — Deep arc (rows 8–10, ~12:00–18:00 wall-clock)

- Mobs: Demons (AD 9–10), Liches (AD 8, with DD), rare 2-mob grouped rooms.
- Combat: 2–4 beat fights. Every hit costs 2 pips. Heartbeat SFX layer
  triggers around HP 5.
- Third (and sometimes last) Shop Shrine in row 8–9 — final upgrade chance.
- Goal: reach End room with at least 7 HP, 4 AD, 3 DD, 3+ potions in
  stockpile.
- Player is **grinding and risk-managing**. Reachability BFS is now loud —
  one wrong commit can strand.
- Pacing: ~10–15 rooms traversed. ~8 combats, mostly scary.

### Act 4 — Boss approach and Nine Dice Duel (~18:00–20:00 wall-clock)

- End room detected within 2 cells. Music layer shifts: low organ, heart-
  beat.
- Doorway into End glows purple. Crossing is a final commit — cannot back
  out.
- Nine Dice Duel begins: 90 seconds median, 3 boss phases, ~60 beats.
- Dracular cycles through Fast Jab / Heavy Slam / AoE Spin patterns with 9
  AD and 9 DD per exchange.
- Potions can be used between beats (no combat lock during opening frames,
  within limits — see §6).

### Act 5 — Outcome

Exactly one exit fires (see §5 below), a summary screen shows, then back to
main menu.

---

## 5. Exit Conditions

The Run Loop has exactly four exits. All are terminal for the current run.

| Exit | Where it fires | Camera | Summary header |
|---|---|---|---|
| **WIN — Dungeon Cleared** | Dracular's final phase hits 0 HP, beat resolves | 3s victory cam, ink-splash VFX, final-hit slow-mo | "Dracular Slain — Run Time 19m42s" |
| **LOSS — Dead** | Player HP reaches 0 during strike-resolve of any beat | 1.5s death cam on killing hit | "Run Over — Died on row 7, killed by Wizard" |
| **LOSS — Blocked** | Reachability BFS returns "End unreachable" after a commit | 2s camera pullback to full 9×10 map, replays the fatal commit | "Stranded — No path to End from cell (4,8)" |
| **Quit (soft)** | Player selects Quit to Menu outside combat lock | Instant fade | "Run Abandoned" (save-and-quit resumes next session; mid-combat quit forfeits run) |

There is **no time-out exit**. The source game tolerates unbounded thinking
and so does RT-SDD. A player may stare at the map for an hour between rooms
with no penalty. This is a deliberate preservation of the source's
player-paced identity, defended in `TemporalMap.md §8 F-1`.

Inside combat lock the player cannot quit cleanly — ESC opens a UX pause
(render frozen, no save) with a 1-second grace window at combat start for
emergency pauses. Mid-combat forced quit forfeits the run.

---

## 6. Pacing Diagram

Tension intensity over wall-clock time for a typical ~20-minute run. The Y
axis is subjective "felt pressure"; the X axis is minutes into the run.

```
Intensity
    |                                              *** BOSS
  HI|                                             /     \
    |                                            /       \
    |                                           /         *
    |                              .^.         /         
    |                   .^.       /   \    .^./          
    |              .^. /   \ .^. /     \  /              
    |   .    .^.  /   V     V   V       \/               
    |  / \  /   \/                                       
    | /   \/                                             
  LO|/                                                   
    +-------------------------------------------------------->
    0m      4m       8m        12m       16m       18m    20m
    ^       ^        ^         ^         ^         ^      ^
    start   first    mid-arc   row-7     row-9     end    boss
            combat   shop      grind     shop      door
    EARLY ARC          MID ARC          DEEP ARC  BOSS   WIN/LOSS
```

**Phases in the pacing diagram:**

- **Early arc (0–4 min):** Calm. Low sawtooth of room-to-room combats
  against AD 1–2 mobs. Player is learning and routing.
- **Mid arc (4–12 min):** Rising amplitude. AD 3–7 mobs require real parry
  reads. Shop visits create calm islands (tension drops to zero during
  dialogue). Tension compounds because the player is healthier but the risk
  per room is rising faster.
- **Deep arc (12–18 min):** High baseline with sharp spikes on every Demon
  or Lich encounter. Heartbeat SFX kicks in regularly. Reachability warnings
  are loud. Any wrong commit can be fatal.
- **Boss approach (18–19 min):** A deliberate dip — the corridor to End is
  usually empty. The player has 30–60 seconds of heavy silence before the
  fight, to reposition, quaff potions, and read their gear.
- **Nine Dice Duel (19–20:30 min):** Sustained maximum intensity for 90
  seconds. Dracular's pattern rotation creates micro-rhythms within the
  spike. The fight ends in WIN or LOSS.
- **Summary and menu (20:30+ min):** Zero tension. Runs end cleanly.

The macro rhythm: **calm → low sawtooth → rising sawtooth → tight spikes →
dip → boss plateau → flatline**. This is intentionally closer to *Darkest
Dungeon* or *Sekiro* than to *Hades* or *Diablo* — more breathing room
between fights, more tension per fight.

---

## 7. Diagram Reference

See `RT-GameLoop.mmd` for the full Mermaid flowchart. The diagram uses four
subgraphs (Frame Tick / Combat Beat / Room Loop / Run Loop) with
color-coded terminal states:

- **Green (started):** Run start, main menu.
- **Yellow (win):** Victory / Dungeon Cleared.
- **Red (lose):** Dead / Blocked.

Key visible elements in the diagram that the player also sees on-screen:

- **Dice Tray** — called out explicitly in the combat beat subgraph as the
  node that rolls N d6 on every swing contact. This is the identity anchor
  that keeps Solo Dungeon Dash faithful to the source's count-6s-on-d6
  personality.
- **Room lock** — visible in the Room Loop subgraph as the transition that
  seals doorways during combat.
- **Cell-seal** — visible as the "previous cell seals behind" step, enforcing
  the no-revisit rule physically.
- **Reachability BFS** — visible as the "End unreachable" branch that exits
  to LOSS — Blocked.

The frame tick loop is shown as a discrete subgraph to make explicit that it
is the 60 Hz heartbeat underneath all three higher loops — the combat beat
loop, the room loop, and the run loop all run *on top of* the frame tick,
not in parallel to it.

---

## 8. Design Pillar Check

Every loop element above must serve at least one corner of the Identity
Triangle (`GenreCrystallization.md §8`):

- **Routing Puzzle** — Room loop's doorway hover, cell-seal VFX, BFS
  reachability check, pre-seeded content fog, exit LOSS — Blocked.
- **Parry Combat** — Combat beat's parry window, perfect-parry zone,
  opening window, dodge-roll fallback, boss block frame shimmer.
- **Dice Tray** — Combat beat's Dice Tray roll on every swing, count-6s
  cleave, monster AD roll on strike, Dracular's 9 AD / 9 DD dice pool.

Each loop contains elements from all three corners. The run loop acts as the
container that lets the player express all three simultaneously.

---

*End of RT-GameLoop.md*
