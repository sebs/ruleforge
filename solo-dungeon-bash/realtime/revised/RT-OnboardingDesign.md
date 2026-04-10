# Revised Onboarding Design — Solo Dungeon Dash (Real-Time)

> RealTimeForge Stage RT-D8
> Supersedes: `output/solo-dungeon-bash/OnboardingDesign.md` (board-game version)
> Genre lock: **Action Roguelite / Dungeon Crawler**, 2D top-down, hand-drawn ink-on-parchment, Roll-and-Parry combat, 9×10 room grid, ~20 min runs, death resets.

---

## 1. Onboarding Philosophy

The source onboarding was a **14-screen wall of narrated text** over a scripted 5×5 micro-run. That was correct for a solo roll-and-write — in a pencil-and-paper game the text IS the input modality. **It is the wrong shape for Solo Dungeon Dash.**

Action games teach through play, not through reading. The player's hands must be on the controls inside the first three seconds, and no narrator should ever stop them from moving. Our reference points:

- **Hades** — seamless, contextual hints that appear near the hand and fade. No modal dialogs. The tutorial is the first five rooms of the first run.
- **Dead Cells** — the first level IS the tutorial. Every mechanic is introduced by a room whose geometry demands it.
- **Celeste** — one mechanic per screen. Never more than one new verb at a time. The room is the lesson.
- **Sekiro** — the training dummy is generous, the real world is not. Our "Orc Dummy" in Room 3 has a 1200ms telegraph and a 500ms parry window; the real game's Orc has a 600ms telegraph and a 200ms parry window.

### Four guiding principles

1. **Teach by doing, not by reading.** UI copy is never more than 6 words on screen at a time, and only appears contextually at the moment the mechanic is first usable. No text walls. No pause-the-world tutorials. Painted-on-floor iconography over sentences where possible.
2. **One mechanic per room.** Celeste's rule. If a room introduces Parry, it does not also introduce Shop. Mechanics stack; they don't collide.
3. **Fail-forward.** Every tutorial action can fail without game-over. If the player eats the Orc's hit in Room 3, they take ZERO damage (the hit is a scripted flinch) and the tutorial continues. If the player dies anyway (impossible by script, but sitting still for 60s in Room 3 triggers a "mercy kill" scenario), they respawn at the start of the room with a softer hint.
4. **The tutorial is diegetic.** There is no "LEARN" button on the title screen. The first run IS the tutorial; the first six rooms are a scripted Atrium wing of the dungeon, the seventh is the real dungeon. A returning player's save flags the Atrium as already-completed and their first run spawns them directly in the real first row, skipping it entirely.

### What this costs the source onboarding
- The source's narrator banners are **gone**. All text is environmental (painted on floor) or contextual one-shot (small caption near the relevant element).
- The source's scripted 5×5 micro-run is **gone**. Replaced by a linear 6-room corridor (the Atrium Wing) with forced geometry.
- The source's "Screen 0 Welcome" is **gone**. The player spawns in Room 1 with their hands already on the controls.

---

## 2. Complexity Rank — Order of Introduction

The order mechanics are taught, from first-second-of-play to last. Each item is introduced when, and only when, the preceding item is understood.

| # | Mechanic | Where taught | Why this order |
|---|---|---|---|
| 1 | **Movement (WASD / left stick)** | Room 1 "The Atrium" | First physical contact. Nothing else can happen until the player can move. |
| 2 | **Doorways / room transitions** | Room 1 → Room 2 threshold | The door opening on approach is self-teaching. No text needed. |
| 3 | **Dodge roll (safe first — no enemies)** | Room 2 "The Passageway" | Dodge is the second verb, taught on empty ground where missing it has zero cost. |
| 4 | **Telegraph reading** (scripted dummy that telegraphs slowly) | Room 3 "The First Foe" | The Orc Dummy glows, windmills its arm for 1200ms, flashes red at 600ms. Player learns: "red = danger." |
| 5 | **Parry** (scripted easy parry with huge window) | Room 3 "The First Foe" | Immediately after telegraph-reading, the parry prompt appears with a 500ms window (vs. 200ms in real game). |
| 6 | **Dice Tray** (visible after first successful attack) | Room 3 "The First Foe" | The tray only appears when the player earns their first kill. Reveal-as-reward. |
| 7 | **Hearts / damage** (controlled hit) | Room 3 or Room 6 | If the player hasn't taken a hit by Room 6, the Paired Orcs guarantee one. A lost heart pip animates visibly; the player learns what damage looks like. |
| 8 | **Potion healing** | Room 4 "Hoard Room" | A potion drops alongside treasure; auto-pickup, then the Q-key prompt appears next time the player is at <17 HP. |
| 9 | **Treasure pickup** | Room 4 "Hoard Room" | 3 treasure piles, walk-over auto-pickup, counter animates. Teaches passive resource gain. |
| 10 | **Shop Shrine** (scripted, cheap item) | Room 5 "The Shrine" | Altar visible on entry; one item, one price, affordable by the Room 4 treasure haul. Guaranteed successful purchase. |
| 11 | **Slot exclusivity** | Room 5 (if player tries to buy a second weapon, which the script prevents) | Deferred to real-game contextual hint. Tutorial only teaches ONE purchase. |
| 12 | **Reachability warning** | First real-game room where BFS detects a potential block | NOT taught in the Atrium Wing. Surfaces contextually in the real dungeon. |
| 13 | **Boss arena teaser** | Cutscene after Room 6 | Map pulls back, camera flies to show Dracular's sealed door at the top of the grid, then returns. Emotional stakes, not mechanics. |

**Explicit deferrals.** The following are **not** taught in the tutorial and are left for contextual hints in real play:
- Slot exclusivity (blocked weapon purchase)
- Reachability / path-blocking warning
- Stamina meter (regenerates in rest beats; taught implicitly)
- Map overlay / Ponder mode (taught by a Hint-1 toast the first time the player stands still for 8s in a combat room)
- Monster type tells beyond the Orc (every new monster family gets a one-shot "This one has more dice" caption)

---

## 3. Progressive Disclosure Plan

The Atrium Wing is a **6-room linear corridor** grafted onto the bottom of the 9×10 grid, sitting below row 0. Its rooms are visually distinct — lighter parchment, thicker ink borders, painted instructional glyphs on the floor. When Room 6's exit door opens, the camera pulls up to reveal the real dungeon, and the Atrium Wing seals behind the player forever (both for this run and across the save file if tutorial is completed).

```
[Room 6: The Pair]           ← tutorial ends, door to real dungeon
      |
[Room 5: The Shrine]         ← first shop
      |
[Room 4: The Hoard Room]     ← treasure + potion
      |
[Room 3: The First Foe]      ← scripted combat (parry, attack, dice tray)
      |
[Room 2: The Passageway]     ← movement challenge (dodge + doorway choice)
      |
[Room 1: The Atrium]         ← spawn, movement only
```

### Per-room complexity budget

| Room | New verbs | Assisted parameters | Expected time |
|---|---|---|---|
| Room 1: Atrium | Move | — | 10–20s |
| Room 2: Passageway | Dodge, doorway choice | — | 15–25s |
| Room 3: First Foe | Telegraph read, Parry, Attack, Dice Tray reveal | Telegraph 1200ms (real: 600ms); parry window 500ms (real: 200ms); damage-on-fail = 0 | 30–60s |
| Room 4: Hoard Room | Treasure pickup, Potion pickup, Q-heal | — | 15–30s |
| Room 5: Shrine | Shop approach, purchase | Single item, single price, auto-affordable | 20–40s |
| Room 6: The Pair | Dodge + Parry in rotation, 2-enemy positioning | Telegraphs 800ms (real: 600ms); parry window 300ms (real: 200ms) | 30–90s |

**Target median tutorial duration: ~3 minutes.** Target 90th percentile: ~6 minutes. If a player takes longer than 8 minutes, the error-mode hint system steps in.

**Difficulty ramp within the Atrium Wing:** Room 3's Orc Dummy is trivial by design — the first parry should *always* succeed for any player who sees the red flash. Room 6's Paired Orcs are a real test, but with still-forgiving parameters. The jump from Room 6 to the real dungeon's first Orc (600ms telegraph, 200ms parry window, full damage) is the true difficulty cliff, and it is deliberate — the player must *want* to learn real parry timing.

---

## 4. Tutorial Scripted Scenario

Full beat-by-beat script. All UI copy is shown in SMALL CAPS for on-screen captions and in italics for painted-on-floor glyphs or environmental text.

### Room 1 — "The Atrium"

**Scene.** The player spawns in the centre of a small octagonal chamber, about 1.5 screens wide. Parchment-textured floor. A single sealed door at the top. **Painted on the floor in ink**, radiating outward from the spawn point, are four arrows pointing up/down/left/right, labelled *W A S D* next to each arrow. A tiny animated gamepad left-stick icon pulses beside the painted arrows for gamepad players.

**World state.** No enemies. No items. No timer. No HUD yet except a faint heart row at the top-left.

**Beat 1 — spawn.** Camera is already framed on the room. Player has control from frame 1.

**Beat 2 — first movement.** As soon as the player presses any movement key, a small caption fades in at the bottom-centre of the screen:

> MOVE — WASD OR LEFT STICK

The caption disappears after 3 seconds or after the player has moved 2 tile-lengths, whichever comes first. Never blocks input. Never requires dismissal.

**Beat 3 — approach the door.** As the player walks toward the top door, the door emits a soft ink-ripple and the painted floor arrows *blow away like ash*. When the player is within 0.5 tiles of the door, it creaks open with a brush-stroke animation.

**Beat 4 — cross threshold.** Player walks through. Camera pans up to frame Room 2. Door seals behind with a heavy inkblot splash.

**Exit condition:** Player crosses the Room 1 → Room 2 threshold. No skip allowed this early (but the whole tutorial is skippable from the pause menu on first launch via a single "SKIP TUTORIAL" button).

**Failure mode:** If the player does not move within 30 seconds, the painted arrows on the floor *pulse brighter* and a soft chime plays. If 30 more seconds pass, a small caption appears: "MOVE WITH WASD OR LEFT STICK" with a visible key glyph.

---

### Room 2 — "The Passageway"

**Scene.** A long corridor, roughly 2.5 screens tall. The player enters from the bottom. Two doors at the top — one on the left, one on the right. A painted-on-floor glyph halfway up shows a stylized figure doing a dodge-roll, with the label *SPACE / B* next to it.

**World state.** No enemies. The corridor has a single harmless obstacle: a fallen column halfway across. Stepping over it is trivial, but the painted dodge-roll glyph invites the player to try the dodge verb.

**Beat 1 — entry.** Camera settles. Caption:

> TWO DOORS. ANY UNVISITED DOOR WILL DO.

Caption fades after 4 seconds.

**Beat 2 — the dodge glyph.** As the player walks past the painted dodge icon, a subtle highlight pulses around the icon and a second caption appears:

> DODGE — SPACE OR B (BRIEF INVINCIBILITY)

The caption does not demand the player use dodge. If the player does dodge-roll, the game emits a satisfying swoosh and the painted icon ink-splatters, leaving a small "✓" behind. If they don't, the glyph fades naturally as they pass.

**Beat 3 — doorway choice.** The player picks one of the two doors. Both lead to Room 3; the choice is cosmetic. On crossing either threshold:

- The unchosen door seals with an inkblot.
- The chosen door opens; camera pans to reveal Room 3.

**Exit condition:** Player crosses either Room 2 → Room 3 threshold.

**Failure mode:** 30 seconds of no movement → caption "TRY WALKING TO EITHER DOOR" fades in for 4 seconds.

---

### Room 3 — "The First Foe"

**Scene.** A wider octagonal arena, 2 screens wide. In the centre stands the **Orc Dummy** — a cartoonish Orc sketched with extra-thick ink lines, wearing a straw "TRAINING" sign around its neck. The Orc is stationary, facing the player's entry door. No other enemies.

**World state.** One scripted Orc Dummy with custom parameters:
- Telegraph duration: **1200ms** (double the real Orc's 600ms).
- Red-flash moment: 600ms into the telegraph (i.e. 600ms before the swing lands).
- Parry window: **500ms** (2.5× the real 200ms).
- Damage on unblocked hit: **0** (scripted flinch, no pips lost).
- HP: 1 (dies on first opening strike).

**Beat 1 — world pause.** The moment the player crosses the threshold, the world soft-pauses for ~1 second. Music dips to a low drone. A caption appears at the top of the screen, with the parry button glyph inline:

> WATCH THE GLOW. WAIT FOR RED. PARRY [RMB / LT].

The world resumes after 1 second regardless of player input.

**Beat 2 — the Orc telegraphs.** The Orc Dummy begins its attack cycle immediately: a big, slow windmill wind-up. At 0ms the arm begins rising. At 600ms the entire Orc flashes red and a visible chord plays. At 1200ms the arm strikes the air where the player stands.

**Beat 3a — if the player parries successfully.** Hit-stop for 80ms. The world slowmos to 25% speed for 900ms. A sharp *clang* plays. The Orc reels; its Guard drops and a glowing **OPENING** shimmer appears around it. Caption:

> OPENING — STRIKE NOW.

**Beat 3b — if the player fails to parry (either no input or too late).** The Orc's arm lands. The player's sprite flinches visibly but takes zero damage. Caption:

> MISSED. IT'S OK — WATCH THE RED FLASH. TRY AGAIN.

The Orc resets to its starting pose and begins the telegraph cycle again. The cycle repeats indefinitely until the player successfully parries.

**Beat 4 — the attack.** During the Opening slowmo, the player presses LMB / RT to attack. The hero swings. On connect, the Orc's silhouette breaks into ink particles. **And then:**

**Beat 5 — the Dice Tray reveal.** The Dice Tray flies in from the right edge of the screen for the first time in the game, accompanied by a satisfying wood-on-stone sound. It displays the player's starting 3 Attack Dice, which animate rolling in slow-mo. One of the three dice lands on a **6** (scripted). The 6 pulses with a gold glow. A caption appears immediately:

> YOU ROLLED A SIX. THAT'S A HIT. DICE ARE YOUR DAMAGE.

The Orc bursts into ink and a small "✓ 1" floats above the tray, showing the hit count.

**Beat 6 — room clear.** The door at the top of Room 3 opens. The Dice Tray slides to its permanent position on the right side of the HUD (where it will live for the rest of the game). Caption fades.

**Exit condition:** Player crosses Room 3 → Room 4 threshold.

**Failure modes:**
- If the player sits still with no parry attempts for 30 seconds, the red-flash moment extends to 1000ms into the telegraph (nearly impossible to miss) and the caption becomes "HOLD RMB WHEN THE ORC FLASHES RED."
- If the player has been in Room 3 for 120 seconds without a successful parry, the Orc Dummy stops attacking and the caption becomes "TRY PRESSING RMB ANY TIME NOW." The next parry succeeds regardless of timing.

---

### Room 4 — "Hoard Room"

**Scene.** A small octagonal chamber with three hand-drawn treasure piles scattered around the floor and a single glass potion bottle resting on a small plinth near the exit door.

**World state.** No enemies. 3 treasure pickups (scripted values: +1, +2, +1 = 4 Treasure total). 1 Potion pickup. Auto-pickup on walkover.

**Beat 1 — enter.** Camera settles. No caption yet — the treasure piles are visually unmistakable.

**Beat 2 — first treasure.** Player walks over the nearest pile. Gold coins animate up into the HUD treasure counter (0 → 1). A tiny gold "+1" particle floats above the pickup. Caption:

> TREASURE — WALK OVER IT.

**Beat 3 — second and third piles.** Player collects the other two piles. Counter climbs to 4. No new captions; the mechanic is already understood.

**Beat 4 — the potion.** As the player approaches the exit door, they cross the potion plinth. The potion fades into the HUD potion counter (0 → 1). Caption:

> POTION — PRESS Q / Y TO HEAL.

The caption stays visible for 5 seconds, long enough for the curious player to try it. If the player presses Q at full health, a soft "denied" chime plays with a small caption: "FULL HEALTH — SAVE IT FOR LATER."

**Exit condition:** Player crosses Room 4 → Room 5 threshold. Treasure count is scripted to be exactly 4 so Room 5's shop item is always affordable.

**Failure mode:** None — this room has zero fail states. If the player somehow skips a treasure pile, that's fine; the scripted values are generous enough that 1 missed pile still leaves them with 2+ Treasure, enough for the Shrine.

---

### Room 5 — "The Shrine"

**Scene.** A dimly-lit chapel-like room. In the centre rises a stone altar with a single glowing rune above it — the Buckler icon, drawn in ink. Next to the altar is a price tag: *2 TREASURE*. No other items. No enemies.

**World state.** One Shop Shrine, scripted with exactly one item (Buckler, +1 Defence Die, price 2). The player has 4 Treasure from Room 4.

**Beat 1 — entry.** Camera settles. Caption:

> APPROACH THE SHRINE. PRESS [E] / [X].

**Beat 2 — interact.** Player walks within 1 tile of the altar. The altar glyph pulses gold. Interact prompt appears floating above the altar: *[E]*.

**Beat 3 — shop opens.** Player presses E. The world locally freezes (Shop Shrine behaviour, per AgencyModel §8). The Buckler rune expands into a small shop card showing: icon, name, price "2", effect "+1 DEFENCE DIE". Caption:

> TAP AN ITEM TO BUY.

**Beat 4 — purchase.** Player clicks / confirms. The Buckler rune descends onto the altar as a physical pickup. Treasure counter deducts (4 → 2). The shrine sinks back into the floor. Dice Tray animates: a new die slides in from the right, joining the existing 3 Attack Dice as a **Defence Die** (distinct colour — blue ink — so the player can see the tray now has two kinds of dice). Caption:

> NEW DICE — ADDED TO YOUR TRAY.

**Beat 5 — walk to exit.** The exit door opens as the player approaches. No further captions.

**Exit condition:** Player crosses Room 5 → Room 6 threshold.

**Failure modes:**
- If the player ignores the altar entirely and walks toward the exit door, the door stays SEALED until they interact with the Shrine at least once. A hint caption appears: "THE SHRINE — PRESS [E]."
- If the player opens the shop and closes it without purchasing, they can still proceed; the shop re-opens on re-interact. After 60 seconds with no purchase, a softer hint: "TAP THE BUCKLER TO BUY IT."

---

### Room 6 — "The Pair"

**Scene.** A larger arena, 2.5 screens wide, with two Orcs (not Dummies — these are normal Orcs with slightly relaxed parameters). The Orcs stand on either side of the room, facing the player.

**World state.** Two scripted Orcs:
- Telegraph duration: **800ms** (vs. real 600ms).
- Parry window: **300ms** (vs. real 200ms).
- Damage on hit: **1 pip** (REAL damage, first time in the tutorial).
- HP: 1 each.
- Attack cadence: alternating — Orc A telegraphs first, then Orc B after A's cycle resolves. They do NOT attack simultaneously during the tutorial; simultaneous attacks are a mid-run mechanic.

**Beat 1 — entry.** Camera settles. Caption:

> TWO FOES. DODGE. PARRY. LEARN.

**Beat 2 — combat.** The player fights both Orcs. The script allows the player to take up to 2 heart pips of damage without penalty; more than 2, and the fight restarts via the respawn system (see Error Modes §9). The dice tray rolls visibly on every attack.

**Beat 3 — clear.** When both Orcs are dead, the exit door — now visible at the top of the arena — emits a bright ink flare and opens.

**Exit condition:** Player crosses Room 6 → dungeon threshold.

**Failure modes:**
- If the player dies in Room 6 (loses all 17 hearts, which is extremely unlikely given scripted damage caps), they respawn at the Room 6 entry with full health and a softer caption: "TAKE IT SLOW. WAIT FOR THE RED FLASH." The Orcs' telegraph extends to 1000ms and parry window to 400ms for the second attempt.
- If the player sits still for 30 seconds without engaging, the Orcs advance slowly toward the player and a caption appears: "APPROACH AND STRIKE."

---

### Post-Room 6 Cutscene — "The Real Dungeon"

**Scene.** The moment the player crosses Room 6's exit, the camera lifts and pulls back, up and up, until the entire 9×10 grid of the real dungeon is visible as a hand-drawn map overlay. The player's current position — still at the Atrium Wing exit — pulses with a gold glow. Far at the top of the grid, a single sealed door glows red: **Dracular's chamber.**

A short caption floats on screen in a slightly more dramatic hand-lettered typeface:

> YOU ARE NOW IN THE REAL DUNGEON.
> DRACULAR WAITS ABOVE.

The caption holds for 3 seconds. The camera then zooms back down to the player's current room (which is now Row 0, Column 4 of the real dungeon — the first legitimate room). The tutorial is over. The Atrium Wing has sealed behind the player forever.

From this point, all parameters are real: 600ms telegraphs, 200ms parry windows, real damage, BFS reachability warnings active, the full 25-mechanic game live.

---

## 5. Tooltip & Hint Library

Contextual one-shot hints that fire inside the real game, not the tutorial. Each hint is shown **once per save file** (opt-in re-show in Settings). All hints appear as small captions at the bottom-centre or anchored near the relevant UI element; none pause the game.

| ID | Trigger | UI Copy |
|---|---|---|
| H-01 | First time parry fails (miss or too-late) in real dungeon | "Parry window is ~200ms. Wait for the red glow, not the yellow." |
| H-02 | First time a potion is purchased at any Shop Shrine | "Bought potions unlock next room." |
| H-03 | First time approaching a tile that BFS flags as potentially blocking | "Be careful — that path may trap you." |
| H-04 | First time entering combat with a tier-4+ monster (Troll, Skeleton Lord, Demon, etc.) | "This one has more dice. Wait longer for the opening." |
| H-05 | First time the Dice Tray shows a full 5+ Attack Die pool | "Your dice tray is full. Every hit is stronger now." |
| H-06 | First time the player reaches Row 9 (the final row before End) | "The boss waits beyond the last door. Ready yourself." |
| H-07 | First time HP drops below 5 | "Low health. Drink a potion — Q or Y." |
| H-08 | First time the player stands still for 8+ seconds in a non-combat room | "Hold TAB / Select to open the map overlay." |
| H-09 | First time the player takes damage from an unparried simultaneous two-enemy attack | "Two tells at once. Dodge first, then parry the second." |
| H-10 | First time the player buys a gear item that would conflict with a currently-owned item slot | "That slot is taken. Buying will replace what you own." |
| H-11 | First time the player dies in real dungeon | "Runs reset on death. You keep the bestiary, not the gear." |
| H-12 | First time the player completes a run (reaches End and survives Dracular) | "You beat Dracular. Try a harder seed tomorrow." |

**Implementation note.** Hints are stored in a lightweight state dict keyed to save file. They are suppressed entirely on daily-seed runs (competitive integrity) and on Hard/Nightmare difficulties (opt-in "no training wheels" modes).

---

## 6. Non-Tutorial "Help" Layer

For players who skip the tutorial (via title screen "SKIP TUTORIAL" button, or returning players whose save flags the Atrium Wing as complete), a single scrollable Help screen is accessible at any time from the pause menu. No gameplay interaction required.

### Help screen structure (single scrollable page)

**Section 1: Controls reference**
- Movement: WASD / left stick
- Attack: LMB / RT
- Parry: RMB / LT
- Dodge roll: SPACE / B
- Heal (use potion): Q / Y
- Interact (Shop Shrine, doors): E / X
- Map overlay / Ponder mode: TAB (hold) / Select (hold)
- Pause: ESC / Start

**Section 2: Combat verbs**
- **Parry** — press RMB / LT during the monster's red-flash window (~200ms). Cancels the attack, opens a strike window, banks a Defence die roll.
- **Dodge** — press SPACE / B to roll through a monster attack with ~300ms of invincibility. Costs 2 stamina. Forfeits the strike window.
- **Attack** — press LMB / RT during an Opening to strike. Rolls your Attack Dice; each 6 is a hit. Normal monsters die to one hit.
- **Heal** — press Q / Y to drink a potion. +1 heart. Useable any time, including mid-combat.

**Section 3: Monsters overview**
A 3-column grid of monster sprite / name / "dice count" stat, from weakest (Orc, 1 AD) to strongest (Demon, 10 AD), plus Dracular (9 AD / 9 DD, boss). Each row has a 1-line flavor text.

**Section 4: Shop items overview**
A 3-column grid of gear icon / name / slot / effect.
- Buckler (armour): +1 Defence Die
- Shield (armour): +2 Defence Dice (conflicts with Buckler)
- Spiky Armour (body): +1 Defence Die + 1 reflect pip
- Magical Armour (body): +2 Defence Dice
- Big Sword (weapon): +1 Attack Die, faster swing
- Big Axe (weapon): +2 Attack Dice, slower swing + stagger
- Potion Packs (consumable): 1 / 3 / 6 potions for 1 / 2 / 3 Treasure

**Section 5: Win / lose conditions**
- **Win:** Reach the End room at the top of the grid and defeat Dracular in the Nine Dice Duel.
- **Lose — death:** HP reaches 0.
- **Lose — blocked:** Your path to the End is severed by no-revisit rules. (BFS warning fires on any move that would cause this.)
- **Lose — quit:** Well.

The page is scrollable with no transitions, no animations, no interactivity beyond scrolling. Print-friendly. Accessibility-friendly.

---

## 7. Tutorial Metrics

Measure these via anonymous telemetry (opt-in at first launch; on by default but with a clear toggle):

| Metric | Target | Why it matters |
|---|---|---|
| **% who complete tutorial** (spawn to Room 6 exit) | ≥ 95% for first-time players | If < 95%, the tutorial is failing — either too hard or too boring. |
| **Time to first successful parry** (from Room 3 entry to first parried swing) | Median ≤ 15 seconds; 90th pct ≤ 45 seconds | Tests whether Room 3's scripted forgiveness is working. |
| **Time to first successful shop purchase** (from Room 5 entry to Buckler bought) | Median ≤ 20 seconds; 90th pct ≤ 60 seconds | Tests whether the single-item shop is self-explanatory. |
| **Skip rate for returning players** | 100% (tutorial auto-skipped on save flag) | If returning players see the tutorial again, onboarding is broken. |
| **Death rate in tutorial** | < 5% | If > 5%, the scripted damage caps are too loose. |
| **Tutorial duration (full, first-time player)** | Median 3 min; 90th pct ≤ 6 min | If > 6 min for a median player, the tutorial is too long. |
| **% who engage with Dodge glyph in Room 2** | ≥ 60% | If < 60%, the painted glyph is not salient enough. |
| **% who press Q (heal) in the tutorial** | ≥ 40% (optional behaviour) | Soft metric; informs Hint-H07 tuning in real play. |
| **% first-run attempts (after tutorial) that last ≥ 10 rooms** | ≥ 60% | Inherited from source metric. Tests whether skills transfer. |
| **% first-run attempts (after tutorial) that kill Dracular** | ≥ 15% (stretch goal: 25%) | Ultimate test of onboarding effectiveness. |

**Instrumentation note.** All metrics are collected as anonymous counters keyed to a device-scoped install ID, never a user account. No per-input latency tracking (that's a privacy footgun and not needed for these targets).

---

## 8. Accessibility in Onboarding

The tutorial is where accessibility settings must either be already-enabled or easily discoverable. We do not force a Settings menu on the player before the first run, but we do provide multiple paths.

### Settings accessible before tutorial starts
- **First-launch prompt (optional, 1 screen).** On the very first launch ever, after the splash, a single compact screen offers three toggles with large touch targets:
  - Slow-mo mode (25% game speed, applies to all combat)
  - Wider parry windows (parry windows are doubled — 400ms real game, 1000ms tutorial)
  - Screen reader captions (all on-screen captions are also spoken aloud)
  Plus a "SKIP THESE — SET LATER" link. Default all three OFF.
- **Tutorial-room accessibility button.** A small accessibility icon persists in the top-right of the HUD during the Atrium Wing. Tapping opens a compact accessibility panel without pausing the tutorial (the current room soft-pauses; the real game does not). This panel is identical to the first-launch prompt.

### Accessibility integration with the tutorial flow

**Slow-mo mode.** If enabled, the entire tutorial (and real game) runs at 25% game speed. The Orc Dummy telegraph is still 1200ms in wall-clock terms but feels like ~5 seconds at 25%. Captions, dice rolls, and animations all scale proportionally. Slow-mo can be toggled mid-game from the pause menu without penalty.

**Wider parry windows.** If enabled, Room 3's parry window becomes 1000ms (up from 500ms), Room 6's becomes 600ms (up from 300ms), and all real-dungeon parry windows become 400ms (up from 200ms). The game flags this as "Accessibility Mode" in the post-run dashboard — achievements still unlock, but daily-seed leaderboards are filtered to an "Accessible" bracket.

**Screen reader captions.** All on-screen captions are dispatched to the platform screen reader (NVDA on Windows, VoiceOver on macOS, TalkBack on Android). Additionally, combat events emit audio cues: a telegraph start has a distinct ascending chirp, the red-flash has a distinct percussive thud, a successful parry has a high clang, a failed parry has a low thud, dice 6s ring with a bell tone. These audio cues are designed to be fully sufficient for a sightless player to complete the tutorial on Slow-mo.

**Caption text size.** Scalable from 75% to 200% via the accessibility panel. Default is 100%. All tutorial captions respect the chosen scale.

**Motor accessibility.** Dodge can be rebound to a hold-instead-of-tap gesture. Parry can be rebound to any key including mouse side-buttons. Full keybind remapping is available from the first-launch prompt's "More options" subpanel.

**Colour contrast.** The red-flash telegraph is the only colour-coded cue that matters for tutorial completion. A "High contrast mode" setting replaces red with a heavily saturated magenta + a shape-change (the Orc's silhouette gains a visible white outline at the red-flash frame). Colour-blind players can complete the tutorial identically.

**No required audio.** The tutorial is fully completable with audio muted. All cues have a visual counterpart.

---

## 9. Error Modes

What happens when the player deviates from the happy path? Every error has a graceful recovery.

### E-1: Player sits still for 30 seconds in a tutorial room

**Room 1 (Atrium):** Painted floor arrows pulse brighter. Soft chime. After 60s total idle, caption fades in: "MOVE WITH WASD OR LEFT STICK" with a key glyph.

**Room 2 (Passageway):** After 30s, caption: "TRY WALKING TO EITHER DOOR."

**Room 3 (First Foe):** The Orc Dummy pauses its telegraph cycle after 30s of no parry attempts, then resumes with a wider red-flash window (1000ms instead of 600ms). After 120s total with no successful parry, the Orc stops attacking entirely and the next player parry attempt succeeds regardless of timing.

**Room 4 (Hoard Room):** After 30s, a subtle trail of ink particles leads from the player to the nearest treasure pile.

**Room 5 (Shrine):** After 30s, the altar rune pulses louder and a caption appears: "APPROACH THE SHRINE."

**Room 6 (The Pair):** After 30s, the Orcs slowly advance toward the player and caption: "APPROACH AND STRIKE."

### E-2: Player dies in the tutorial

**Scenario.** Extremely rare (scripted damage caps should prevent it), but possible in Room 6 if the player deliberately eats every hit.

**Handling.** Fade to black with an ink-bleed transition. Respawn at the ENTRY of the room they died in, with full HP (17 pips), full potion inventory from Room 4, and the Dice Tray intact. A softer hint caption appears: "TAKE IT SLOW. WAIT FOR THE RED FLASH." The Orcs' parameters relax by 25% for the retry (telegraphs +200ms, parry windows +100ms). If the player dies a second time in the same room, the room's parameters relax by another 25% and a mercy caption appears: "YOU'LL GET IT. STAY PATIENT." A third death auto-completes the room with a gentle "Let's move on" and the door opens. No shame, no retry limit advertised.

### E-3: Player ragequits

**Scenario.** Player closes the game or alt-tabs out during the tutorial.

**Handling.** Tutorial progress is checkpoint-saved at every room transition. On next launch, the title screen offers: "RESUME TUTORIAL (ROOM X)" as the primary action, with "RESTART TUTORIAL" and "SKIP TUTORIAL" as secondary options. The resume state includes current HP, Treasure, Potions, and Dice Tray contents — exactly where the player left off. If the player has been away for more than 7 days, the resume prompt includes a tiny "NEED A REFRESHER?" link that replays the most recent completed room's first 3 seconds as a silent vignette (no captions, no dialog).

### E-4: Player presses "SKIP TUTORIAL" during the tutorial

**Scenario.** Player has started the tutorial but wants out.

**Handling.** The pause menu's "SKIP TUTORIAL" option always works. Confirmation: "Skip? You'll start a real run at Row 0." Single click confirmation. On skip: camera fades to black, the Atrium Wing seals, and the player spawns at Row 0 of the real dungeon with default loadout (3 AD, 1 DD, 17 HP, 0 Treasure, 0 Potions) — no shortcut bonuses from the tutorial rooms. The save file flags tutorial as SKIPPED (distinct from COMPLETED), which affects exactly one thing: the Help screen gets a brief bouncing-arrow hint on the pause menu button for the first 30 seconds of real play.

### E-5: Player somehow gets stuck (geometry glitch, softlock, etc.)

**Scenario.** An unexpected bug traps the player in a non-progressable state.

**Handling.** Pause menu always has a "RESTART ROOM" option (tutorial only). Any room can be restarted at any time. If the player restarts 5+ times in the same room, a telemetry flag fires and the game offers "SKIP THIS ROOM" as a panic-button exit, which teleports them to the next room with defaults preserved.

### E-6: Player completes tutorial and immediately dies in real dungeon Row 0

**Scenario.** The difficulty cliff from tutorial to real dungeon is real. A player who breezed through Room 6 with 500ms parry windows may faceplant on Row 0's 200ms windows.

**Handling.** This is NOT a tutorial error — the tutorial is over. But we do fire hint H-01 ("Parry window is ~200ms. Wait for the red glow, not the yellow.") on the first failed parry in real play, and the "SCRAPBOOK" post-death screen shows a small tip: "Run 2 starts easier — your bestiary knows these monsters now." No actual difficulty change, but a morale boost caption to encourage a second attempt. This is the seam where real onboarding ends and the roguelite run-cycle loop takes over.

---

## 10. Handoff Notes to RT-D9 (Revised Feature List)

The onboarding flow above introduces the following feature dependencies that RT-D9 must honour:

- **Atrium Wing** is a distinct level asset: 6 hand-authored rooms that are neither procedurally generated nor part of the 9×10 grid. Level design must budget for authored content.
- **Orc Dummy** is a distinct enemy class with tunable telegraph duration, parry window, and damage-on-fail. Not the same as the real Orc enemy class.
- **Painted floor glyphs** are an art asset class (movement arrows, dodge icon) used only in the Atrium Wing.
- **Contextual caption system** must support: text, key glyph inline, auto-fade, anchor-to-element, screen-reader dispatch, scalable size.
- **One-shot hint system** must support: per-save-file state, trigger-fire, show-once, suppress-on-daily-run.
- **Accessibility first-launch prompt** is a distinct UI surface — not buried in Settings.
- **Save state checkpointing** must happen at every Atrium Wing room transition, independently from the real game's per-run state.
- **Pause menu RESTART ROOM** is a tutorial-only feature that real dungeon play does not have.

All other onboarding behaviour reuses the real game's systems (dice tray, parry, shop shrine, movement, etc.) with parameter overrides for the scripted rooms.

---

*End of RT-OnboardingDesign.md — Stage RT-D8 complete.*
