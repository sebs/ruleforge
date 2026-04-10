# Faithfulness Audit — Solo Dungeon Bash → Solo Dungeon Dash

> RealTimeForge Stage RT-8
> Purpose: Side-by-side comparison of every core mechanic in the source to its RT equivalent, with a faithfulness rating and a final score.

## Legend

| Status | Meaning |
|---|---|
| **KEPT** | Survives intact. Same behavior, same numbers, same player intent. |
| **TRANSFORMED** | Exists but changes shape. Player-observable behavior preserved; implementation differs. |
| **DISSOLVED** | Disappears. Artifact of the physical medium with no RT equivalent — player experience is replaced by a different but compatible moment. |
| **RT-NATIVE** | Did not exist in source. Added for RT to work. |

| Faithfulness | Meaning |
|---|---|
| **High** | An experienced source-game player would recognize this immediately as the same idea. |
| **Medium** | Recognizable, but the RT version feels different in ways that matter. Worth a discussion. |
| **Low** | A translation the source author might contest. Flagged for scrutiny. |

---

## Mechanic-by-Mechanic Audit

### M1: Dice Rolling (count-6s on d6 pool)

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| Roll a d6, count 6s as successes | **KEPT** | Dice Tray visibly rolls on every attack/defence, count-6s identical | **High** | The math is preserved verbatim. The only change: the player no longer physically picks up dice. |
| Monster rolls before player | **TRANSFORMED** | Monster always initiates combat by aggroing first on room entry | **High** | "Monster first" identity is preserved. |
| Attack Dice / Defence Dice pools (1–10) | **KEPT** | Exact same numbers feed the Dice Tray | **High** | No rebalance. |

**Subtotal: High faithfulness.**

---

### M2: Roll-and-Write Grid Exploration

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| 9×10 grid, 90 rooms | **KEPT** | Same 9×10 grid as the dungeon topology | **High** | Literal preservation. |
| Start square below row 1, middle column | **KEPT** | Same | **High** | |
| End square above row 10, middle column | **KEPT** | Same | **High** | |
| King-adjacency (8-way) | **KEPT** | Expressed as room doorway connectivity | **High** | Every king-adjacent room has a door. |
| No revisit rule | **TRANSFORMED** | Doors physically seal behind you | **High** | The prohibition is enforced world-literally rather than by honor. Arguably more vivid than paper. |
| Draw your path in pencil | **DISSOLVED → TRANSFORMED** | Ink trail auto-drawn on map overlay | **Medium** | The tactile pleasure of hand-drawing a line is lost. Replaced by watching an inked trail appear and by a persistent "you can see your route" map overlay. This is probably the biggest sacrifice. |
| You may return to previous dungeon levels | **KEPT** | Yes — the 9×10 graph allows traversal in any direction; rows are positional, not sequential | **High** | |

**Subtotal: High faithfulness (with one Medium).** The "draw your path" concession is the hardest pill to swallow but unavoidable in RT.

---

### M3: Resource Management (HP / Treasure / Potions)

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| Starting HP 17 | **KEPT** | 17 hearts, one heart per source HP | **High** | |
| HP cap 17 | **KEPT** | Hearts cannot exceed 17 | **High** | |
| Starting Attack Dice 1 | **KEPT** | Same | **High** | |
| Starting Defence Dice 1 | **KEPT** | Same | **High** | |
| Treasure counter (unbounded) | **KEPT** | Treasure counter | **High** | |
| Potion counter (unbounded) | **KEPT** | Potion counter | **High** | |
| Potion 1:1 → HP | **KEPT** | Same | **High** | |
| HP cap prevents over-heal | **KEPT** | Same | **High** | |

**Subtotal: High. Resource system is a verbatim port.**

---

### M4: Random Encounter Tables (per-level d6 lookup)

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| 10 level tables + 1 boss table | **KEPT** | Same tables, same probabilities, exposed in Codex | **High** | |
| d6 rolled on room entry | **TRANSFORMED** | Contents pre-seeded per run from the same distribution | **Medium** | Probabilistically identical (over the player's full career). A purist might note that pre-seeding removes the "the die decided on that room" drama. Mitigated by the content-fog reveal preserving the moment-of-surprise feeling. |
| Contents: Empty / Treasure / Potion / Monster | **KEPT** | Same four (+Shop Shrine as an RT-native 5th) | **High** | Shop Shrine is a new variant of "Empty" that the source doesn't have. |
| Monster-type-per-row distributions | **KEPT** | Same tables, same IDs | **High** | |

**Subtotal: High (with one Medium).** The pre-seeding is a real difference that purists will notice.

---

### M5: Item Shop / Loadout Tableau

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| Shop with 9 item slots | **KEPT** | Same 9 items at Shop Shrines | **High** | |
| Exact costs (1–6 Treasure) | **KEPT** | Same | **High** | |
| Exact effects (AD/DD modifiers) | **KEPT** | Same | **High** | |
| Exclusive slots (weapon/shield/armour) | **KEPT** | Same XOR constraints | **High** | |
| Shop is available "at end of turn" | **TRANSFORMED** | Shop is accessed at dedicated Shop Shrine rooms (diegetic) | **Medium** | The source allows the player to shop *every* turn; RT restricts it to specific rooms. This changes pacing — shop access is now a resource to be found. We're trading availability for agency and spatial meaning. |
| Duplicate items allowed | **N/A — Ambiguous rule (A-4)** | Single copy per item | **Low → N/A** | Source rulebook ambiguous; we resolve via the RuleForge `AmbiguousRules.md` default. Not a faithfulness issue. |
| Buckler XOR Shield, Sword XOR Axe, Spiky XOR Magical | **KEPT** | Same | **High** | |

**Subtotal: Mostly High.** The Shop Shrine room-gating is a meaningful transformation — we promote availability-of-shopping from "every turn" to "you must find one." This is justified because otherwise every room would end in a menu pause, which breaks RT flow. A source-purist might prefer the every-room shop, and we note this as the strongest Medium.

---

### M6: Push-Your-Luck

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| Deciding when to push deeper vs farm shallower | **KEPT** | Identical — the same meta-decision survives | **High** | Routing IS the push-your-luck in both versions. |
| Committing to the boss fight (one-way) | **KEPT** | Crossing into End room is one-way, confirmed by walking through the last doorway | **High** | |
| Farming trade-off (more rooms = more treasure but more risk) | **KEPT** | Same | **High** | |

**Subtotal: High. This is one of the cleanest translations.**

---

### M7: Hand Management (Potions, delay rule)

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| Stockpile potions | **KEPT** | Same | **High** | |
| Use potions between rooms (step 6) | **TRANSFORMED** | Use potions at any time via hotkey, but with constraints | **Medium** | The source restricts potion use to a specific moment (between combats); RT allows hotkey use at any time EXCEPT mid-combat (to preserve the source's A-2 strict ruling). So functionally the player can only drink between rooms = faithful in practice. |
| Bought potions usable only next turn | **KEPT** | Bought potions "pending" until the player exits the Shop Shrine room | **High** | Preserved literally. |
| Found potions usable immediately | **TRANSFORMED** | Available as soon as the player reaches the next room | **High** | Same effect. |

**Subtotal: High.** Medium rating on the first row softens slightly, but the spirit is preserved.

---

### M8: Engine Building / Character Progression

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| Permanent stat upgrades within a run | **KEPT** | Same; upgrades persist until death | **High** | |
| No persistent progression between runs | **TRANSFORMED** | Same for *combat* stats; cosmetic/bestiary unlocks persist | **Medium** | The source has zero metaprogression; RT adds light cosmetic/bestiary unlocks. We do not add stat-based metaprogression. This is a small concession to modern roguelite expectations while holding the line on power. |
| Time-value of treasure (earlier > later) | **KEPT** | Same | **High** | |

**Subtotal: High. The bestiary/cosmetics concession is a small Medium.**

---

### M9: Hit-Point Combat Subsystem

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| Monster HP always 1 | **KEPT** | 1 HP preserved for all non-boss monsters | **High** | Kept as a design signature. |
| Alternating Monster attack → Player defend → Player attack → Monster defend | **TRANSFORMED** | Compressed into a Roll-and-Parry beat: Monster telegraph → Parry window → Opening → Player attack → Loop | **Medium** | The 4-step sub-loop is still present but now expressed as a continuous rhythm rather than discrete turns. Players who know the source will recognize the structure. The sacrifice is that "initiative is strictly alternating" becomes "initiative is rhythmically driven by telegraphs." |
| No flee rule | **KEPT** | Same — no flee option in RT | **High** | |
| Monster attacks first (fixed initiative) | **KEPT** | Preserved — monster aggros immediately on room entry, player cannot out-time the first telegraph | **High** | |
| Dracular: 9 AD + 9 DD | **KEPT** | Same stats for Dracular; his 9 DD is visualized as a dice cluster that the player must burn through | **High** | |
| Combat is deterministic once rolled | **TRANSFORMED** | Still deterministic per-swing dice roll; player input determines *when* to swing, not what the dice say | **Medium** | This is an important change — the player now has agency over *timing* which they didn't have in the source. We added skill expression. Whether a purist sees this as a betrayal or an improvement depends on taste. |
| No criticals, status effects, or ability procs | **KEPT** | None of these added | **High** | Deliberate minimalism. |

**Subtotal: Mostly High with a couple of Mediums.** The Roll-and-Parry translation is the single biggest design bet in this whole adaptation, and the Mediums reflect that honestly.

---

### M10: Self-Blocking Pathfinding Constraint

| Board Game Mechanic | Status | RT Equivalent | Faithfulness | Notes |
|---|---|---|---|---|
| Lose if End becomes unreachable | **KEPT** | Same loss condition | **High** | |
| Implicit reachability check (player must notice) | **TRANSFORMED** | Active live BFS with a warning HUD element | **Medium** | Source puts the onus on the player; RT makes it explicit. This is more player-friendly but removes a "gotcha" failure mode. We believe this is a net positive. |
| You can still commit to a blocking move | **KEPT** | Yes — the warning is a warning, not a veto | **High** | |

**Subtotal: High. Medium rating on the live-check is a quality-of-life improvement, not a loss.**

---

## Dissolved Mechanics (not part of the RT game)

| Source Mechanic | Why Dissolved | Replaced By |
|---|---|---|
| Physical dice-rolling action | Player can't pause combat to roll dice | Automatic Dice Tray animation |
| Pen-and-paper tracking | Automation is the point of a digital port | Auto-HUD |
| Hand-drawing of the path | Translated to auto-drawn ink trail on map overlay | Map overlay with ink trail |
| 7-step turn sequence as a *structure* | Continuous RT flow makes strict step-counting impossible | Concurrent systems behind the scenes |
| Explicit "roll to determine contents" moment | Pre-seeded contents + on-entry reveal | Fog-of-content dissolve |
| The rulebook / printed sheet | Replaced by in-game Codex | Full Codex with all level tables |

**Dissolved mechanics count: 6.**

---

## RT-NATIVE Additions (not in source)

| Mechanic | Why needed | Faithfulness impact |
|---|---|---|
| Parry timing window (200ms) | Source combat has no player skill expression | Adds skill; arguably *increases* the dignity of combat |
| Dodge roll + i-frames | Needed so players can disengage from mis-timed parries | No source analog |
| Guard state on monsters | Without this, 1-HP enemies die in ~0.3s and combat feels cheap | Preserves lethality |
| Opening window (1s vulnerability) | Created by parry; gives player a "strike now" moment | Preserves "attack step 3" of source combat loop |
| Dice Tray on-screen UI | Source dice are physical; in RT we make them visible | Massively preserves identity — this is a keep-mechanism not a loss |
| Monster telegraphs (600ms wind-up, red flash) | Required so parry is possible | None |
| Doorway seals | Enforce no-revisit physically | Strengthens source rule |
| Reachability warning HUD | Rescue from A-1 ambiguity | Quality of life |
| Room camera transitions | Needed for room-at-a-time framing | None |
| Shop Shrine rooms | Avoid menu-pause every room | Medium tension with source's every-turn shop |
| Dracular Nine Dice Duel (3 phases) | Single-phase dice duel is not climactic in RT | Mostly additive |
| Bestiary & cosmetics | Modern roguelite expectation | Small concession |
| Daily seed challenge | Replayability engine | Optional, off by default |
| Map overlay toggle | Compensation for lost paper sheet | Brings tactility back |
| Hit-stop / VFX / audio feedback | RT combat feel requirement | None |

**RT-NATIVE mechanics count: 15.**

---

## Summary Counts

| Category | Count |
|---|---|
| **KEPT** (identical in effect) | 26 rows |
| **TRANSFORMED** (same intent, different shape) | 14 rows |
| **DISSOLVED** | 6 items |
| **RT-NATIVE** additions | 15 items |

---

## Faithfulness Scoring

### Per-mechanic scoring methodology
- **High** faithfulness = 1.0
- **Medium** = 0.7
- **Low** = 0.3

### Score calculation
We weight each mechanic area by its importance to the source's identity:

| Mechanic Area | Weight | Score | Weighted |
|---|---|---|---|
| M1 Dice Rolling | 15% | 1.00 | 0.150 |
| M2 Grid Exploration | 15% | 0.94 (avg of 6×High + 1×Medium) | 0.141 |
| M3 Resource Management | 10% | 1.00 | 0.100 |
| M4 Encounter Tables | 10% | 0.93 (3×High + 1×Medium) | 0.093 |
| M5 Item Shop | 10% | 0.93 (6×High + 1×Medium) | 0.093 |
| M6 Push-Your-Luck | 5% | 1.00 | 0.050 |
| M7 Hand Management | 5% | 0.93 (3×High + 1×Medium) | 0.046 |
| M8 Engine Building | 5% | 0.90 (2×High + 1×Medium) | 0.045 |
| M9 HP Combat | 20% | 0.85 (5×High + 2×Medium) | 0.170 |
| M10 Self-Blocking | 5% | 0.90 (2×High + 1×Medium) | 0.045 |
| **Total** | **100%** | | **0.933** |

### Overall Faithfulness Score: **93%**

This is an unusually high faithfulness score, because:
- The source game is mechanically simple (few places to break)
- No physical-only mechanics (no dexterity, no table-talk, no hidden hands)
- No multiplayer
- The two hardest translation areas (combat, shop) have their Medium ratings softened by strong core preservation

---

## Low Faithfulness Flags (potential betrayals)

**None.** No mechanic translated at Low faithfulness.

The rows that might concern a source purist most are:
1. **M2: Pen-drawn path → auto-drawn ink trail (Medium, 0.7).** This is the single biggest concession. The tactile joy of drawing can't be preserved. Mitigation: we render the ink trail beautifully, add pen-scritch audio, and support a "take notes" overlay where the player can scribble annotations.
2. **M5: Shop every turn → Shop at dedicated Shop Shrine rooms (Medium, 0.7).** We sacrificed availability for pacing. Mitigation: Shop Shrine rooms are generous (one per ~8 rooms) and players can plan routes that pass through them. Alternative considered: ambient shops in every room (rejected as too noisy) and between-room menu shops (rejected as flow-breaking).
3. **M9: Deterministic combat → Timing-based combat (Medium, 0.85).** We added skill expression. Some purists will argue the source's "deterministic once rolled" is the point. We argue that RT combat requires *some* skill expression to be a game at all; pure-RNG combat in RT is an auto-battle, which is a different genre.

None of these rise to the level of "this isn't the same game." They are honest, documented trade-offs with specific mitigations.

---

## Honorable Concessions List

Things we could have made more faithful but chose not to, each with a reason:

1. **We pre-seed dungeon contents instead of rolling on entry.** Reason: reproducibility for daily seeds, speedruns, and regression testing. Cost: an imperceptible feel change.
2. **We add cosmetic/bestiary metaprogression.** Reason: modern roguelite retention. Cost: a tiny drift from the "every run starts fresh" purity. Offset: no stat-based metaprogression at all.
3. **We add a reachability warning HUD.** Reason: A-1 ambiguity + player kindness. Cost: removes a "gotcha" mistake. Offset: warning is a warning, not a veto; players can ignore it.
4. **We add a live boss fight for Dracular instead of a dice duel cutscene.** Reason: in RT, a 30-second scripted dice sequence would be a non-interactive cinematic. Cost: the boss fight is now a 90-second rhythm encounter rather than an instant resolution. Offset: we still roll 9 AD/9 DD mathematically. The numbers are preserved.
5. **We add telegraphs.** Reason: required for parry to be possible. Cost: none — no telegraphs existed in source. Offset: telegraphs arguably make combat *more* respectful of player skill.

---

## What a source designer (Felbrigg Herriot) would likely say

Our best guess at how the original designer would react to this adaptation:

- **"The grid is still 9×10. Good."** ✅
- **"Dracular is still 9 AD / 9 DD. Good."** ✅
- **"Monsters still die to a single hit if you land it. Good."** ✅
- **"I see you kept the item slots with the same costs. Good."** ✅
- **"Wait — you added a parry window?"** *Likely raises an eyebrow. We would explain: "In RT, without a parry window, combat lasts 0.3 seconds."*
- **"And you kept the dice visible?"** *Likely smile.* The Dice Tray is our gift to the source.
- **"What about drawing the path?"** *Likely disappointment.* This is the one thing we couldn't fully preserve.

Overall: we think a fair source fan would call this **"respectful, recognisable, and a real game in its own right."**

---

## Final Verdict

**Faithfulness Score: 93%**

Classification: **High faithfulness.** The core identity is intact. Translations are honest. The RT version is recognizably the same game at every level that matters — grid, dice, numbers, items, monsters, boss, stakes — while being a playable real-time experience.

**The two biggest sacrifices are:**
1. The tactile joy of drawing your path in pencil (dissolved; compensated by ink-trail visualization).
2. Shop availability per turn vs per Shop Shrine room (transformed; compensated by generous shrine spawns).

**The biggest win is the Dice Tray** — we turned the source's most-physical element (actual dice in actual hands) into the RT game's most distinctive visual identity, which many RT translations of dice games fail to do.

**If we ever reduce faithfulness below 85%, flag it.** Current design does not approach that line.
