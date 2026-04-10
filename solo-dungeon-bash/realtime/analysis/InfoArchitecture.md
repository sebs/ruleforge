# Information Architecture Translation — Solo Dungeon Bash → Real-Time

> Stage RT-3 of RealTimeForge. Source: paper-and-dice solo roll-and-write. Target: real-time action dungeon crawler (2D/3D). This document defines what the player sees, where, when, and in what format.

---

## 0. Executive Summary

Solo Dungeon Bash is an **open-information, random-content** game. The player is literally holding the rulebook while they play. Every probability is public, every stat is listed, every state tracker is visible. The only things the player does not know are (a) what is in an unentered room and (b) what the next dice roll will show.

In real-time terms, that translates to a single dominant design constraint: **the HUD must be minimal and trust-based, the dungeon must use a soft fog reveal on entry, and the player should retain their "dungeon master with the rulebook" relationship via an always-accessible Codex.** Do not gate information behind meta-progression — that betrays the source's pedagogical honesty.

The chosen fog-of-war model is **No Fog (layout visible) + Content Fog (contents hidden until entry)**. Justification below in Section 4.

---

## 1. Source Information Model

A complete inventory of every data point in the board game, classified by visibility.

### 1.1 Always-Public Information (the player sees this from turn 0)

| Information | Where it lives on the sheet | Notes |
|---|---|---|
| Dungeon grid dimensions (9×10) | Printed sheet | Topology is known in advance |
| Start square location | Printed sheet | Fixed below row 1, mid column |
| End square location | Printed sheet | Fixed above row 10, mid column |
| Current Health | Tracker box | 0–17 |
| Current Attack Dice | Tracker box | 1–5 |
| Current Defence Dice | Tracker box | 1–8 |
| Current Treasure count | Tracker box | 0–n |
| Current Potion count | Tracker box | 0–n |
| Owned items | Sheet margin / list | Buckler, Shield, Big Sword, Big Axe, Spiky Armour, Magical Armour |
| Level tables (all 10) | Rulebook page | d6 → content mapping for every level |
| Monster roster & stats | Level tables | 11 monster types, AD listed |
| Dracular's stats | Boss table | 9 AD, 9 DD |
| Item shop catalogue | Rulebook page | 8 items, prices, effects, exclusions |
| Combat rules (initiative, d6=6 hits) | Rulebook page | Deterministic procedure |
| Movement rules (king-adjacent, no-revisit) | Rulebook page | Deterministic procedure |
| Visited squares | Grid (inked circles) | Path history permanent |
| Current position | Grid (most recent circle) | Self-evident |
| Own path trace (lines) | Grid | Full history retained |
| Level of current row | Grid layout | Trivially derived from row number |
| Probability of each outcome per level | Derivable from tables | Player can math it out |
| Reachability of End from current cell | Derivable via inspection | Hard cognitive work but free information |

### 1.2 Revealed-on-Entry (hidden until triggered)

| Information | Trigger | Duration |
|---|---|---|
| Contents of a specific unvisited cell | Entering it (step 1→2) | Persists for the rest of the run |
| Result of the room-content d6 roll | The roll itself | Recorded on sheet |
| Result of each monster attack roll | Combat round | Instantaneous |
| Result of each player defence roll | Combat round | Instantaneous |
| Result of each player attack roll | Combat round | Instantaneous |
| Result of each monster defence roll (Dracular only) | Combat round | Instantaneous |

### 1.3 Never Known (true uncertainty)

| Information | Why unknowable |
|---|---|
| Future d6 outcomes | RNG; no decks, no seeds the player can see |
| Which monsters will appear in future rooms | Depends on future rolls |
| Exact total damage the player will take over the run | Emergent from a chain of rolls |
| Whether a specific path will succeed | Depends on rolls yet to come |

### 1.4 Summary Table: Every Data Point × Visibility

| # | Data | Fully Public | Revealed On Entry | Never Known |
|---|---|:---:|:---:|:---:|
| 1 | Dungeon dimensions | X | | |
| 2 | Start / End positions | X | | |
| 3 | Player HP / AD / DD / Treasure / Potions | X | | |
| 4 | Owned items | X | | |
| 5 | Level tables (probabilities) | X | | |
| 6 | Monster stats (all 11) | X | | |
| 7 | Dracular stats | X | | |
| 8 | Shop catalogue | X | | |
| 9 | Combat rules | X | | |
| 10 | Movement rules | X | | |
| 11 | Visited cells | X | | |
| 12 | Current position | X | | |
| 13 | Path history | X | | |
| 14 | Unvisited room contents | | X | |
| 15 | Room-content roll result | | X | |
| 16 | Combat roll results | | X | |
| 17 | Future dice outcomes | | | X |
| 18 | Future room contents | | | X |

---

## 2. Translation to Real-Time

For each source-game information category, the real-time equivalent. Three tags are used: **DIRECT** (1:1 translation), **ADAPTED** (source concept mapped to a new UI idiom), **RT-NATIVE** (no source analogue, added because real-time demands it).

### 2.1 Always-Public → HUD + Codex

| Source | RT Equivalent | Tag |
|---|---|---|
| HP tracker box | HP bar + numeric readout top-left | DIRECT |
| AD tracker box | Attack icon + numeric "AD: 3" in HUD | DIRECT |
| DD tracker box | Shield icon + numeric "DD: 2" in HUD | DIRECT |
| Treasure count | Coin icon + numeric top-right | DIRECT |
| Potion count | Potion icon + numeric + hotkey [Q] | DIRECT |
| Owned items | Paper-doll slots in inventory panel (tab) | DIRECT |
| Level tables | Codex → Bestiary → "Level N table" page | ADAPTED |
| Monster stats | Codex + enemy inspect on alt-hover | ADAPTED |
| Dracular stats | Codex → Boss entry, also a pre-arena cinematic intro | ADAPTED |
| Shop catalogue | In-game shop UI with full catalogue visible, greyed if unaffordable | DIRECT |
| Combat rules | Tooltip on each stat, full rules in Codex | ADAPTED |
| Movement rules | Implicit (controls) + tutorial prompts | ADAPTED |

### 2.2 Grid & Path → Minimap + World

| Source | RT Equivalent | Tag |
|---|---|---|
| Grid dimensions | World map visible at all times via minimap | DIRECT |
| Start / End markers | Green "Entry" portal / red "Boss" door icon on minimap | DIRECT |
| Visited cells | Minimap cells illuminated; floor shows a subtle walked-path trail | DIRECT |
| Current position | Player arrow on minimap + obvious 3rd-person camera | DIRECT |
| Path history | Persistent trail on minimap + breadcrumb particles in world (optional) | ADAPTED |
| Reachability of End | RT-NATIVE: "Path to Boss" toggle on minimap that BFS-traces a legal route hint | RT-NATIVE |

### 2.3 Revealed-on-Entry → Fog & Reveal

| Source | RT Equivalent | Tag |
|---|---|---|
| Unvisited room contents | Room is visible in silhouette on minimap, contents hidden; on entry a "room reveal" flourish plays | ADAPTED |
| Room-content d6 roll | Becomes a 0.3s "content spawn" moment — dust puff, enemy fade-in, pickup bounce-in | ADAPTED |
| Combat dice rolls | Become actual swings, parries, blocks in real-time; dice are simulated internally | ADAPTED |

### 2.4 Never Known → Stays Never Known

Future randomness remains unknowable. However, in RT we should show the **current roll seed** on a dev overlay (optional) and preserve the player's right to *not* know the future. Do not add precognition.

---

## 3. Information Visibility Map — Where on Screen

Grouped by UI zone. This is the master contract for what lives where.

### 3.1 HUD (persistent, always visible, glanceable)

Purpose: **sub-1-second reads during combat.** Nothing here should require thought.

- **HP bar + numeric** (red, top-left) — current/max, visible damage flashes
- **AD badge** (sword icon + number, near HP) — current attack dice count
- **DD badge** (shield icon + number, near HP) — current defence dice count
- **Treasure counter** (coin icon + number, top-right)
- **Potion counter** (flask icon + number + hotkey label, bottom-center or bottom-right)
- **Current dungeon level** (e.g. "Level 4", top-center) — tiny, gold-on-dark
- **Minimap** (top-right corner, square, ~180×180 px) — see 3.2
- **Crosshair / lock-on reticle** (center)
- **Damage numbers** (world-space, above targets, fade 1.2s)
- **Active status icons** (under HP bar) — "Just-Purchased Potions (unusable 1 turn)" etc.

### 3.2 Minimap (persistent, secondary, glanceable)

Purpose: **1–3 second reads between combats.**

- **Full 9×10 grid layout** — ALWAYS visible (no fog on topology)
- **Entry icon** (green portal) at Start
- **Boss icon** (red skull) at End
- **Visited cells** — bright, with a small glyph indicating resolved content (loot/empty/fight-icon)
- **Unvisited cells** — dim grey, no content icons
- **Current cell** — pulsing player marker
- **Path trail** — faint line connecting visited cells in order of visit
- **Legal next-step highlight** — subtle shimmer on the 8 king-adjacent unvisited cells
- **"Path-to-End" toggle** — on press (M), BFS a legal route to End and overlay it

### 3.3 Room Reveal (contextual, transient)

Purpose: **appears on entry, gone in ~2 seconds.**

- **Room banner** (top-center) — e.g. "A Wolf lairs here." / "Treasure: 1 chest" / "Empty chamber" (2s fade)
- **Content spawn VFX** — dust, light, particle burst as enemies/pickups appear
- **Enemy nameplate** — appears on first aggro, fades after combat or out-of-LOS

### 3.4 Inspection Menus (on-demand, readable)

Purpose: **5–30 second reads, game slows or pauses.**

- **Enemy Inspect** (hold Tab with enemy targeted) — stats, lore, tactics, icon
- **Item Inspect** (hover in shop or inventory) — effect, cost, exclusions, comparison
- **Shop UI** (opens at safe rooms / post-combat) — full catalogue, greyed if unaffordable or slot-locked
- **Inventory Panel** (I key) — paper-doll with 3 slots (weapon / shield / armour), potion stack

### 3.5 Codex / Logbook (opened at will, pause-worthy)

Purpose: **the "rulebook" — preserves source trust.** Pause-on-open is acceptable.

- **Bestiary** — all 11 monsters, stats, lore, which levels they appear on, current encounter count
- **Level Tables** — the actual d6 tables, unchanged, with percentages annotated ("16.7% Cyclops")
- **Item Catalogue** — full shop list, acquired status, exclusivity rules
- **Combat Rules** — "6s hit, defence cancels hits, monster attacks first"
- **Run Log** — chronological list of every room entered, every roll, every purchase, every HP change (see Section 10)

### 3.6 Post-Run Dashboard

See Section 10.

---

## 4. Fog of War Design — DECISION

### The choice

**No layout fog + Content fog.** The entire 9×10 dungeon topology is visible from turn 1 (Start and End marked, all cells shown as empty frames). Cell **contents** are hidden until the player physically enters the cell, at which point the content reveals with a short flourish.

### The four options and why the others lose

1. **Full fog** — cells entirely invisible until entered.
   - Reject: the source game prints the whole grid on the sheet. The player knows exactly where End is and can count steps. Full fog betrays that.

2. **Partial fog (walls visible, contents hidden)** — our pick. Below.

3. **Proximity reveal (cells light up when near, contents hidden)** — halfway option.
   - Reject: halfway house. The source gives you the whole grid instantly; proximity reveal introduces a friction (wandering to see what cells exist) that does not exist in source. Also weakens pre-planning, which is a core strategic pleasure of SDB.

4. **No fog at all (contents visible too)** — betrays the source's "roll to reveal" core.
   - Reject: would remove the only moment of tension in exploration.

### The choice, restated precisely

- **Topology:** always visible. 9 columns × 10 rows, Start below row 1, End above row 10, all cell frames drawn on the minimap and floor from turn 1.
- **Contents:** hidden until entry. Minimap shows unvisited cells as dim grey frames with no glyph. Visited cells show a small glyph for what they were (enemy-slain, treasure-taken, potion-taken, empty).
- **World view:** the player can physically walk in a large room and see adjacent unvisited rooms through doorways, but cannot see what is *inside* them. Adjacent doorways are dark until you cross the threshold.
- **Reveal moment:** crossing the threshold triggers a 0.3s content-spawn (dust puff + enemy/pickup fade-in) plus the room banner in 3.3.

### Justification

The source game is a **strategic pathfinding puzzle** where the player sees the whole maze and plans a route. The randomness is local (per-cell) not global (per-layout). A digital fog would hide something the paper original never hid, and it would break the player's ability to plan self-avoiding routes that reach End — which is half the game. Contents, on the other hand, *are* hidden in the source (you roll them on entry), so content fog is a direct translation.

---

## 5. Randomness Visibility — DECISION

### The question

The source player is literally reading the rulebook. They know Level 6 has a 16.7% Cyclops roll. Should RT preserve this, or gate it behind a bestiary/meta unlock?

### The choice

**Full preservation. Level tables are visible in the Codex from the first launch.** No meta-gating.

### Justification

1. **Trust the player.** The source game trusts the player as a DM. Meta-gating would be a betrayal of the source's ethos and a violation of the project's stated translation principles.
2. **No competitive context.** This is solo. There is no advantage to hiding odds from one player while another has them.
3. **Gameplay is still tense.** Knowing 16.7% Cyclops does not tell you *when* you will roll a Cyclops. The uncertainty is about specific draws, not the distribution.
4. **Strategic depth increases when odds are public.** The "Farming Spiral" and "Glass Cannon Rush" chains in the InteractionModel require the player to math their expected damage. Hiding tables makes this impossible and collapses the strategic space.
5. **Meta-unlock bestiaries are a roguelite cliché**, not a translation of SDB's design. Do not import foreign design patterns.

### Implementation notes

- The Codex page for each level shows the d6 table exactly as printed, with percentages annotated.
- On hover, each monster entry shows AD, expected hits/round, and a difficulty color (green/yellow/orange/red relative to current player stats).
- "Encountered in this run" counters add flavor without gating.
- The *only* thing that is not pre-shown is Dracular's full attack pattern telegraphs (see Section 9), because those are RT-NATIVE content that did not exist on paper.

---

## 6. Read-Ability vs Reaction-Ability — Information Hierarchy

Real-time combat forces a new axis that paper does not have: **how long do you have to read this?**

### 6.1 The three tiers

| Tier | Max read time | Format | What belongs here |
|---|---|---|---|
| **Glanceable** | ~1 second | Bars, icons, colors, numbers | Must be safe to consult mid-combat without taking damage |
| **Readable** | ~5 seconds | Tooltips on hover, short labels | Consulted between combats or during a safe lull |
| **Study-able** | ~30+ seconds | Codex pages, menus, long text | Consulted out-of-combat or at paused safe zones |

### 6.2 Placement by tier

**Glanceable (HUD):**
- HP bar (color + length)
- AD / DD badges (icon + single digit)
- Treasure and Potion counts (icon + single digit)
- Minimap (glanceable for "where is End, where am I")
- Damage numbers (floating, 1.2s)
- Enemy HP ring (around enemy, color-coded)
- Status icons (buffs/debuffs, under HP bar)

**Readable (tooltips, pop-ups, brief panels):**
- Room banner on entry (2s)
- Shop item tooltip (appears on hover)
- Enemy nameplate + current AD (on first aggro)
- "You levelled!" / "Item acquired" toasts (4s)
- Hotkey reminders (first run only)

**Study-able (Codex, pause menus, post-run dashboard):**
- Level tables (full d6 charts)
- Bestiary entries (lore + stats + encounter notes)
- Full item catalogue
- Combat rules
- Run log
- Post-run dashboard (Section 10)
- Accessibility settings

### 6.3 Rule of thumb

If the player is going to die while reading it, it must be a **bar** or **icon** — not text. If the player needs to consult it to make a good mid-combat decision, it must be a **glyph on an existing UI element** — not a menu.

---

## 7. HUD Wireframe — ASCII

Target resolution: 1920×1080 base. All positions relative. Solo action game conventions.

```
+---------------------------------------------------------------+
| [HP]========.....   LEVEL 4                  [MINIMAP       ] |
| [||||||||||      ]                           [ . . . . X . . ]|
| HP 12/17  AD:3  DD:2                         [ . . o-o . . . ]|
|                                               [ . o . . . . . ]|
|                                               [ . S . . . . . ]|
|                                               [_______________]|
|                                                                |
|                                                                |
|                                                                |
|                     [ 3D WORLD VIEW ]                          |
|                                                                |
|                          (player)                              |
|                                                                |
|                                                                |
|                                                                |
|                                                                |
|                                                                |
|                       [Enemy nameplate                         |
|                        Wolf   AD:2   HP:1 ]                    |
|                                                                |
|                                                                |
|                                                                |
| [STATUS: Spiky Armour]                        [COMBAT LOG    ] |
|                                                [+1 Treasure   ]|
| [Q: Potions 4]  [E: Inspect]  [I: Inv]         [Hit Wolf x1   ]|
| [COINS: 7]                                     [Wolf hit you 1]|
+---------------------------------------------------------------+
```

### 7.1 Element placement spec

- **Top-left (HP cluster):** HP bar (280×20 px), HP numeric "12/17", AD badge, DD badge, stacked vertically with 8px gap. Always visible.
- **Top-center:** current dungeon level ("LEVEL 4"), tiny gold text, 16px.
- **Top-right:** minimap, 220×220 px, always visible, click-to-ping.
- **Mid-screen center:** 3D/2D world view, clean, no UI clutter.
- **Above enemies:** nameplate (name + AD + HP dot), only on aggro.
- **Bottom-left:** status effects row (active buffs/debuffs), hotkey strip (Q, E, I), treasure coin count.
- **Bottom-right:** combat log, last 4 lines, auto-fade after 6s.
- **Dead center on damage:** floating damage numbers (world-space, not screen-space).
- **Screen edge red-vignette:** when HP ≤ 25%, pulsing red vignette.
- **No clutter bottom-center:** reserved for room banners (temporary) and boss telegraphs (Section 9).

### 7.2 Safe zones

The **center 60%** of the screen is reserved for gameplay. No persistent UI. The minimap and HP cluster hug the corners. Combat log never scrolls into the action area.

### 7.3 What is NOT on the HUD

Kept out deliberately to protect minimalism:
- XP bars (no XP in source)
- Quest trackers (no quests in source)
- Chat (solo)
- Party frames (solo)
- Ability cooldowns beyond core combat (keep it tight)
- Minimap legend (learned once, not persistent)

---

## 8. Combat Information Flow

The paper game's combat is: monster rolls, you roll defence, you roll attack, monster rolls defence. Four rolls per round. Real-time translates each to an animated beat with information layers.

### 8.1 Combat begin (t = 0)

- **Nameplate** appears above enemy (name, AD, HP-dot). Linger: until combat end or LOS break.
- **Aggro sting** (audio cue, 0.4s).
- **Camera lock-on** (optional, toggle).
- **"Threat:" indicator** in HUD if enemy AD > player DD + 1 (yellow outline) or AD ≥ player HP / 2 (red outline).

### 8.2 Enemy attack wind-up (t = 0.0 → 0.8s)

- **Telegraph** (see Section 9) begins. Visual (red cone/arc/circle) + audio growl.
- **Warning icon** on enemy nameplate (exclamation mark).
- **Player has 0.8s to react** (dodge, block, interrupt).

### 8.3 Enemy attack strike (t = 0.8 → 1.0s)

- **Strike animation plays.**
- **Damage numbers float up** from player (red, world-space, 1.2s fade).
- **HP bar drains** with a 0.2s ease-out tween; ghost-bar trails in dark red for 0.5s so the player sees "what just happened".
- **Hit audio** (meaty impact).
- **Screen shake** (small, 0.1s, scaled by damage).
- **Red vignette** flash if damage > 2.

### 8.4 Player attack (t = 1.0 → 1.5s or on input)

- **Player swing animation** with weapon trail.
- **Hit confirmation:** x-cross pulse at impact point (0.15s).
- **Damage numbers float up** from enemy (white for normal, yellow for crit — crits are RT-native flavor).
- **Enemy HP ring** drains; if 0, kill flourish + loot burst.

### 8.5 Sustained combat readouts

Persistent during combat:
- **Enemy HP ring** — orbit around enemy, color-coded (green → yellow → red).
- **Player HP bar** — HUD.
- **Cooldown strips** — if RT-native abilities are added (e.g., Potion use = 1.0s cooldown), show a small CD ring on the hotkey icon.
- **Combat log** — last 4 lines in bottom-right, 6s auto-fade.

### 8.6 Linger durations (contract)

| Element | Appears at | Fades at |
|---|---|---|
| Nameplate | Aggro | LOS break + 2s, or combat end |
| Telegraph | Attack wind-up | Strike |
| Damage number | Strike | +1.2s |
| HP bar drain | Strike | +0.2s |
| Ghost-bar trail | Strike | +0.5s |
| Screen shake | Strike | +0.1s |
| Red vignette | HP ≤ 25% | HP recovers above 25% |
| Combat log line | Event | +6s |
| Enemy death burst | Enemy dies | +0.8s |

---

## 9. Telegraphing — RT-NATIVE

The source game has **zero** telegraphs. All attacks are instantaneous dice rolls, hit or miss. Translating to RT requires inventing a reactive system, because a real-time game without telegraphs is either trivially hard or trivially easy.

Mark the entire section: **[RT-NATIVE]**

### 9.1 Telegraph vocabulary

Three channels, always combined:

1. **Visual** — ground decal or overlay showing where the attack will land.
2. **Audio** — wind-up cue scaled by attack severity.
3. **Animation** — enemy pose/body language pre-swing.

### 9.2 Per-monster telegraphs

| Monster | AD | Telegraph | Duration | Avoidance |
|---|---|---|---|---|
| Orc | 1 | Small red cone in front of orc; grunt | 0.4s | Side-step |
| Wolf | 2 | Wolf crouches, red dash-line to player; growl | 0.5s | Step perpendicular |
| Skeleton | 3 | Red arc swing; bone clack | 0.6s | Back-step |
| Evil Warrior | 4 | Overhead red vertical line; grunt | 0.7s | Side-step |
| Devil Bat | 5 | Swoop preview (red curve through air); screech | 0.7s | Duck |
| Cyclops | 6 | Big red circle (AOE slam); bellow | 1.0s | Exit circle |
| Dark Elf | 7 | Red laser dot; sharp whistle | 0.5s | Strafe |
| Skeleton Lord | 8 | Multi-cone (three arcs); chant | 0.8s | Pillar-kite |
| Wizard | 9 | Red orb glow + bolt line; incantation | 0.9s | Strafe perpendicular |
| Demon | 10 | Screen-wide red flash + two red circles; roar | 1.1s | Position between circles |
| Dracular | 9 | Multi-phase: fire cone → bat swarm → dark pulse; themed audio | Phase-dependent 0.7–1.4s | Kite arena |

### 9.3 How telegraphs map to source dice

Source rule: "Monster rolls N d6, each 6 = 1 hit."
RT rule: **The number of dice rolled internally matches the source.** The dice are rolled behind the scenes the instant the strike lands. **But the telegraph is deterministic** — the attack always plays out its animation. Damage is then based on the internal roll.

This preserves the source's probability math (same expected damage per hit) while making the attack *dodgeable* — a successful dodge cancels the internal roll and the hit does not count. A dodged wind-up = 0 dice rolled = 0 hits. This is the primary RT-native skill layer.

### 9.4 Telegraph accessibility

- Colorblind: all telegraphs also use a high-contrast outline pattern (diagonal stripes) and shape differences.
- Audio: telegraphs are redundantly cued by directional audio.
- Low-motion: telegraph durations can be extended by +50% or +100% in settings.
- Subtitles: "Cyclops winds up slam attack!" captions accompany each telegraph.

---

## 10. Information Dumping on Death / Run End — Post-Run Dashboard

The board game gives you **nothing** on run end. You close the sheet or ball it up. This is a huge RT-native opportunity because the entire run is already digital.

### 10.1 Dashboard structure

A full-screen "Run Report" that appears on Game Over (death or win):

**Header**
- Outcome (WIN / DEATH BY X / BLOCKED FROM END)
- Total time
- Final level reached
- Rooms entered / total

**Summary stats**
- Total damage taken
- Total damage dealt
- Potions drunk
- Treasure earned / spent
- Monsters killed by type (bar chart)
- Rooms by type (pie: empty / treasure / potion / monster)

**Path trace**
- Replayable minimap showing the full path frame-by-frame (scrubbable slider)
- Each room shows its resolved content
- Hover any room for "on turn N you entered and rolled X"

**Roll history**
- Expandable list: every d6 roll (internal) with context ("Turn 14, Level 5, Monster attack roll: [6, 3, 2, 1, 4] = 1 hit")
- Filters: content rolls vs. combat rolls
- Lucky/unlucky roll highlights (rolls ≥2σ from expected)

**Decision review**
- Each shop purchase decision with an analysis ("You bought Big Axe on turn 12; alternatives at that Treasure level were Big Sword or Shield")
- Each movement branch with the reachability state

**Closing card**
- A seed-share button (for replaying the same RNG)
- "Try again" / "Return to menu"

### 10.2 Why this is the right RT-native add

- Honors the paper game's tradition of holding the whole rulebook: now the player holds the *whole run*.
- Supports learning without modifying core rules.
- Gives meaning to deaths, which are frequent in the source game.
- Aligns with the "trust the player" ethos of Section 5.

---

## 11. Accessibility Considerations

The source game is paper. Accessibility was whatever the player brought to the table. RT must do better and can.

### 11.1 Visual

- **Colorblind modes:** protanopia / deuteranopia / tritanopia palettes. Red telegraphs become high-contrast diagonal-stripe overlays. HP bar uses shape + color.
- **High-contrast HUD mode:** bars thicken, text outlines bolden.
- **Text scaling:** 100% / 125% / 150% / 200% for all readable text (tooltips, codex).
- **Minimap scale toggle.**
- **Damage number size toggle.**
- **Screen shake intensity slider** (0–100%).
- **Red vignette toggle** for photosensitive players.

### 11.2 Motor

- **Remappable controls** (all actions).
- **Hold-to-toggle** option for aim/block.
- **Auto-attack toggle** — once in combat, normal attacks fire automatically.
- **Telegraph extension** — +50% / +100% on wind-up durations.
- **Pause in combat** allowed (solo game, no competitive integrity concerns).

### 11.3 Cognitive

- **Slow-mode** — global time dilation to 75% / 50%.
- **Tooltip-on-pause** — hovering anything while paused shows full detail.
- **Persistent tutorial prompts** — never hide them after first run.
- **Hint overlay** — shows "path-to-End" and "next legal cells" always (see 3.2).
- **Dungeon map mode** — pauses the game and lets the player study the grid.
- **No time-limited decisions** except during combat telegraphs (which can be extended).

### 11.4 Hearing

- **Full subtitles** for all audio cues — telegraph warnings, ambient, shop greetings.
- **Visual-first telegraphs** — audio is never the only cue.
- **Closed-caption style** — caption size and placement options.

### 11.5 Screen reader

- **HUD announce** — HP changes, item pickups, room contents narrated on entry.
- **Codex navigable** — all pages have semantic structure and are linearly navigable.
- **Combat log announce toggle** — reads each log line.
- **Minimap described on demand** — "You are at column 5 row 4. Boss is 6 cells north. Unvisited cells adjacent: NE, E, SE."

### 11.6 Accessibility-critical UI elements

| Element | Must have |
|---|---|
| HP bar | Color + shape + numeric |
| Telegraphs | Color + stripe pattern + audio + caption |
| Treasure / potions | Icon + label + hotkey |
| Minimap | Screen-reader describable |
| Combat log | Readable by screen reader, persistent toggle |
| Shop items | Tooltip navigable by keyboard |

---

## 12. Flagged Issues — Information That Doesn't Translate Cleanly

### 12.1 The "dice pool" problem

The source exposes Attack Dice and Defence Dice as literal numbers (1 to 8). In RT, most players do not think in dice. We must either:
- Keep the dice language (authentic, jargon-heavy)
- Translate to percent/damage/block numbers (accessible, loses flavor)

**Recommendation:** keep the dice language in HUD ("AD: 3 / DD: 2") but on hover show "expected damage 0.50 / block 0.33" in the tooltip. Best of both.

### 12.2 The "no revisit" rule in RT

Paper has a bright line: no revisit. RT has doors and rooms; a player naturally wants to backtrack to the previous room. Either:
- Lock doors behind the player on exit (feels arbitrary)
- Allow physical backtrack but mark revisited rooms as "already resolved" and skip content (faithful but confusing)
- Show a visible "seal" closing behind the player (flavor + clarity)

**Recommendation:** visible seal + "already-cleared" icon. RT-NATIVE flavoring, not source drift.

### 12.3 Reachability loss condition (A-1)

The source has a cryptic "don't block yourself" rule. RT must signal this before it's fatal:
- **Proactive warning:** when the next move would sever End-reachability, show a yellow "!" on that cell and confirm on click.
- **Hard block option:** setting to disallow self-blocking moves entirely (defensive mode).
- **Post-death explanation:** dashboard highlights the move that sealed the loss.

### 12.4 Clutter risk

The source sheet is *beautifully minimal*: grid + six tracker boxes. RT temptation is to pile on ability icons, quest markers, XP bars, menus. **Resist.** The HUD spec in Section 7 is already near the ceiling. Any additional element must justify itself against a deletion.

Explicit cuts from typical action-game HUDs:
- No XP bar
- No quest tracker
- No minimap legend
- No party frame
- No ability hotbar beyond Potion (+ maybe Dodge)
- No chat
- No guild / social

### 12.5 Combat log as paper trail

The source doesn't have a combat log — the dice roll, the result lands, you move on. But in RT the log is critical for understanding what happened during fast combat. This is an RT-native add but should be kept quiet: 4 lines, 6s fade, not a wall of text.

### 12.6 Tooltip depth vs. flow

The Codex contains the full level tables and monster lore. This is pause-worthy reading. But if players open it mid-dungeon, it should NOT pause the game by default (that breaks RT pacing). Provide a "pause on Codex open" toggle in accessibility settings (on by default for accessibility profiles; off by default for combat profiles).

### 12.7 "Future rolls unknown" vs. replay seeds

The dashboard (Section 10) offers a seed-share. This is an RT-native add that creates a *shared* form of known information that paper never had. Flag: this is a trust-preserving replay aid, not a strategic exploit, because the RNG is per-seed and the player still has to play it out.

### 12.8 Source has no "ambient noise" but RT needs it

Paper is silent. RT needs audio. All audio is RT-NATIVE and should be tagged as such in the asset list. Do not pretend audio was in the source.

---

## 13. Closing Principles

1. **Trust the player.** If it was in the rulebook, it goes in the Codex. No gating.
2. **Glanceable first.** If it's needed in combat, it's a bar/icon — not text.
3. **Preserve minimalism.** The source sheet is six boxes and a grid. Resist the urge to add HUD elements that don't map back to a paper tracker or an RT-native necessity.
4. **Name RT-native additions.** Telegraphs, dashboard, audio, enemy nameplates — all tagged so the source relationship stays legible.
5. **Reveal, don't hide.** Layout always visible. Contents hidden until entry. Probabilities always visible. Future rolls unknowable.
6. **Every information element has a home.** HUD / Minimap / Room reveal / Inspect / Codex / Dashboard. If it doesn't fit a zone, reconsider whether the player needs it at all.

---

## Appendix A — Quick Reference

| Info Zone | Read time | Example elements |
|---|---|---|
| HUD | <1s | HP, AD, DD, potions, minimap |
| Room reveal | 1–2s | Banner, spawn VFX |
| Tooltip | 3–5s | Item effect, shop cost, enemy stats |
| Codex | 10–60s | Level tables, bestiary, rules |
| Dashboard | 30s–5min | Full run log, path replay, stats |

| Decision | Choice |
|---|---|
| Fog of war | No layout fog + content fog |
| Randomness visibility | Full (Codex from turn 1, no meta-gating) |
| Dice language in HUD | Keep, annotate with expected values on hover |
| Telegraphs | All RT-native, visual + audio + animation |
| Post-run dashboard | Yes, replayable path + roll history |
| Pause in combat | Allowed (solo game) |
| Backtrack through cleared rooms | Physical yes, content skipped, visible seal |

*End of InfoArchitecture.md*
