# Agency Translation Model — Solo Dungeon Bash → Real-Time

> Stage RT-2 of the RealTimeForge pipeline. This document defines how the player's decision-making and skill expression translate from the original 2007 solo roll-and-write to a real-time interactive game.

---

## 1. Summary of Source Agency

Solo Dungeon Bash is an **extremely low-agency, high-strategy** game by modern standards. The player's entire input surface across a 20–40 minute run is five discrete decision types — and one of them (combat) is pure dice with zero player input after engagement. The game's "feel" is the feel of a pen hovering over graph paper while the player squints at a d6 and mutters "please be a 6".

**Agency character:**

- **Strategic, not tactical.** The meaningful decisions are multi-turn routing questions ("do I push up to row 5 now or farm row 3 for two more turns?"). Single-turn decisions are almost trivial once you know the level tables.
- **Paced and deliberate.** Every decision is made at a complete stop. There is no clock, no tempo, no interruption. A turn can take 10 seconds or 10 minutes; the game does not care.
- **Zero dexterity.** No aiming, no timing, no coordination. The only physical skill is drawing a tidy grid on graph paper.
- **No reaction, no interrupt.** The player cannot respond to a bad roll. If the monster rolls four 6s and the player rolls no 6s on defence, the damage applies instantly and the player watches.
- **Information is public.** Tables are printed on the sheet. There's no hidden state except unrevealed rooms and un-rolled dice. A perfect-information planner could play the game optimally with a calculator.
- **Binary outcome, low score dimension.** You live or you die. There's no "how well did you do" other than "did you win".
- **Tactile/kinaesthetic agency is ink.** The core joy layer the board game offers that most digital roll-and-writes lose is the *drawing* — circling the square, drawing the line, writing the new HP total over the old one. This is agency as craft, not decision.

**Agency density (source):** Roughly **2.5 meaningful decisions per turn** (pick square, use potions?, buy what?). Total meaningful decisions across a full run: ~40–60. That's less than a typical mobile puzzler gives you in 90 seconds.

**Skill ceiling (source):** Very low in the traditional sense. An expert player is separated from a bad player almost entirely by:
1. Row-table memorisation (knowing which rows are potion rows).
2. Reachability planning (not painting yourself into a blocked corner).
3. Shop sequencing (when to buy defence vs. offence).

Everything else is variance. Two equally skilled players can differ by 80% in win rate purely on dice.

---

## 2. Per-Action Translation Table

This is the spine of the translation. Each board-game action becomes a specific real-time agency pattern, with explicit skill layers.

| # | Board Game Action | Class | RT Agency Pattern | Skill Expression Layers |
|---|---|---|---|---|
| A1 | **Pick next square** | Strategic | **Direct movement control** on a 2D tile-grid — WASD/stick steers the hero, adjacent walkable tiles light up; the player nudges into one. Diagonal movement is free. | Routing/planning, prediction, positioning, light execution precision (don't bump the wall) |
| A2 | **Roll d6 for room contents** | Random | **Auto-resolved reveal with juice** — the moment the hero crosses the tile threshold, the fog dissolves and content materialises with a satisfying "thunk". No player input. | (none — this is pure game state; the agency is upstream in A1) |
| A3 | **Fight monster (the fight itself)** | Tactical | **Auto-attack with cooldown + hotkey ability (Parry)** — the hero auto-swings at the target in melee range on a rhythm; the player holds a single *Parry* key to roll defence dice at the moment the monster winds up. One-button timing game. | Timing window, reaction speed, decision under pressure ("do I parry or dodge away?"), rhythm |
| A4 | **Use a potion** | Strategic (resource) | **Hotkey ability** — pressing Q drinks one potion (+1 HP), usable *any time including mid-combat*. This is an explicit RT-NATIVE upgrade from source rule F-2. | Resource management, decision under pressure, timing (drink on the parry window, not during a swing) |
| A5 | **Buy Big Sword / Big Axe** | Strategic | **Context menu at Shop Shrine** — end-of-turn safe tile opens a radial/hotbar shop; clicking the icon commits the purchase. No pause; the shrine is a safe zone where monsters cannot spawn. | Resource management, build planning, opportunity cost |
| A6 | **Buy Buckler / Shield** | Strategic | Same as A5 — context menu / radial | Resource management, build planning |
| A7 | **Buy Spiky / Magical Armour** | Strategic | Same as A5 — context menu / radial | Resource management, build planning, risk profiling |
| A8 | **Buy potion packs (1/3/6)** | Strategic | **Quick-buy hotkey at Shop Shrine** — holding the shop key while clicking the potion icon cycles 1→3→6 pack size. Faster than re-opening the menu. | Resource management, reserve planning |
| A9 | **Enter the End square (commit to boss fight)** | Strategic | **Commit-then-reveal** — the End tile is visually special (a glowing sealed door). Walking into it triggers a 2-second "Are you sure?" hold-to-confirm; releasing cancels, holding confirms. No accidental boss starts. | Decision under pressure, self-assessment |
| A10 | **Path planning (meta-level, many turns)** | Strategic | **Map overlay (hold Tab)** — shows the full 9×10 grid, visited cells, reachability highlight, suggested safe rows. Does not pause time in combat but does pause the world during exploration (a "minimap pull-up"). | Routing/planning, prediction, spatial reasoning, pattern recognition |

**Legend for Classes:**
- **Strategic** = multi-turn, plan-based decision, not time-critical
- **Tactical** = in-the-moment, sub-second decision
- **Social** = involves another agent (none in solo)
- **Random** = no player input; stochastic

Notice that A2 is deliberately marked "Random" and not folded into A1. This preserves the source game's "reveal tension" — the moment between choosing a door and seeing what's behind it. That moment is sacred.

### Action-by-action deep notes

**A1 (Pick next square) — Why direct movement control?** We considered three alternatives: (a) point-and-click pathfinding (click the destination tile, the hero walks), (b) drag-to-draw (like the source: drag a line from the current tile to a neighbour), and (c) direct WASD/stick movement. We chose (c) because:
- Point-and-click removes the *committal feel* of stepping into a room. You want the player to *lean into* the decision.
- Drag-to-draw is closer to the source but feels clunky for rapid play and awkward on gamepad.
- Direct movement lets the player "hesitate at the threshold" of a tile — a small physical gesture that mirrors the board game's pencil-hovering-over-the-grid moment.

**A3 (Fight monster) — Why a single Parry button instead of dodge/block/parry?** The source game has exactly one combat decision (nothing, you roll defence dice automatically). Adding a single button is the *minimum* addition that preserves the "count your 6s" identity while giving the player something to do. More than one button turns this into an action game and breaks the source's contract.

**A4 (Use a potion) — Why a hotkey instead of a menu?** The source gives you a full decision phase (step 6) to fiddle with potions. In RT, a menu-based potion UI would mean pausing combat to drink — unthinkable. The hotkey is the only idiom that fits. This is also why we resolve ambiguity F-2 in favour of "yes, drink any time": the game physically cannot work otherwise.

**A9 (Enter End) — Why hold-to-confirm instead of a modal dialog?** A modal dialog breaks flow, feels bureaucratic, and in Stories.csv S025 it's already specified as "a confirmation dialog". We're deliberately upgrading that: hold-to-confirm is diegetic (the hero is *pushing against the door*), it's reversible mid-press, and it doesn't open a pop-up during gameplay. Same safety, better feel.

**A10 (Path planning) — Why make the map a hold-Tab overlay instead of a persistent minimap?** A persistent minimap drags attention away from the main grid, splits focus, and makes the 9×10 grid feel smaller by giving the player a second view. Hold-Tab keeps attention on the hero until the player *needs* to plan, at which point they get the full board-game view. This matches how the physical game is played: the player only looks at the whole grid at pivot moments, not constantly.

---

## 3. Agency Density Map

The source game has rest beats measured in *minutes*. The RT version needs to decide: do we honour that calm or commercialise it into constant input?

**Decision: Preserve the calm, but concentrate the agency into short spikes.**

Agency density profile per turn (RT version):

```
Phase                   Duration    Input load     Notes
----------------------  ----------  -------------  --------------------------------
Exploration (move)      3–8 sec     Low (steer)    Player drifts toward next tile
Reveal (content roll)   0.5 sec     None           Watch the tile animate
Resolve empty/item      0.5 sec     None           Pick-up is passive (walk-over)
Combat (if monster)     4–12 sec    High (parry)   Full attention, rhythmic
Step-6 potion window    0–2 sec     Optional       Quick-heal hotkey
Shop shrine (optional)  3–10 sec    Low (browse)   World quiet, no threats
Transition to next tile 1 sec       None           Brief breath
```

**Total average input load per turn: ~15–25 seconds, with a 4–12s combat spike.**

Compared to the board game's "decide in your head, write for 2 seconds, repeat", the RT version trades *time spent per decision* for *decisions per minute*. Meaningful decisions per run climb from ~50 to ~150–200 (every parry timing, every potion drink, every movement adjustment is now a decision).

**Deliberate design principle: No hidden agency creep.** The RT version must not fill every second with busywork. Rest beats — the walk between tiles, the breath after a kill — are where the strategic brain catches up. If playtesters say "it's too busy, I can't think", we cut encounter density, not routing complexity.

**The calm is the texture.** This is not a twin-stick shooter. This is a roll-and-write that learned to breathe on its own. The right reference points are Downwell's rhythm (burst-rest-burst), *not* Hades' constant pressure.

---

## 4. Skill Ceiling vs Skill Floor

**Source skill ceiling:** Low. Maybe 10% skill, 90% variance. A bad player dies on row 4. A great player dies on row 8. A god-tier player wins 30% of runs.

**RT version skill ceiling: Must be higher, or the game has no longevity.**

### Skill Floor (what a new player needs)
- Can move the hero.
- Can press Parry when something flashes on screen.
- Can find and click the Shop shrine at end of turn.
- Can press Q to heal when HP is low.

If you can do those four things, you can beat row 4 on Easy.

### Skill Ceiling (what separates amazing from good)

1. **Routing mastery (inherited from source).** Knowing which rows have potions (3/6/9), knowing when the BFS frontier is getting thin, knowing when to burn a turn farming vs push up. This is the source game's skill and we preserve it *verbatim*.
2. **Parry timing (RT-native).** The parry window is tight (200–250ms). A great player parries 80%+ of monster attacks; a mediocre player parries 30%. This directly multiplies survival math.
3. **Parry-under-load (RT-native).** Dracular attacks with multiple dice per wind-up; parrying 9 dice is a mini-rhythm-game in itself. Great players read the tell.
4. **Potion micro-timing (RT-native).** Using a potion *at the frame you drop below 5 HP* is the difference between surviving and dying to the next swing. Great players don't hoard; they drip-feed.
5. **Build sequencing (inherited and intensified).** The source game already rewards "Big Axe first", but in RT the earlier you get +2 AD the more monsters you one-shot, the less parrying matters, the less HP you lose. Compounding returns are sharper in RT.
6. **Threat assessment at reveal time (RT-native).** The 0.5s between tile-reveal and monster-aggro is enough for a great player to mentally commit to a parry pattern; a bad player freezes.

**Ceiling/floor ratio:** The RT version roughly doubles the skill span. In the source, a perfect player wins ~45% of runs; in the RT version with the parry layer, a perfect player should win 70–80% on Normal. That is *healthy* — it means the game has mastery to chase.

**Crucially: routing remains the TOP skill.** A godlike parry player who routes stupidly still loses to a mediocre parry player who routes well. The board game's strategic core is preserved as the dominant skill axis; parry is a secondary force-multiplier, not a replacement.

### Skill hierarchy (ordered)

1. **Routing & reachability** — 40% of skill impact. Inherited from source verbatim.
2. **Build sequencing** — 25% of skill impact. Inherited and sharpened. Knowing when to pivot from offence to defence, when to buy potion packs.
3. **Parry timing** — 20% of skill impact. RT-native. Can compensate for a mediocre build but cannot replace it.
4. **Potion micro-management** — 10% of skill impact. Drip-feeding, not hoarding.
5. **Tell reading** — 5% of skill impact. Eventually automatic for any dedicated player, but a real first-100-hours skill.

A new player touches skill 3 first (it's the most visible), climbs to skill 2 within 3–5 runs, and finally understands skill 1 around run 10. This is the correct progression order for keeping novices engaged: give them something to get better at *immediately*, while the deeper routing mastery reveals itself over time.

---

## 5. Automation Spectrum

For every action, where does it sit on the manual ↔ automated axis?

```
FULLY AUTOMATED   ←———————————————————————————————→   FULLY MANUAL
```

| Action | Position | Rationale |
|---|---|---|
| Move between tiles | **70% manual** | Player steers; game auto-snaps to adjacency; illegal tiles are walls |
| Content roll (d6) | **100% automated** | No input. This preserves source surprise. |
| Treasure/Potion pickup | **90% automated** | Just walk over it; counter bumps |
| **Combat damage resolution** | **50/50 (this is THE key decision)** | Auto-attack on your side, timed parry on defence |
| Potion use | **80% manual** | Hotkey; pressing is a choice, but the effect is instant |
| Shop purchase | **90% manual** | Context menu, player clicks item |
| Path planning / reachability | **60% manual** | Map overlay with optional reachability hint (toggleable) |
| Boss commit | **100% manual** | Hold-to-confirm |
| Blocked-path warning | **100% automated** | BFS runs silently; warning surfaces on the offending tile |

### Combat: The Automation Crux

The single biggest design question in translating Solo Dungeon Bash: **is combat auto-battle or action?**

Three options considered:

**Option A — Pure auto-battle (like Auto-Chess / The Bazaar).** Player enters combat, watches dice pools roll, walks away. Preserves source fidelity. **Rejected** because it makes combat a non-event in RT; players will skip-click through every fight and the game becomes a routing simulator with a cutscene attached.

**Option B — Full action combat (like Hades).** Player swings a sword, dodges, aims. **Rejected** because it shreds the source game's DNA. The dice pools, the "count 6s" aesthetic, the stat-modifier tableau — none of it maps to a twin-stick action game. You'd be building a different game.

**Option C — Auto-attack + timed parry (the Punch-Out / Sekiro middle ground).** ✅ **Chosen.**
- The hero auto-attacks on a rhythm dictated by attack dice (more AD = faster swings).
- The monster telegraphs its attack with a clear wind-up (0.8–1.2 sec).
- Player holds **Parry** at the flash. Holding rolls the defence dice; the count-6s result still determines blocks.
- Successful parries stagger the monster briefly.
- Missed parry = full unblocked damage.

This preserves the dice aesthetic (players see their d6 roll during parry, 6s highlight), honours the source combat math, *and* gives the player a real input to care about. It also scales naturally: more Defence Dice = more dice rolling in the parry window = higher block chance, matching the source economy.

**This is the RT-NATIVE combat core.** Call it **"Roll-and-Parry"**.

---

## 6. Input Method Decision

**Recommendation: Mouse + keyboard (PC), with first-class gamepad support.**

### Justification

1. **Agency density is moderate, not high.** Rest beats between combat spikes mean the game doesn't demand touch-screen immediacy or controller twitch-reflexes. A relaxed mouse-and-keyboard rig fits.
2. **The shop is a menu-heavy experience.** Six items, slot exclusivity, affordability filters — all much cleaner with a cursor than a D-pad or a thumb-tap.
3. **The map overlay is essential.** Players will hold Tab constantly to plan routes. This is peak PC ergonomics.
4. **Parry is a single button.** Mouse button or keyboard, either works. No complex combo input required.
5. **The audience overlap.** Solo roll-and-writes ship on Steam first. The digital roll-and-write space (Welcome To, Cartographers, Railroad Ink) lives on PC and tablet. PC is the proven home.
6. **Gamepad works as a second-class input because:**
   - Movement = left stick (feels great on grid).
   - Parry = right trigger (rhythmic, tactile).
   - Shop = radial menu on right stick (console-standard).
   - Map = touchpad / select button.
   - Works on Steam Deck handheld, which is the ideal venue for a 20-minute solo run.

### Why not touch-only (mobile)?
Parry timing windows of 200ms on a touchscreen are miserable. Touch input has 50–80ms latency plus finger travel, so the parry window becomes 120ms effective — below human reliable reaction time. Mobile would force us to widen parry to 400ms, which kills the skill ceiling. **Mobile is a port target, not a design target.** A simplified "auto-parry" mode can be offered for mobile specifically.

### Why not mouse-only (web-casual)?
Mouse-only loses the hotkey-potion, hotkey-parry idiom. You'd have to click a button for every heal, which breaks combat flow. Mouse-only is viable as a *browser demo* with reduced skill ceiling but not as the primary design target.

**Final: Design for mouse+keyboard on Steam. Port to gamepad natively. Ship mobile as a simplified companion.**

---

## 7. The Boss-Fight Problem (Dracular)

### The problem

In the source, Dracular is 9 AD / 9 DD. With a typical endgame loadout of 5 AD / 6–8 DD, the fight is roughly:
- Monster rolls 9 dice, expected 1.5 hits. Player blocks ~1.33. Net ~0.17 damage per round.
- Player rolls 5 dice, expected 0.83 hits. Monster blocks 1.5. Net ~0 damage per round — the fight can literally stall.

It's a **coin-flip endurance slog**, 10–30 rounds of dice clacking. It is, by the standards of 2007 print-and-play, perfectly fine. By the standards of 2026 real-time games, it is **an unplayable disaster** — the climax of your 25-minute run is watching numbers scroll for 45 seconds with no input.

### The RT-NATIVE solution: **"The Nine Dice Duel"**

Dracular is not a monster. Dracular is a **nine-phase rhythm boss**. Each phase corresponds to one of his nine Attack Dice.

**RT-NATIVE Mechanic: The Nine Dice Duel**

1. **Phase Structure.** The fight has **9 Phases**, one per Attack Die. At each phase, Dracular powers up one die into a "Loaded Die" (visible, glowing, spinning above him). The player must handle it before the next phase begins.
2. **Per-Phase Loop (~6 seconds each):**
   - Dracular winds up his Loaded Die and hurls it as a slow, telegraphed attack.
   - The player must do ONE of three things:
     - **Parry** at the perfect instant → cancels this die, "banks" it as a bonus block for future phases.
     - **Attack window** (the 1.5s after the parry) — the player unleashes all their Attack Dice at once, with the player's 6s chipping a "wound mark" on Dracular.
     - **Dodge (RT-NATIVE)** — if the player holds parry *early*, they spend 2 stamina to negate the die entirely but forfeit the attack window.
3. **Dice Pool Visual.** All nine of Dracular's dice are arranged in a ring around him, slowly spinning. Each banked parry removes one die from the ring. This gives *visible, linear progress* through the fight: you can *see* Dracular running out of dice.
4. **The Hunger Phase (phase 4).** Dracular's fourth phase is a **Hunger** attack — he lunges to drain HP. Parrying it doesn't just block, it *heals the player by 2*. This turns the boss fight into a dramatic mid-fight recovery opportunity and fixes the "one-shot cliff" from combat chain 5.
5. **The Final Phase (phase 9).** When all nine dice are gone, Dracular enters **Unbound Rage** — a 6-second window where the player must land *one* final unblocked hit. His DD are all 9 dice at once, constantly re-rolling. The player's job: time their Attack to a moment when Dracular's defence ring spins down to fewer than 6 dice. A pure rhythm-reading execution test.
6. **Death animation = the victory screen.** No transition. The boss collapses into a pile of d6s; the "You Win" banner spawns from the dice pile.

**Design rationale:**
- Translates "9 AD" into nine discrete, digestible phases — *the source game's number becomes the pacing unit.*
- Keeps the dice-visual front and centre; parrying still *rolls a d6 pool* under the hood.
- Gives the player 9 × 3 = ~27 input moments during the boss fight, versus the source game's zero.
- Total fight duration: ~60 seconds on a clean run, ~90 seconds on a messy run. Exactly the right length for a 25-minute run's climax.
- Escalation: phases 1–3 are teaching, 4 is the twist, 5–8 are the test, 9 is the execution. Classic 5-beat boss structure.

**What we preserve from the source:** Every die is still rolled. The 6s still matter. The 9-AD stat is the fight's structure. A player who ignored the shop will have 1 AD and will never beat phase 9 — same punishment as the source. **The math didn't change; the presentation became interactive.**

---

## 8. The Shop Problem

### The problem

The source game shops at step 7 of every turn. That's a *pause-and-browse* moment in a game with no urgency. In RT, pausing every 5 seconds to open a shop menu is friction; conversely, a shop that streams during combat is overwhelming; conversely, removing the shop entirely guts the engine-building mechanic.

### Options considered

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| A. Pause-for-shop (full pause menu every turn) | Preserves source pacing | Kills flow, seven pauses per minute | ❌ |
| B. Safe-zone shop (dedicated shop tiles) | Gives shop a spatial identity | Changes source rule (shop is available every turn) | ⚠️ close second |
| C. Streaming shop (buy while fighting) | No flow interruption | Overwhelms the player during combat | ❌ |
| D. Remove shop entirely, auto-upgrade | Simplest | Destroys player agency in the build | ❌ |
| E. **Shop Shrine at end of turn (chosen)** | Preserves source + reduces friction | Minor spatial addition to grid | ✅ |

### Chosen: **The Shop Shrine**

**Mechanic:** After each monster combat resolves (or each empty/treasure room), a small glowing **Shop Shrine** rises from the ground on the current tile. It's always present, always at the end of the resolve phase — it's a physical manifestation of "step 7".

- **The world freezes while the Shrine is open.** This is a *local* pause — only the shop UI is interactive, but it's framed as an in-world "the hero kneels at the shrine" beat, not a menu pause.
- **The Shrine auto-opens for 2 seconds** showing the affordable items as glowing runes. If the player ignores it (walks away, presses Escape), the shrine sinks and the turn ends. If the player engages, time stops until they close the menu.
- **Items purchased are dropped onto the tile** as visible pickups that the hero walks through to equip. Tangible feedback for an abstract purchase.
- **Slot exclusivity is visualised** as grayed-out runes with a red chain between the already-owned and the blocked item. No words needed.
- **Potion packs are a separate Shrine section** (a vial rack beside the item plinth) — keeps potion buying distinct from gear buying.

**Why this works:**
- Source rule preserved: shop is available every turn.
- Flow preserved: the pause is framed as a *beat*, not a menu (Dark Souls' bonfire UX, not Minecraft's inventory UX).
- Opt-out is easy: walking away is the "close" button. No modal frustration.
- The shrine is diegetic — it's *in the world*, so the world pausing feels correct. Nothing breaks the fourth wall.

**What this costs us:** 3–6 seconds per turn. Acceptable. The source game's step 7 takes 5–15 seconds; we're net faster.

---

## 9. RT-Native Agency Additions

The RT version adds agency layers the board game physically cannot have. These are *additions*, not replacements — everything from the source is still there, but stacked on top are:

1. **Parry timing (core new mechanic).** The count-6s dice pool becomes a timing window. See §5 Roll-and-Parry. This is the single most important addition.

2. **Dodge roll (secondary defensive tool).** A stamina-gated sidestep that negates a phase but forfeits the attack window. Gives the player an "out" when they can't read a tell. Costs nothing in the source; costs 2 stamina in RT.

3. **Stamina meter.** A tiny 5-pip bar that regenerates in rest beats. Dodging, heavy attacks, and map-peeking spend stamina. Prevents spam of RT-native tools and preserves the source's "risk is meaningful" feel.

4. **Mid-combat potion use (RT-NATIVE, resolves source ambiguity F-2).** Q-hotkey drinks a potion any time. Source rule was ambiguous; the RT version canonicalises it as "yes, during combat". This alone dramatically changes the game feel — no more helpless dying on the "1-HP cliff" (combat chain 5).

5. **Positioning (mild — grid-slot-based).** The hero occupies a tile but can wiggle within it during combat, visually dodging. This is aesthetic, not mechanical — it doesn't change hit math, but it lets the player *feel* close calls. This matters for kinaesthetic engagement.

6. **Read-the-tell meta-skill.** Every monster type has a unique wind-up animation and sound. Orcs telegraph differently from Skeleton Lords. A mastery player learns the animations; a new player just watches the glowing flash. This converts the source game's flat "X Attack Dice" stat into a pattern-recognition skill.

7. **Weapon feel differentiation.** Big Sword and Big Axe no longer just differ in math (+1 AD vs +2 AD). They differ in *animation speed* — Sword swings fast, Axe swings slow with a stronger stagger. Both have the source's AD value; both feel distinct in the hand.

8. **Monster approach vector.** In the board game, the monster "is in the room". In RT, the monster walks *toward* the player from a room edge. A clever player reads the approach vector to back-pedal into a corner for better parry framing. This is new positional agency without altering combat math.

9. **Run modifiers / seed-of-the-day (long-term agency).** A daily seeded run (from S030) becomes a meta-agency layer: the player chooses which seed to play, competes against others on time/HP-remaining. Adds long-term skill expression the source never had.

10. **Juice-based feedback as kinaesthetic agency.** Screen shake, dice clacking, ink splatter on crit, the sword *cutting* a visible stroke across the tile like graph-paper ink. The source's tactile joy (drawing on paper) is translated into **visible ink trails** left by the hero's path — every movement "draws" the dungeon map as you go. This is *the most important aesthetic translation in the document* (see §10 Flag).

---

## 10. Flagged Issues (Actions That Don't Cleanly Translate)

These are the genuine translation problems where the RT version must accept a loss or invent aggressively.

### Issue 1: The ink-drawing agency (CRITICAL)

The source game's deepest non-obvious pleasure is **physically drawing the line** between tiles. The player circles a square, draws a line, writes a number. This kinaesthetic craft layer has **no RT analogue** unless we invent one.

**Mitigation:** The hero leaves a visible **ink trail** behind them as they move. Every visited tile is "circled" in hand-drawn ink the instant the hero leaves it. Treasure rooms get a little dollar-sign ink scribble; potion rooms get a bottle squiggle; empty rooms get an X. The *entire grid becomes a hand-drawn map* over the course of the run, and the player can look at it at the end and say "I drew that".

This is the RT version's closest answer to the source's tactile layer. It's not the same, but it's the right *metaphor*.

**Risk:** If implemented as a cheap overlay, it feels like a sticker. If implemented with physics-simulated ink (the line drags, bleeds, pools in corners), it becomes the game's signature visual. **Recommend investing here.**

### Issue 2: The "rest and think" agency

The source game lets you sit with a decision for as long as you want. RT games, even slow ones, apply soft pressure via ambient motion, sound, or enemy re-aggro. Some players will feel the RT version is *faster to think in* than the source, and this is a real loss.

**Mitigation:** A dedicated **Ponder** button (hold Tab). When held, the world freezes entirely — no music, no particles, no animation. Just the grid, the hero, and the player's brain. Unlimited duration. Releases when the player lets go. This is the RT version's answer to "I want to stare at the board for 3 minutes".

**Caveat:** Competitive/daily-seed runs disable Ponder. You can think all you want, the timer keeps running.

### Issue 3: Mid-combat potion ambiguity (F-2 from source)

Source game ambiguity: can you drink potions mid-combat? The RT version **must** resolve this because combat is continuous in RT. We choose **yes**, for gameplay reasons (see §9 point 4). This is a **rule deviation** that must be documented in the RevisedRules stage.

### Issue 4: Combat is now a skill test instead of a stat test

In the source, combat is purely your stats vs the monster's stats. A 1-AD player with bad luck dies; a 5-AD player with average luck wins. RT combat introduces a skill axis: a 1-AD player with *perfect parry timing* can theoretically beat a row-6 Skeleton. This is a **meaningful power-curve shift**.

**Mitigation:** Enemies in higher rows get *shorter parry windows* and *feinted tells*. A skilled low-build player can still win, but the difficulty scales fast enough that build is still dominant. We should explicitly **playtest** that an un-upgraded player cannot cheese the boss purely on parry skill — if they can, the progression system is broken.

### Issue 5: The "blocked path" loss condition

In the source, the player loses if they can't reach End (via BFS). In RT, this loss feels *terrible* — you were having fun killing orcs and suddenly the game says "you lose because of a spatial puzzle you didn't realise you were in".

**Mitigation (already in Stories.csv S012):** Every candidate movement runs BFS; moves that would block End reachability are surfaced with a **red warning icon** on the tile and require a hold-to-confirm to enter. This preserves the loss condition but eliminates the "didn't see it coming" feeling. Expert players can disable the warning in settings for a harder challenge.

### Issue 6: No stop-and-stare for the room reveal

The source game lets you *savor* the moment of rolling the d6 for a new room. In RT, if the reveal is automatic the instant you enter a tile, it becomes wallpaper. **Fix:** the reveal animation is ~500ms and includes a *physical d6 tumbling on the tile* that the player watches roll. This is dead air in the input sense, but alive in the narrative sense. **Do not shorten this animation below 400ms — it's the source game's rhythm.**

### Issue 7: No "rolling dice in your hand" tactile pleasure

Nothing in digital beats the feel of shaking dice in a cupped hand. RT version has no answer. **Accept the loss.** The best we can do is excellent dice sound design: cupped-in-hand rattle, the clack against wood, 6s ringing differently than 1s.

---

## Summary Table: Agency Decisions

| Decision | Choice | Confidence |
|---|---|---|
| Combat model | Auto-attack + timed parry ("Roll-and-Parry") | High |
| Input method | Mouse + keyboard (PC/Steam primary, gamepad native, mobile port) | High |
| Shop model | Shop Shrine at end of tile resolve (diegetic local pause) | High |
| Boss fight | Nine Dice Duel (9-phase rhythm fight) | High |
| Pacing philosophy | Preserve calm, spike agency in combat, honour rest beats | High |
| Ink drawing | Auto-drawn ink trail with physics feel — core aesthetic | High |
| Mid-combat potions | Allowed (resolves source ambiguity F-2 in favour of RT) | Medium — rule deviation |
| Blocked-path warning | BFS warning + hold-to-confirm (from S012) | High |
| Ponder mode | Unlimited hold-Tab world-freeze, disabled in daily runs | Medium |
| Skill ceiling target | Routing > Parry > Build > Execution; preserve routing as dominant | High |

---

## Open Questions for Later Stages

These are the agency questions that RT-3 (Control Scheme) and RT-4 (Feel) will need to nail:

1. **Exact parry window duration.** 200ms? 250ms? Must be playtested.
2. **Stamina regeneration rate.** Fast enough to not feel stingy, slow enough to matter.
3. **Whether dodge exists at all** or is folded into "early parry" as a single mechanic.
4. **Whether movement is grid-snap or free-form within a tile.** Leaning grid-snap for clarity; free-form for feel.
5. **Music-as-tempo-clock?** If the parry rhythm syncs to music BPM (Crypt of the NecroDancer-style), the skill floor rises but the ceiling does too. Tempting but potentially divisive.
6. **Daily seed leaderboards: global or friend-only?** Affects the competitive agency layer.
7. **Does the Shop Shrine rise after *every* tile or only after monster tiles?** If every tile, the game becomes shopping-heavy and rhythmic. If monster-only, the shop ties to combat reward. Current choice: every tile (matches source step 7), but worth playtesting.
8. **Should the Ponder mode also pause source-of-truth BFS so a player can "taste" a move before committing?** Leaning yes — it becomes the RT analogue of "hovering the pencil over the paper".

None of these are blockers for RT-3 — they're feel questions that emerge in playtest.

---

## Handoff Notes to RT-3 (Control Scheme)

The next stage inherits the following concrete agency commitments that must not be changed without revisiting this document:

- Combat is **timed-parry auto-attack**, not twin-stick action, not auto-battle.
- Primary input target is **mouse + keyboard on Steam**, gamepad second.
- The **Shop Shrine** is diegetic and locally pauses the world.
- **Dracular is a nine-phase rhythm boss**, not a dice slog.
- **Ink trails** are the kinaesthetic soul of the RT version and must be implemented with feel, not as a cheap overlay.
- **Routing skill remains dominant**; RT-native combat skill is a secondary layer.

Everything else is tuneable.

*End of AgencyModel.md — Stage RT-2 complete.*
