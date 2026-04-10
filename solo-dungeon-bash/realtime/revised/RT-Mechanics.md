# RT Mechanics Identification & Categorization — Solo Dungeon Dash

> RealTimeForge Stage RT-D2: Revised Mechanics
> Source: `output/solo-dungeon-bash/Mechanics.md`
> Genre lock: Action Roguelite / Dungeon Crawler with Dice-Driven Combat (2D top-down, octagonal rooms, hand-drawn ink-on-parchment, Roll-and-Parry combat, 17 hearts, diegetic Shop Shrines, ~20 min median runs, solo only)

This document translates the 10 source mechanics into their real-time equivalents and documents the RT-native mechanics invented to make the genre lock cohere. Structure parallels the source Mechanics.md.

---

## 1. Mechanics Overview Table

| # | Mechanic | Category | Role |
|---|---|---|---|
| RM1 | Dice Tray (visible count-6s pool) | Dice Visualization / Combat Feedback | **Primary** |
| RM2 | Cell-Snapped Room Traversal | Grid-Based Movement / Exploration | **Primary** |
| RM3 | Hearts + Gold + Potion HUD | Resource Management | **Primary** |
| RM4 | Pre-Seeded Room Contents | Procedural Content Seeding | Secondary |
| RM5 | Shop Shrine Loadout | Diegetic Menu / Tableau Building | Secondary |
| RM6 | Routing Push-Your-Luck | Spatial Risk/Reward | Secondary |
| RM7 | Potion Slot Management | Consumable Inventory | Secondary |
| RM8 | Permanent Gear Upgrades | Engine Building (stat compounding) | Secondary |
| RM9 | Roll-and-Parry Combat | Parry-Riposte Action / Guard-Break | **Primary** |
| RM10 | Reachability Warning System | Constraint / Soft Puzzle Layer | Tertiary |
| RM11 | Parry Timing Window | RT-NATIVE Combat Input | **Primary** |
| RM12 | Dodge-Roll with I-Frames | RT-NATIVE Defensive Movement | Secondary |
| RM13 | Guard / Opening Cycle | RT-NATIVE Enemy State Machine | **Primary** |
| RM14 | Shop Shrine Proximity Interaction | RT-NATIVE Diegetic UI | Secondary |
| RM15 | Telegraph / Tell System | RT-NATIVE Readability | **Primary** |
| RM16 | Hit-Stop Feedback | RT-NATIVE Game Feel | Secondary |
| RM17 | Doorway Seals | RT-NATIVE Navigation Constraint | Secondary |
| RM18 | Room Transitions / Camera Lock | RT-NATIVE Framing | Tertiary |
| RM19 | Ink Trail / Map Drawing | RT-NATIVE Kinaesthetic Layer | Tertiary |
| RM20 | Boss-Entry Commit Window | RT-NATIVE Commitment Gate | Tertiary |

---

## 2. Detailed Per-Mechanic Breakdown

### RM1 — Dice Tray (Primary)
- **Category:** Dice Visualization / Combat Feedback
- **How it works:** A vertical side-panel on the right edge of the screen shows the player's current Attack Dice (AD) and Defence Dice (DD) as physical d6 models. On every player swing connection, the AD pool tumbles in the tray for ~400ms and the count of 6s is read out as a cleave multiplier. On every incoming hit the DD pool tumbles and 6s cancel hits. The tray never hides — it is always glanceable during combat. The count-successes-on-d6 math from the source is preserved exactly.
- **Key parameters:**
  - Tray position: right side-panel, ~12% of screen width.
  - Dice tumble duration: 400ms (never shorter — this is the source rhythm).
  - Success threshold: 6 (1/6 per die, unchanged from source).
  - Starting pools: 1 AD / 1 DD.
  - Max pools: 5 AD / 7 DD (per EconomyModel's A-3 stacking ruling).
  - Dice physics: 3D rigid-body tumble with audible clack per die.
- **Lineage:** Direct descendant of **M1 Dice Rolling**. The count-6s mechanic is preserved verbatim; only the input is removed (dice are rolled automatically on every combat event instead of by hand).

### RM2 — Cell-Snapped Room Traversal (Primary)
- **Category:** Grid-Based Movement / Exploration
- **How it works:** The dungeon is rendered as a 9×10 grid of octagonal rooms. The hero moves with WASD/left-stick inside a room freely, but **room-to-room transitions** are cell-snapped: the player walks toward a doorway and the camera hard-cuts to the next octagonal room. The adjacency is king-move (8-way including diagonals) — each octagonal room has up to 8 visible doorways. The player chooses the next cell by walking toward the corresponding doorway.
- **Key parameters:**
  - Grid dimensions: 9 × 10 = 90 rooms + Start + End = 92 cells.
  - Neighbourhood: 8-way king (cardinals + diagonals).
  - Room size: ~10m × 10m playable, octagonal footprint.
  - Intra-room move speed: ~4 m/s base (modifiable by gear).
  - Transition time: ~600ms camera cut + fade.
  - Revisit: forbidden (enforced by Doorway Seals, RM17).
- **Lineage:** Descendant of **M2 Roll-and-Write Grid Exploration**. The 9×10 grid and king-move adjacency are preserved literally. What changes: no drawing with a pencil, no self-resolving turns — the movement is continuous real-time locomotion within and between cells.

### RM3 — Hearts + Gold + Potion HUD (Primary)
- **Category:** Resource Management
- **How it works:** Three interlocking resources are surfaced as distinct HUD elements:
  - **Hearts:** 17 discrete heart pips along the top-left HUD. Each incoming unblocked hit removes 1 pip. Hard cap at 17.
  - **Gold (Treasure):** numeric counter with coin icon, top-center. Auto-picked on proximity from kills and destroyed chests. Unbounded.
  - **Potions:** vertical stack of up to 6 vial slots, top-right. Quick-heal hotkey drinks one for +1 heart.
- **Key parameters:**
  - Starting hearts: 17. Cap: 17.
  - Potion inventory cap: **6 vials** (new RT cap, prevents hoarding exploit).
  - Potion→heart ratio: 1:1.
  - Item costs: 1–6 gold (preserved from source).
  - No passive regen. No overheal.
- **Lineage:** Direct descendant of **M3 Resource Management**. All three source resources survive with the same caps and ratios. The potion cap is tightened from "unbounded" to 6 to prevent panic-chug trivialization in real-time combat.

### RM4 — Pre-Seeded Room Contents (Secondary)
- **Category:** Procedural Content Seeding
- **How it works:** At run start, the game rolls the d6 encounter table for every room in the 9×10 grid and commits the result to a seed. Room contents are thus deterministic-once-committed but still feel stochastic to the player, because content is **fog-of-war hidden** until the hero crosses the doorway. The source's per-row level tables are preserved exactly, including the probability of monsters, treasure, potions, and empty rooms by level.
- **Key parameters:**
  - Seed generated at run start; daily runs use a shared seed.
  - Monster probability by level: L1 60%, L2 33%, L3–L10 50–67% (preserved from source table).
  - Treasure probability: ~17% flat per row.
  - Potion probability: 17% on rows 1, 3, 6, 9 only; 0% elsewhere.
  - Empty probability: 0–50% (row-dependent).
  - Content reveal animation: ~500ms physical d6 tumble on the tile (never shortened below 400ms — it is the source's rhythm).
- **Lineage:** Descendant of **M4 Random Encounter Table Lookup**. The level tables are unchanged. The only shift is *when* the dice are rolled: at run-start instead of at tile-entry. This gives the game reproducible seeds for speedrunning and daily challenges while preserving the surprise-on-reveal moment.

### RM5 — Shop Shrine Loadout (Secondary)
- **Category:** Diegetic Menu / Tableau Building
- **How it works:** After the resolve phase of every room (monster dead, empty room walked through, or treasure collected), a small glowing stone **Shop Shrine** rises from the floor of the current octagonal room. The player can approach the shrine to buy any of the 9 source shop items with accumulated gold. Slot exclusivity is preserved (Weapon / Shield / Armour). World time freezes locally when the shrine menu is open (diegetic "kneeling at the shrine" beat), unfreezes when the player walks away or presses Escape.
- **Key parameters:**
  - Shrine auto-rise delay: ~1 second after resolve.
  - Shrine auto-open hover: 2 seconds of glowing runes before requiring interaction.
  - Menu open time: ~400ms radial animation.
  - Three exclusive slots: Weapon (Sword/Axe), Shield (Buckler/Shield), Armour (Spiky/Magical).
  - Item costs: 1–6 gold.
  - Slot swap refund: 50% (rounded down), resolves source ambiguity F-3.
- **Lineage:** Descendant of **M5 Item Shop / Loadout Tableau**. Source's "end-of-turn shop always available" rule is preserved literally. The shop is present every turn, with all 9 items always buyable. Changes: the shop is now a diegetic in-world object instead of an out-of-game menu, slot exclusivity is visualized with greyed runes + red chains, and a 50% refund on slot swaps is added.

### RM6 — Routing Push-Your-Luck (Secondary)
- **Category:** Spatial Risk/Reward
- **How it works:** The player's meta-decision is which row of the dungeon to push into next. Deeper rows have stronger monsters and richer rewards. The player "presses" by walking toward an upward doorway and "stops" by electing to farm a lower row. There is no explicit press/stop button — the choice is continuous and made with the feet. A second push-your-luck layer appears at the End tile (RM20) where a hold-to-confirm window gates the boss fight.
- **Key parameters:**
  - Monster AD by row: L1 = 1 AD, L2 = 2 AD, … L10 = 10 AD (source table preserved).
  - Reward density increases roughly linearly with row.
  - No explicit tempo clock.
  - Daily run mode adds a visible wall clock for competitive pressure.
- **Lineage:** Descendant of **M6 Push-Your-Luck**. The decision shape is identical — when to push deeper vs when to farm shallower. What changes: the push-your-luck now has a kinaesthetic layer (walking vs turning around) and a commit gate at the End tile.

### RM7 — Potion Slot Management (Secondary)
- **Category:** Consumable Inventory
- **How it works:** Potions occupy up to 6 inventory slots visible as a vertical vial stack in the HUD. Potions found in rolled rooms become instantly usable the next frame. Potions *bought* at a Shop Shrine are greyed out for 20 seconds of wall-clock time, a real-time translation of the source's "next turn" delay. Quick-heal hotkey (Q on keyboard, LB on controller) drinks one potion for +1 heart at any time, including mid-combat (resolves source ambiguity F-2 in favour of mid-combat use).
- **Key parameters:**
  - Inventory cap: 6 slots.
  - Buy delay: 20 seconds of grey-out with visible countdown ring.
  - Found potions: instant-usable.
  - Drink animation: 500ms (player is mobile but cannot attack).
  - HP cap blocks potion use at 17/17 (shows "HP full" prompt, potion not consumed).
  - Potions cannot be drunk during the first 0.3s of a boss telegraph wind-up (anti-panic-chug rule during boss phases).
- **Lineage:** Descendant of **M7 Hand Management (Potions)**. The timing constraint (bought potions delayed, found potions instant) is preserved as a wall-clock grey-out. The source ambiguity F-2 is resolved: yes, usable mid-combat. New: hard cap of 6 to prevent hoarding, and an anti-panic-chug rule during boss telegraphs.

### RM8 — Permanent Gear Upgrades (Secondary)
- **Category:** Engine Building (stat compounding)
- **How it works:** Every Shop Shrine purchase permanently modifies the hero's AD or DD pool. More AD means more dice tumble in the tray on every swing → more cleaves → faster kills. More DD means more dice cancel hits → more surviving each encounter. Early gold is worth more than late gold because upgrades compound across all future combats. There is no experience or levelling system — progression is purely via gear.
- **Key parameters:**
  - Max raw AD: 5 (1 base + 2 Big Axe + 2 Spiky Armour).
  - Max raw DD: 7 (1 base + 1 Spiky + 5 Magical, or 1 + 2 Shield + 5 Magical per A-3 ruling).
  - No XP, no stat points, no levelling.
  - No item rarity tiers — the 9 shop items are the entire progression space.
- **Lineage:** Descendant of **M8 Engine Building**. Identical stat compounding curve. The ladder of 9 items is untouched.

### RM9 — Roll-and-Parry Combat (Primary)
- **Category:** Parry-Riposte Action / Guard-Break
- **How it works:** When the hero enters a monster-occupied room, the monster is already positioned and begins its attack windup before the hero's weapon draws. The player cannot swing first — this enforces "monster always attacks first" in frame data. Combat resolves in three phases per beat:
  1. **Monster windup & telegraph.** Clear visual+audio tell (0.3–1.2s depending on monster).
  2. **Player reaction.** One of: Parry (tight window, see RM11), Dodge-roll (i-frames, RM12), or do nothing (take the hit).
  3. **Opening window.** A successful parry drops the monster's Guard (RM13) for ~400ms. The player's swing during that window rolls the Dice Tray (RM1); any 6 = a killing hit. Extra 6s cleave to adjacent enemies. If the swing lands outside the opening, the dice roll is converted into a Guard Spark (clang, no damage).
- **Key parameters:**
  - Monster initiative lockout: 0.4s at encounter start (player cannot attack).
  - Monster telegraph length: 0.3–1.2s depending on type.
  - Opening window after successful parry: ~400ms.
  - Player swing wind-up: ~200ms (Sword) / ~350ms (Axe).
  - Player swing recovery: ~300ms (Sword) / ~500ms (Axe).
  - All monsters: 1 HP (unchanged from source).
  - Dracular: 1 HP + 9 DD + 9-phase structure (see ConflictModel §7).
- **Lineage:** Descendant of **M9 Hit-Point Combat Subsystem**. The 1-HP rule, "monster first" initiative, and count-6s dice math are preserved verbatim. What changes: the dice roll is gated by a parry-driven Opening instead of alternating rounds, and the Guard/Opening cycle replaces the turn-based attack/defence steps.

### RM10 — Reachability Warning System (Tertiary)
- **Category:** Constraint / Soft Puzzle Layer
- **How it works:** A silent BFS (breadth-first search) runs every frame on the uncleared graph, asking "can every reachable tile still reach End?" When the player hovers over a doorway whose entry would orphan End, the doorway turns red and requires a 400ms hold-to-confirm to enter. This preserves the source's self-blocking loss condition without making the player feel ambushed by a spatial puzzle they didn't realize they were in. Expert players can toggle the warning off for a harder challenge.
- **Key parameters:**
  - BFS frequency: every frame (90×10 grid is trivial to recompute).
  - Warning trigger: doorway choice that disconnects End from any uncleared tile.
  - Hold-to-confirm duration: 400ms.
  - Toggleable off in Settings → Accessibility → "Classic Routing".
- **Lineage:** Descendant of **M10 Self-Blocking Pathfinding Constraint**. The constraint is preserved literally; only the feedback layer is new (the warning). The Hamiltonian-trap loss state still exists for players who disable the warning.

---

## 3. Mechanic Interaction Summary

- **RM2 (Traversal) feeds RM4 (Pre-Seeded Reveal):** Walking through a doorway resolves the pre-seeded content for that cell.
- **RM4 feeds RM3 (HUD) and RM9 (Combat):** Rolled content either drops into resources or spawns a monster.
- **RM3 (Gold) feeds RM5 (Shop Shrine):** Gold is spent at the shrine.
- **RM5 feeds RM8 (Engine):** Purchases change the dice pool.
- **RM8 feeds RM1 (Dice Tray):** More AD/DD means more dice tumble per event.
- **RM1 feeds RM9 (Combat):** The dice tray's 6-count determines hit/cleave outcomes.
- **RM9 feeds RM3 (Hearts):** Unblocked hits subtract heart pips; monster kills drop gold.
- **RM6 (Routing PYL) sits across RM2 and RM8:** The meta-choice of when to push deeper.
- **RM10 (Reachability) constrains RM2:** Some doorways become conditional (hold-to-confirm) based on BFS state.
- **RM11 (Parry Window) is the input gate for RM9:** Without a successful parry, the Opening never appears.
- **RM12 (Dodge) is the fallback for RM11:** Sacrifice the Opening to preserve hearts.
- **RM13 (Guard/Opening) gates RM1:** The dice tray rolls always, but only Opening-window swings convert 6s into damage.
- **RM14 (Shrine Proximity) gates RM5:** The shop menu only opens when the hero stands in the shrine's trigger ring.
- **RM15 (Telegraphs) gates RM11:** Parry timing keys off the telegraph's flash frame.
- **RM16 (Hit-Stop) is pure feedback on RM9:** No mechanical change, massive feel change.
- **RM17 (Doorway Seals) enforces RM2's no-revisit:** Doors close behind the hero.
- **RM18 (Camera Lock) frames RM2 and RM9:** One room at a time, hard cut between rooms.
- **RM19 (Ink Trail) is pure aesthetic feedback on RM2:** No mechanical change.
- **RM20 (Boss Commit) gates entry to RM9's boss encounter.**

---

## 4. New RT-Native Mechanics (Invented for RT)

### RM11 — Parry Timing Window
- **Category:** RT-NATIVE Combat Input
- **How it works:** The player holds (or taps) the Parry button during a monster's telegraph. If the button is active within a narrow window centered on the frame where the monster's attack would connect, the parry succeeds: the Dice Tray rolls DD, any 6s cancel incoming hits, the monster's Guard drops (RM13), and the player gets an Opening window to kill. A missed parry (too early or too late) means the hit lands at full damage and no Opening is granted. This is the single most important RT-native addition — it converts the source's zero-input combat into a timing skill.
- **Key parameters:**
  - Base parry window: 200–250ms (playtest-tunable).
  - Higher-row monsters shrink the window to ~150ms by L10.
  - Parry cooldown: none (spammable), but failed parries still eat the incoming hit.
  - Parry button: Right Mouse / RT on gamepad.
  - Visual tell: monster's weapon flashes white on the frame the window opens.
- **Why invented:** The source's "0 player input during combat" is a non-starter in RT. Parry is the minimum possible input that preserves the count-6s dice identity.

### RM12 — Dodge-Roll with I-Frames
- **Category:** RT-NATIVE Defensive Movement
- **How it works:** A stamina-gated sidestep. Pressing Space/A causes the hero to roll ~3 metres in the current movement direction, granting ~300ms of invincibility frames. The dodge roll sacrifices the Opening window — the monster is not staggered and the player gets no guaranteed-kill moment. It is the fallback when the player cannot read the telegraph or misses the parry window.
- **Key parameters:**
  - I-frame duration: 300ms.
  - Dodge distance: ~3 metres.
  - Stamina cost: 2 pips (of a 5-pip stamina bar).
  - Stamina regen: ~1 pip/sec during rest beats.
  - Cooldown: 400ms between rolls (prevents spam).
- **Why invented:** Provides an "out" for moments when the player cannot commit to a parry. Without it, new players would be brutalized on row 6+.

### RM13 — Guard / Opening Cycle
- **Category:** RT-NATIVE Enemy State Machine
- **How it works:** Every monster cycles between three states: **Guard** (hits do nothing, dice tray rolls are converted to Guard Sparks), **Windup** (telegraphing attack, vulnerable to parry), and **Opening** (triggered by a successful parry or the monster's own recovery frame — hits convert 6s into damage). The player's job is not to deal damage — it is to manufacture Openings. The source's 1-HP rule becomes meaningful because the hit that actually connects is very hard to set up.
- **Key parameters:**
  - Default state: Guard (permanent until windup).
  - Windup duration: 0.3–1.2s by monster type.
  - Opening duration after successful parry: ~400ms.
  - Opening duration after unparried recovery: ~250ms (shorter, riskier).
  - Guard Spark effect: clang SFX, small particle, tiny enemy stagger.
- **Why invented:** Solves "the 1-HP problem" from ConflictModel §2. Makes connecting the hit the hard part, preserving source lethality without making fights 0.4-second bursts.

### RM14 — Shop Shrine Proximity Interaction
- **Category:** RT-NATIVE Diegetic UI
- **How it works:** The shrine is a physical object in the world with a ~2m trigger radius. Walking into the radius causes the shrine to glow and opens an ambient hint. Pressing E (or approaching the altar while pressing the interact button) opens the radial shop menu and locally freezes world time. Walking away closes the menu and resumes world time. The shrine is not a modal dialog — it is a diegetic kneel beat.
- **Key parameters:**
  - Trigger radius: ~2m.
  - Menu open animation: 400ms radial.
  - World freeze scope: local (only the shop UI is interactive; particles still drift slightly).
  - Auto-close: walking outside the trigger radius.
- **Why invented:** Replaces "step 7 shop phase" with an in-world object. Solves the Shop Problem from AgencyModel §8.

### RM15 — Telegraph / Tell System
- **Category:** RT-NATIVE Readability
- **How it works:** Every monster has a unique, rehearsable windup animation and audio cue that signals the incoming attack. Orcs raise a club overhead. Wolves crouch before a pounce. Skeletons rattle before a triple-swipe. Wizards glow with a colour-coded spell aura. The player learns these tells over runs, converting "unknown threat" into "known rhythm". The parry timing window (RM11) keys off the tell's flash frame.
- **Key parameters:**
  - Tell duration: 0.3–1.2s (by monster AD tier — stronger monsters get longer tells to balance higher damage).
  - Per-monster audio cue: unique growl/screech/chant on room entry.
  - Visual flash: on the exact parry-window frame, the weapon or limb flashes white.
  - Feinted tells (L7+ monsters): rare fake-out windups that do nothing, punishing greedy parriers.
- **Why invented:** The source's "X AD" stat is flat and opaque. In RT each monster needs a learnable identity. Tells convert the flat stat into a pattern-recognition skill.

### RM16 — Hit-Stop Feedback
- **Category:** RT-NATIVE Game Feel
- **How it works:** On every killing hit, the game pauses all animation and motion for ~80ms. During hit-stop, the screen lightly shakes, the dice tray flashes its 6s, and a splash of hand-drawn ink erupts from the enemy. This is pure juice with no mechanical effect, but it is load-bearing for the "hits feel weighty" promise. Parry successes also trigger a shorter hit-stop (~50ms) with a white-flash on the monster.
- **Key parameters:**
  - Kill hit-stop: 80ms freeze.
  - Parry hit-stop: 50ms freeze.
  - Boss phase transition hit-stop: 200ms (longer for drama).
  - Screen shake amplitude: ~3 pixels.
- **Why invented:** In RT, feel is mechanical. Without hit-stop, the 1-HP-enemy combat would feel flat. This is the single most important feel investment.

### RM17 — Doorway Seals
- **Category:** RT-NATIVE Navigation Constraint
- **How it works:** Every doorway the player exits through is physically sealed behind them — a stone slab drops, or an ink-drawn X covers the door. The source's "no-revisit" rule is enforced through geometry rather than a bookkeeping rule. The player never has to remember which rooms they've visited because the sealed doors are visible on the minimap overlay.
- **Key parameters:**
  - Seal animation: 400ms drop.
  - Visual: stone slab in the dungeon, matching ink-X on the minimap.
  - Un-sealable: no. Sealed doors cannot be opened even with consumables.
- **Why invented:** Replaces the source's "don't re-enter drawn squares" bookkeeping rule with a diegetic constraint. The player cannot accidentally violate the rule.

### RM18 — Room Transitions / Camera Lock
- **Category:** RT-NATIVE Framing
- **How it works:** The camera locks to the current octagonal room and follows the hero only within it. When the hero crosses a doorway, the camera hard-cuts to the next room and the new room's contents reveal. This reinforces "one room at a time" pacing and prevents the player from seeing adjacent room contents. It also draws a sharp spatial boundary for combat — monsters cannot chase across rooms.
- **Key parameters:**
  - Camera cut duration: ~100ms black fade + 500ms reveal animation.
  - Camera mode: fixed to room center with slight player-tracking lerp.
  - No zoom — fixed framing for readability.
- **Why invented:** The source is literally one cell at a time (you draw a square, resolve it, move on). The camera lock is the RT analogue.

### RM19 — Ink Trail / Map Drawing (Kinaesthetic Aesthetic)
- **Category:** RT-NATIVE Kinaesthetic Layer
- **How it works:** As the hero moves, they leave a physics-simulated ink line behind them on the floor. Every visited tile is marked with a hand-drawn ink symbol when the hero leaves it: treasure rooms get a dollar-sign scribble, potion rooms get a bottle squiggle, monster rooms get an X on the defeated corpse, empty rooms get a tidy circle. Holding Tab pulls up a map overlay showing the entire 9×10 grid with all the hero's drawn ink visible. This is the RT version's closest answer to the source's pencil-on-paper tactile agency.
- **Key parameters:**
  - Ink line physics: light drag, slight bleed at corners, pooling on stops.
  - Map overlay: hold Tab, world freezes (or slows in daily runs).
  - Ink symbol set: ~6 glyphs for the 6 room-content types.
  - Ink colour: sepia on parchment.
- **Why invented:** The source's deepest non-obvious pleasure is physically drawing the dungeon. Without some analogue, the RT version loses its kinaesthetic soul (AgencyModel §10, Issue 1).

### RM20 — Boss-Entry Commit Window
- **Category:** RT-NATIVE Commitment Gate
- **How it works:** The End tile (Dracular's chamber doorway) is a visually special sealed door. Walking into it triggers a 2-second "hold-to-confirm" — the hero leans against the door, pushing it open gradually. Releasing cancels; completing the hold commits to the boss fight. No accidental boss starts.
- **Key parameters:**
  - Hold duration: 2000ms.
  - Cancel on release: yes, anywhere before full hold.
  - Visual: hero animation of pushing against a heavy door.
- **Why invented:** Source has no "commit" moment — you just walk into the End square. In RT, accidentally starting a 90-second boss fight is unacceptable UX.

---

## 5. Dissolved Mechanics (Present in Source, Removed in RT)

The following source mechanics no longer exist as player-facing mechanics in Solo Dungeon Dash. Each is dissolved with a stated reason.

### Literal Dice Rolling as a Player Action
- **Source:** Player physically rolls d6 in hand.
- **Why dissolved:** Real-time combat cannot stop to wait for a physical dice roll. The dice are still visible (RM1 Dice Tray) and the count-6s math is preserved, but the roll is automatic and happens on every combat event. This is necessary for the genre lock.
- **What survives:** The visible tumble, the count-6s reading, the bursty binomial distribution.

### Pen-and-Paper Tracking
- **Source:** Player draws the dungeon on graph paper, writes HP, crosses out visited squares.
- **Why dissolved:** Digital doesn't need bookkeeping. All state is tracked by the game engine.
- **What survives:** The ink-trail aesthetic (RM19) and the hand-drawn visual style. The *feeling* of drawing is preserved; the *task* is automated.

### The 7-Step Turn Sequence (as a Hard Structure)
- **Source:** Each turn is literally a 7-step procedure: move, roll, resolve, combat, potion, shop, end.
- **Why dissolved:** Real-time play doesn't have turns. The steps become continuous phases that can overlap: movement happens in real-time, content reveals trigger on entry, combat begins when monsters aggro, potions are hotkeyed any time, shopping happens at a diegetic shrine. The 7-step structure is implicit, not enforced.
- **What survives:** Every step's *function* still happens in order per room (reveal → resolve → optional shop → exit), but the player never explicitly transitions between steps.

### Alternating-Round Combat
- **Source:** Monster attacks, player defends, player attacks, monster defends, repeat.
- **Why dissolved:** Real-time combat cannot alternate cleanly. Combat is continuous: the monster telegraphs and swings, the player parries or dodges, the player's counter-swing rolls the dice tray during the Opening window.
- **What survives:** "Monster always attacks first" — enforced by frame data (player cannot swing during the 0.4s weapon-draw at encounter start). And both sides still roll dice for every swing and block.

### Forbidden-Revisit as a Bookkeeping Rule
- **Source:** Player must not re-enter a drawn square.
- **Why dissolved:** Bookkeeping rules don't work in RT because the player can't be expected to remember a rule mid-action.
- **What survives:** Doorway Seals (RM17) enforce no-revisit through physical geometry — the doors close behind you.

### Step-6 "End of Turn" Potion Phase
- **Source:** Potions can only be drunk during step 6 of a turn.
- **Why dissolved:** No turns means no step 6. A menu-based potion UI mid-combat would be unthinkable.
- **What survives:** The hotkey-quick-heal (Q / LB) — usable any time. This resolves source ambiguity F-2 in favour of mid-combat use. Bought potions still have a delay (20s wall-clock instead of "next turn").

### Step-7 "Shop Phase" Menu
- **Source:** At step 7 the player opens a shop sheet and spends treasure.
- **Why dissolved:** An out-of-game menu breaks RT flow and feels bureaucratic.
- **What survives:** The Shop Shrine (RM5 + RM14) — a diegetic in-world object that presents the same 9 items with the same exclusivity rules. The menu is locally paused, not globally paused.

### Two-Roll Combat Sub-Steps (Attack Roll + Separate Defence Roll)
- **Source:** Attacker rolls AD, then defender rolls DD, then hit count is reconciled.
- **Why dissolved:** Two sequential rolls are too slow for real-time. Both sides roll simultaneously in the dice tray.
- **What survives:** The AD and DD pools are visually distinct (player-side tray vs enemy-side tray), and both still use count-6s. The math is identical; only the visual sequencing collapses.

### Deterministic Table Lookup on Room Entry
- **Source:** The d6 is rolled *when the player enters the room*.
- **Why dissolved:** Rolling on entry prevents reproducible seeds and daily challenges, and creates a moment of dead air.
- **What survives:** The d6 rolls happen at run-start (pre-seeded, RM4). The reveal on entry still plays the tumbling-d6 animation, so the player *feels* the roll even though it already happened.

---

## 6. Notable Absences (Deliberately NOT Added)

To keep faith with the source, the following standard RT genre mechanics were deliberately excluded. Each absence is a design commitment.

### No Stat Trees
The source has exactly 9 shop items. No XP, no levelling, no skill trees, no perk menus. We refuse the temptation to add a stat tree because it would break the "1 run, 1 build, 9 items" simplicity that makes the engine-building decision meaningful.

### No Loot Cascades
No rarity tiers (white/blue/purple/orange), no random item drops, no legendary weapons. The 9-item shop is the entire loot space. Every item is always available; only gold limits you.

### No Levelling
The hero does not gain XP, does not level up, does not unlock abilities. The only progression is gear purchases. Combat skill and routing mastery grow in the player's head, not in a stat sheet.

### No Crit Hits
No critical hit chance, no crit damage multipliers. The dice tray's count-6s is the only source of variance. Adding crits would pollute the bursty-binomial feel with a second variance layer.

### No Status Effects
No burn, no poison, no freeze, no bleed, no stun (other than the brief opening stun from a parry, which is a hit-stop not a status). No debuffs from monsters (other than the Demon's special-case 3-pip grab). The source has zero status effects; we keep it that way.

### No Co-op
Solo only. No lobby, no friend invite, no drop-in player 2. The source is a solo game; co-op would triple the scope and dilute the meditative loop.

### No PvP
No asynchronous leaderboards with attack phases, no "invade another player's run" mode. Only local comparison against seed leaderboards (time, hearts remaining) in daily-seed mode, which is informational not interactive.

### No Metaprogression Stat Buffs
No "keep 10% of your gold after death" upgrades, no "+1 starting HP per boss defeated". Metaprogression is strictly **cosmetic and bestiary unlocks only** — you unlock monster-entry art and hero skins by finding them, nothing that affects run balance.

### No Fog-of-War in Combat Rooms
Combat rooms are fully visible the moment you enter. The source has no fog within a resolved room; neither do we.

### No Pet / Companion / Summon
The hero is alone. No pet, no familiar, no summoned ally, no hireable mercenary. Preserves the "solo" in Solo Dungeon Dash.

### No Item Crafting
No recipes, no forging, no reforging, no upgrade gems. You buy items; that's the entire economy.

### No Map-Wide Minimap
The minimap is hold-Tab only, and it only shows visited tiles + the current tile's immediate neighbours. The full layout is never revealed until you've walked it. This preserves the source's "see one cell at a time" discipline.

### No Flee / Retreat Combat Option
Once combat starts, it runs to a corpse — yours or the monster's. No "run away" button. Preserves the source's no-flee rule. (Exception: dodge-roll is *inside* a combat, not out of it.)

### No Ability Cooldowns Beyond Parry/Dodge
The hero has exactly three verbs in combat: move, parry, dodge. No fireball, no dash-strike, no ultimate. The simplicity is the point.

### No Build-Defining Relics
No Hades-style boon system, no Slay-the-Spire relic drops that fundamentally change the run. The 9-item shop is the entire build space. Runs vary by seed and by skill, not by a layer of magical modifiers.

---

*End of RT-Mechanics.md — Stage RT-D2 complete.*
