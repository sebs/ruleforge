# Solo Dungeon Dash — Rules

## 1. Title & Elevator Pitch

**Solo Dungeon Dash** is a hand-drawn 2D top-down action roguelite. You are a lone adventurer descending through a procedurally seeded 9×10 dungeon, fighting dice-driven monsters room-by-room, hunting Dracular in the End chamber. Every attack and every defence rolls a visible side-tray of d6 dice — count the 6s, and they decide the outcome. Combat is "Roll-and-Parry": parry a telegraphed strike to crack open an enemy's guard, then unload your Attack Dice. Miss the parry and you eat damage, or dodge-roll through it at the cost of your opening. A full run takes roughly 20 minutes. Die once and the run ends.

This is a standalone video game. You do not need dice, paper, or any physical component.

---

## 2. Objective & Win Condition

Your goal is to reach the **End room** at the top of the dungeon and kill the boss, **Dracular**, before you run out of hearts.

The game is binary:

- **Victory** — Dracular's HP is reduced to zero while you still have at least 1 heart remaining.
- **Defeat** — Any of the following:
  - Your heart total drops to 0.
  - You enter a room from which no legal path to the End room exists anymore.
  - You choose to abandon the run from the pause menu.

There is no score. There is no partial credit. You either finish the dungeon or you don't.

---

## 3. Starting Conditions

Every run begins with exactly the same loadout:

| Resource | Starting Value |
|---|---|
| Hearts (HP) | 17 / 17 |
| Attack Dice | 1 |
| Defence Dice | 1 |
| Treasure | 0 |
| Potions | 0 |
| Weapon slot | empty (basic fists) |
| Shield slot | empty |
| Armour slot | empty (cloth) |
| Position | Start room |

Hearts are **hard-capped at 17**. No effect, buff, or potion can push you above 17. Dice pools, treasure, and potions are carried only for the current run — nothing persists between runs except cosmetics and bestiary entries (see §16).

---

## 4. Controls

Solo Dungeon Dash targets **mouse + keyboard** as the primary input, **gamepad** as a first-class secondary (Steam Deck supported), and a simplified **touch** layout for mobile.

### Mouse & Keyboard

| Action | Default Key |
|---|---|
| Move | WASD |
| Light Attack | Left Mouse Button |
| Parry | Right Mouse Button (hold briefly to brace, release on the monster's flash) |
| Dodge-Roll | Space |
| Interact / Open Shop Shrine / Confirm door | E |
| Consumable (drink Potion) | Q |
| Map & Reachability Overlay | Tab (hold) |
| Ponder (world-freeze, single-player only) | Shift (hold) |
| Pause | Esc |

### Gamepad

| Action | Default Binding |
|---|---|
| Move | Left Stick |
| Light Attack | X / Square |
| Parry | Right Trigger |
| Dodge-Roll | B / Circle |
| Interact | A / Cross |
| Drink Potion | Left Bumper |
| Map Overlay | Touchpad / Select |
| Ponder | Hold Left Trigger |
| Pause | Start |

### Touch (mobile)

Virtual stick on the left, tap-to-attack / hold-to-parry on the right, swipe to dodge, thumb-tap icons for map and potions. An "Assisted Parry" option widens the parry window for touch play.

All bindings are fully remappable in Settings.

---

## 5. The Dungeon

The dungeon is a **procedurally seeded 9 columns × 10 rows grid of rooms** (90 rooms total) plus two special rooms:

- **Start** — attached below the middle column of row 1. You always begin here.
- **End** — attached above the middle column of row 10. The boss lives here.

Each room is a visually self-contained **octagonal chamber with chamfered corners**, drawn in hand-drawn ink-on-parchment style. Every room has up to **8 doorways**, one on each cardinal and diagonal edge: N, NE, E, SE, S, SW, W, NW. Adjacency is king-move: from any room you can transition to any of its 8 orthogonal or diagonal neighbours (subject to grid edges).

**Room transitions are snap-to-grid.** Inside a room you move continuously with WASD. To change rooms you walk through a doorway; when your character crosses the threshold, the camera snaps to the next room and you appear on the opposite side of it.

**Doorways seal behind you.** Once you leave a room, its doorways close permanently. You **cannot re-enter a room you have already visited**. Your path through the dungeon is a one-way trail from Start toward End.

**The camera shows one room at a time.** No minimap by default — use the Map Overlay (Tab) to see the full dungeon layout, your path, and reachability status.

---

## 6. Room Contents

When you first enter a room, the fog-of-content in the room dissolves, a single d6 physically tumbles on-screen for about half a second, and one of the following reveals:

| Content | Effect |
|---|---|
| **Empty** | Nothing happens. Walk through and leave. |
| **Treasure** | A small pile of coins / a gold chest. Auto-pickup on room clear (+1 Treasure). |
| **Potion** | A glowing vial. Auto-pickup on room clear (+1 Potion). |
| **Monster** | One enemy aggros on entry; you must kill it to clear the room. |
| **Shop Shrine** (rare) | A glowing altar rises from the floor. Spend Treasure on gear and potions. |
| **Boss** (End room only) | Dracular. See §12. |

Room content weighting **scales with depth**. Shallow rows (1–3) spawn weak monsters (Orc, Wolf) and are mostly empty or loot. Mid rows (4–7) spawn tougher threats (Skeleton, Evil Warrior, Devil Bat, Cyclops). Deep rows (8–10) can spawn Dark Elves, Skeleton Lords, Wizards, and Demons. Shop Shrines are rare but weighted slightly higher at depth transitions (end of row 3, 6, 9).

Treasure and Potion pickup happens automatically the instant the room is cleared — you do not pick items up by hand.

---

## 7. Combat — The Roll-and-Parry System

Combat is the heart of Solo Dungeon Dash. Every monster encounter is a short, tense, skill-driven duel. The math underneath is the **count-6s dice pool** — you see your dice roll in a side tray every single time.

### The Dice Tray

A **visible Dice Tray panel** sits along the right edge of the screen. It contains two dice pools:

- **Attack Dice Tray** (top) — one d6 per Attack Die you own (starts at 1; max 5).
- **Defence Dice Tray** (bottom) — one d6 per Defence Die you own (starts at 1; max 7 or higher with stacked gear).

The tray animates on every attack and every defence. Dice tumble in 3D for about half a second and settle face-up. Every **6** that shows is a **hit** (on attack) or a **block** (on defence). Non-6 faces do nothing.

### Monster State

Every non-boss monster has:

- **1 HP** — a single successful unblocked hit kills it.
- **A Guard state** — by default the monster is guarded and your attacks glance off it. You cannot brute-force damage a guarded enemy.
- **A Windowed Opening** — a short (~0.4s) state where its guard drops and your next attack can land.

### The Encounter Loop

1. **Aggro.** The monster spawns in the room and immediately begins its attack animation. You cannot swing first — your character is still drawing their weapon during the opening 0.4 seconds. This enforces the "monster attacks first" rule diegetically.
2. **Telegraph.** The monster winds up a visible, animated attack. The wind-up lasts 0.3–1.2 seconds depending on monster type. Every monster has a distinct tell.
3. **Your reaction** — you must pick ONE of three responses to each telegraphed strike:
   - **Parry** — briefly hold the Parry button and release just before the monster's strike connects. If your timing hits the ~200ms parry window, the **Defence Dice Tray rolls**, every 6 cancels a pip of damage (usually reducing it to zero), the monster is **staggered** for about 0.5 seconds, and its Guard drops into a **Windowed Opening**. A perfect parry always negates the incoming hit AND creates the opening.
   - **Dodge-Roll** — tap Dodge to sidestep with i-frames. This avoids the hit entirely without rolling dice, but it does **not** create an opening. The fight continues and the monster resets its guard.
   - **Tank** — do nothing. The **Defence Dice Tray rolls automatically**. Each 6 cancels one pip of incoming damage. Any pips that leak through become lost hearts.
4. **Opening strike.** During a Windowed Opening, attack. The **Attack Dice Tray rolls**. Every 6 is a kill-strike (monsters have 1 HP, so any 6 kills). Extra 6s spill over into **cleave damage** on adjacent enemies if multiple monsters share the room. If the tray rolls **zero 6s**, your blade clangs off (guard spark), you do no damage, and the encounter resets to step 2.
5. **Repeat** until the monster dies or your hearts hit 0.

### Key combat rules

- Hits on a **guarded** enemy produce a guard spark with no damage, regardless of dice. You cannot damage a monster outside of a Windowed Opening.
- The **parry window is ~200ms on Normal difficulty** (adjustable per difficulty mode, see §15).
- Dodge-roll i-frames last about 300ms and have a short cooldown (stamina-gated).
- Potions can be drunk at **any time** during combat, including between telegraphs (see §8).
- Each monster has a signature wind-up animation and audio cue so you can learn its timing. Higher-tier monsters have **shorter telegraphs** and may feint.
- Damage dealt to you is capped per-hit by monster tier: low-AD monsters remove 1 heart per unblocked hit; high-AD monsters (7+) remove 2 hearts; the Demon removes 2 with a bleed DoT; Dracular's charged specials can remove 3.

### Monster roster

| Monster | AD | Tell | Notes |
|---|---|---|---|
| Orc | 1 | Club overhead raise, ~0.8s | Tutorial enemy |
| Wolf | 2 | Crouches before pouncing | Teaches dodge over parry |
| Skeleton | 3 | Rattles before triple swipe | Reassembles from falls |
| Evil Warrior | 4 | Lowers stance before thrust | Parries your swings |
| Devil Bat | 5 | Screeches then dives | Only vulnerable on dive |
| Cyclops | 6 | Both-hand shoulder turn, ~1.2s | Huge sweeps, long openings |
| Dark Elf | 7 | Purple glow then volley | Ranged, teleports |
| Skeleton Lord | 8 | Raises staff (summon) / halberd sweep | Spawns 2 skeleton adds |
| Wizard | 9 | Three colour-coded spell tells | Pure ranged |
| Demon | 10 | Inhale / claw / lunge (three tells) | 2 pips + bleed |
| Dracular | 9/9 | See §12 | Boss |

---

## 8. Potions

Potions are your only healing resource. You can carry any number (the HUD stacks them vertically on the right side, capped at 99).

- **Drinking** — press Q (or LB) to drink one potion. Each potion restores **+1 heart**, instantly. Hearts cannot exceed 17; drinking at full health is **blocked** and flashes a "HP Full" prompt.
- **Drinking animation** — 0.5 seconds. You can move while drinking but cannot attack or parry during the animation. Time your heals for the safe beat between monster telegraphs.
- **Found potions** (picked up from room content) are **usable immediately**.
- **Bought potions** (purchased at a Shop Shrine) are **locked for the next room** — the HUD shows them greyed out with a small hourglass icon. When you enter the next room, they unlock and become usable. This preserves the shopkeeping beat: you must plan one room ahead when buying healing.

Potions are the difference between a 1-heart cliff death and a recovered run. Drip-feed them. Do not hoard.

---

## 9. Treasure

Treasure is the currency you spend at Shop Shrines on gear and potions.

- **Gain** — auto-picked up on room clear. Treasure rooms reveal a chest or coin pile worth **+1 Treasure** each. Killing a monster may also leave a small Treasure drop (+1) as part of loot spill.
- **Carry** — unbounded. There is no treasure cap.
- **Spend** — only at a **Shop Shrine** (§10). You cannot spend Treasure anywhere else.
- **Visual** — Treasure count is shown as a coin icon in the HUD bottom-right, next to your Potion stack.

---

## 10. Shop Shrines

When you enter a room that contains a **Shop Shrine**, a glowing stone altar with a vial rack and weapon plinth rises from the floor. Walk up to it and press **Interact** (E). The world enters a **local pause** — enemies in unrelated rooms halt, ambient music lowers, and the shrine menu opens. This is framed as your hero kneeling at the altar, not as a system menu.

The Shop Shrine offers the following items:

### Gear (slot-exclusive)

| Cost | Item | Effect | Slot |
|---|---|---|---|
| 1 | **Buckler** | +1 Defence Die | Shield slot — replaces current shield |
| 2 | **Shield** | +2 Defence Dice | Shield slot — replaces current shield |
| 3 | **Big Sword** | +1 Attack Die, fast swing | Weapon slot — replaces current weapon |
| 4 | **Big Axe** | +2 Attack Dice, slow swing with heavy stagger | Weapon slot — replaces current weapon |
| 5 | **Spiky Armour** | +2 Attack Dice AND +1 Defence Die | Armour slot — replaces current armour |
| 6 | **Magical Armour** | +5 Defence Dice | Armour slot — replaces current armour |

### Potions

| Cost | Pack | Contents |
|---|---|---|
| 1 | 1-pack | 1 Potion (locked for next room) |
| 2 | 3-pack | 3 Potions (locked for next room) |
| 3 | 6-pack | 6 Potions (locked for next room) |

### Slot exclusivity rules

Your character has exactly **three equipment slots**: Weapon, Shield, Armour. Each slot holds at most one item.

- Buying a new weapon **replaces** your current weapon. The old weapon is refunded at **half cost (rounded down)** and added back to your Treasure. Example: you own a Big Sword (3 Treasure). You buy a Big Axe (4 Treasure). You spend 4, get 1 back, net cost 3.
- Buying a new shield replaces your current shield with half-refund.
- Buying new armour replaces your current armour with half-refund.
- Weapons and armour are **not** mutually exclusive with shields — you may wear armour AND wield a weapon AND hold a shield at the same time. Only same-slot items conflict.
- Potions are not slot-based; buy as many packs as you can afford.

### Shrine UX

- Items you cannot afford are greyed out.
- Items in a slot you already own are shown with a small "REPLACE" tag and the net refund cost.
- Slot-conflict is visualised by a red chain linking the currently-equipped item to the blocked item.
- Walking away from the shrine (or pressing Esc) closes the menu with no purchase. The shrine sinks back into the floor and the room is marked cleared.

Equipped gear changes your character's on-screen appearance immediately — the weapon morphs in-hand, the shield appears on the off-arm, the armour snaps on with a small particle burst.

---

## 11. Reachability & Blocking

You may not re-enter a visited room. This means a careless route can isolate you from the End room. **Getting blocked is a loss condition.** To prevent accidental losses, Solo Dungeon Dash runs a continuous reachability check:

- A background algorithm checks, after every move, whether a legal path still exists from your current room to the End room through unvisited rooms.
- When you stand on a threshold and attempt to step into a room whose occupation would **strand you** (no future path to End), a **red warning icon** appears over that doorway and the room is highlighted in red on the Map Overlay.
- Entering a warned room requires a **hold-to-confirm** commit: you must push into the doorway for about 1 full second before the move commits. Releasing early cancels. This is your commit window.
- A small persistent **reachability icon** in the HUD top-right is green when End is still reachable and amber when only one path remains. If you still block yourself despite the warning, the icon turns red and the "Dungeon Sealed" loss screen triggers.
- Expert players may disable the warning icon from Settings for a harder, unassisted challenge.

Transitions through **non-warned** doorways commit instantly on threshold cross — no hold required.

---

## 12. Boss Fight — The Nine Dice Duel (Dracular)

The End room contains **Dracular**, the final boss. The fight is a multi-phase rhythm duel lasting roughly **90 seconds on a clean run**.

Dracular is visible at the centre of a circular chamber. Around him orbits a **ring of 9 glowing d6 dice** — these are his nine Attack Dice, made manifest. Each phase of the fight corresponds to one die.

### Phase structure

- **Phases 1–3 (teach)** — Dracular winds up one Loaded Die at a time, hurls a telegraphed strike, and presents a parry opportunity of increasing difficulty. Parry to cancel the die and bank it out of his ring. A banked die cannot attack you again.
- **Phase 4 (Hunger)** — Dracular lunges to drain your hearts. A successful parry here not only negates the hit but **heals you by +2 hearts** (still capped at 17). Missing Hunger is the biggest damage swing of the fight.
- **Phases 5–8 (test)** — Dracular combines telegraphs, feints, and shorter parry windows. Dodge-rolling is a valid emergency escape but forfeits the counter-attack window.
- **Phase 9 (Unbound Rage)** — All dice are gone from his ring. Dracular enters a 6-second rage window. His 9 Defence Dice are all re-rolling continuously in a spinning ring around him. Watch the ring: when the Defence ring momentarily spins down below 6 live dice, **attack** — your Attack Dice Tray rolls, and any surviving 6 lands the killing blow. You must land at least one clean hit during Unbound Rage or the fight resets to phase 5.

### Boss combat rules

- Your Attack Dice Tray and Defence Dice Tray roll exactly as in normal combat.
- Parrying during the Nine Dice Duel still rolls the Defence Tray. A perfect parry additionally banks one of Dracular's dice, permanently reducing the size of his Attack Dice ring.
- Dodge-rolling is unlimited by stamina during the boss fight (stamina is paused).
- Potions are usable during Dracular but are **locked during the wind-up frames of his telegraphs** — drink them between phases, not mid-strike.
- If you die during Dracular, the run ends.

Full boss details live in the RTGDD.

---

## 13. Victory & Defeat Screens

### Victory

On killing Dracular, time freezes and the boss collapses into a cascading pile of glowing d6s. Ink-splash "YOU WIN" banners spawn from the dice pile. The screen fades to the run summary:

- **Outcome:** VICTORY
- **Clear time:** mm:ss
- **Rooms explored:** N / 90
- **Hearts remaining:** N / 17
- **Treasure collected:** N
- **Monsters defeated:** N
- **Build:** weapon / shield / armour icons
- **Bestiary progress:** new entries unlocked
- **Cosmetic unlocks:** any new unlocks earned

Buttons: **Next Run**, **Main Menu**, **Share Seed**.

### Defeat

On heart-drop to 0, the screen desaturates, the ink bleeds outward from the player, and "YOU DIED" scrawls across in hand-drawn script. The summary screen shows the same stats plus a **cause of death** line ("killed by Skeleton Lord, row 8") and **furthest row reached**.

On reachability loss ("Dungeon Sealed"), a different screen shows the Map Overlay with your path drawn and the dead-end highlighted. Cause: "You sealed yourself from the End room."

Buttons on both: **Retry**, **New Seed**, **Main Menu**.

---

## 14. Run Length

A typical run takes **about 20 minutes**. Fast runs with aggressive routing finish in 12 minutes. Slow, exploratory runs cap at about 35 minutes. The run clock is always visible in the HUD if you enable it in Settings; it is required on in Daily Seed runs.

---

## 15. Difficulty Modes

Four tunable difficulties are offered. You may change difficulty between runs but not mid-run.

| Mode | Parry Window | Enemy Damage | Starting Potions | Notes |
|---|---|---|---|---|
| **Easy** | 350 ms | Reduced by 1 per hit (min 1) | +2 at start | Reachability warnings always on. Assisted Parry available. |
| **Normal** | 200 ms | Baseline | 0 | Default experience. |
| **Hard** | 150 ms | +1 per hit | 0 | Feinted tells. Shorter Shop Shrine timers. |
| **Nightmare** | 120 ms | +1 per hit, bleed on 7+AD | 0 | No reachability warnings. Potions bought cost +1. |

Daily Seed runs always play on a fixed difficulty (Normal) to keep the leaderboard fair.

---

## 16. Metaprogression

Solo Dungeon Dash is a **no-power-creep** roguelite. Nothing you earn between runs makes your next run mechanically easier. You begin every run with exactly 17 hearts, 1 Attack Die, 1 Defence Die, 0 Treasure, 0 Potions, and no gear.

What does persist:

- **Bestiary** — each monster type has a codex entry. The first time you defeat a monster, its entry unlocks. Entries contain lore, tell descriptions, and kill statistics. Complete the bestiary by beating Dracular at least once with all 11 entries unlocked.
- **Cosmetic unlocks** — hero skins, ink colours, dice skins, shrine decoration themes, custom victory banners. Earned through milestone achievements (first Cyclops parried, first no-hit room, first sub-15-minute clear, etc.). Purely visual; no stat impact.
- **Seed history** — your last 50 run seeds with outcome and clear time, for re-play or sharing.
- **Daily Seed leaderboard** — each calendar day generates a fixed global seed; your best clear on that seed is recorded. Friend leaderboards optional.

There is no XP tree. There is no permanent stat buff. Every run is a clean slate, and the only thing that improves is **you**.

---

*End of Solo Dungeon Dash rules.*
