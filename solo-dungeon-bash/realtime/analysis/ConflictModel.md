# Conflict & Resolution Model — Solo Dungeon Bash (Real-Time Translation)

> RealTimeForge Stage RT-4
> Source: `output/solo-dungeon-bash/` (RulesExtraction, Mechanics, BalanceSheet)
> Target: 2D/3D real-time action game preserving the "lethal, bursty, deterministic once rolled" feel of the source's dice-pool combat.

---

## 1. Source Conflict Model Summary

Combat in Solo Dungeon Bash is a **strictly alternating, count-successes dice-pool resolution** with fixed monster initiative. The shape of a single encounter:

- **Monster always attacks first.** There is no initiative roll, no player agency around who swings first. The encounter is literally "you got ambushed."
- **Round structure (4 steps, repeat until dead):**
  1. Monster rolls Attack Dice (1–10 d6). Each 6 = 1 Hit.
  2. Player rolls Defence Dice (1–8 d6). Each 6 cancels 1 Hit. Unblocked Hits subtract from Player Health.
  3. Player rolls Attack Dice (1–5 d6). Each 6 = 1 Hit.
  4. Monster rolls Defence Dice (0 for all, 9 for Dracular). Each 6 cancels. Any surviving Hit kills the monster.
- **Monster HP is always 1.** A single unblocked hit ends any non-Dracular encounter. Dracular is also 1 HP but hides behind a 9-die defensive wall.
- **Player HP is 17, hard-capped.** Potions can top it back up but cannot exceed 17.
- **No fleeing, no interrupts, no abilities, no criticals, no status effects.** Once combat starts it runs to a corpse.
- **Solo only.** No PvP, no co-op. Opposition is purely monster dice vs. player dice.
- **Monsters appear individually, one per room.** There are no mobs, no adds, no linked encounters.
- **Deterministic once rolled.** The player cannot modify or re-roll dice; their only pre-combat agency is the gear they brought in.

**Texture of a typical fight:** Most encounters against AD 1–4 monsters end in one or two rounds with the player taking 0 or 1 damage. Encounters against AD 7–10 monsters are a scary few rounds where the monster's pool outpaces the player's defence pool and multiple hits leak through. The Dracular fight stretches 6–12 rounds because the boss *blocks* almost everything the player throws.

**The characteristic feel of the dice:** Bursty. Swingy. "Often nothing, sometimes wipe." Because the success threshold is a single pip (6 on d6), the distribution of hits per round is a binomial that is heavily skewed left — a 5-die pool rolls zero hits **40%** of the time and rolls 3+ hits only **3.5%** of the time. The player spends a lot of time not taking damage, punctuated by rare "the demon rolled four 6s and I'm dead" spikes.

---

## 2. The 1-HP Problem

**Problem:** In the source, every monster except Dracular dies to a single unblocked hit. A naive real-time translation — "enemy dies the moment your sword touches them" — produces an experience where monsters burst like balloons in 0.4 seconds. There is no dance, no combat, no expression. The moment-to-moment feels cheap.

### Options considered

| Option | Pro | Con | Faithful? |
|---|---|---|---|
| **Give enemies 3–5 HP** | Easiest, makes fights last longer | Breaks the source's "hits are lethal" identity | No |
| **Keep 1 HP + positional setup (Sekiro posture)** | Preserves lethality, adds skill expression | Complex to implement; high learning cost | High |
| **Keep 1 HP + hits are HARD to land (Dark Souls parry gate)** | Preserves lethality, the fight is about *creating* the opening | Requires enemies to have strong blocking/telegraphs | High |
| **Monsters appear in groups (3–6 at once)** | Fight duration scales naturally, 1-HP becomes a feature (power fantasy) | Breaks the "one monster per room" source rule | Medium |

### Pick: **Hybrid of (Sekiro posture / Dark Souls parry-gate) + selective grouping**

**Core decision: Keep monster HP = 1. Make the hit itself the hard part.**

Every non-boss monster has a **Guard** — a short animated posture cycle during which incoming hits do nothing. The player cannot brute-force kill the monster. They must wait for or create a **Windowed Opening**:

- **Passive opening:** The monster telegraphs an attack. During the wind-up or recovery frame of that telegraph, its Guard drops for ~0.4s. A connecting hit in this window = kill.
- **Active opening:** The player **parries** a monster swing (Dead Cells / Sekiro deflect). A successful parry stuns the monster for ~0.5s and its Guard drops. A connecting hit in this window = kill.
- **The player's Attack Dice pool controls the "hit radius" / swing arc**, NOT the damage number (which is always "kill on connect" during an opening). More Attack Dice = a bigger, faster, more forgiving swing that is more likely to actually land on a moving target during the window.

**Selective grouping:** Low-level trash (Orcs, Wolves, Skeletons) can appear in pairs or triples in deeper levels as a secondary occupancy roll — this preserves the room-based structure while giving the player satisfying "two-tap three enemies in a combo" moments. Mid-tier and above remain solo encounters.

**Why this preserves faithfulness:** The source rule "one unblocked hit kills" survives literally. The player still kills every normal monster with a single connecting strike. What changes is that "connecting" now requires timing and positioning rather than just rolling dice.

---

## 3. The 17-HP Wall

**Problem:** The source player has 17 HP against attackers dealing 0.03 – 1.5 damage per round. In a real-time port, the question is: what does "Health = 17" mean when the player is dodging in real time?

### Options considered

| Option | Feel | Readability | Preserves source? |
|---|---|---|---|
| Real HP bar, hits = numerical damage | Diablo / ARPG | High | Yes (directly) |
| Hearts system (Zelda) | Discrete, stylized | Very high | Yes, maps 1:1 |
| Stamina + health hybrid | Souls-like | Medium | No, adds new resource |
| One-hit-kill with lives | Hotline Miami | Very high | No, loses the 17-HP budget |

### Pick: **17 Hearts (discrete pips), Zelda / Hades hybrid**

**The player has a horizontal row of 17 heart pips** at the top-left of the HUD. Every incoming hit subtracts exactly 1 pip. Potions refill exactly 1 pip each. The heart cap is hard at 17 — potions cannot overfill.

**Why hearts over a bar:**
- The source is an *integer* game. 17 HP is 17 discrete states. A bar would require inventing fractional damage rules or arbitrarily discretizing back to integers; hearts preserve the 1:1 mapping natively.
- Hearts are **readable at a glance** during action combat. Player can count remaining health in one frame.
- The 17 number is distinctive and becomes an identifying visual ("oh this is the 17-hearts game"), similar to Zelda's 3-heart start.
- Potions already map cleanly: 1 Potion = 1 Pip. The player sees a potion drop and the math is trivial.

**Scaling damage per monster:** Not every enemy hit removes a full pip. This would trivialize low-level monsters and make 10-AD Demons round-zero executioners.

Instead, each **Monster Attack Die** in the source becomes a **Hit Chance roll at connection time**. When a monster's telegraph lands on the player:
- 1 AD monster = 1 pip consumed (flat)
- 2–3 AD = 1 pip (these are speed/combo threats, not damage threats)
- 4–6 AD = 1 pip but attack pattern is harder to avoid (more sweeps, longer reach)
- 7–8 AD = 2 pips per clean hit (these hits matter)
- 9–10 AD = 2 pips + can stack a DoT bleed (Demon)
- Dracular = 2 pips per hit with a 3-pip "charged" special

This keeps 17 HP meaningful across the whole run — a late-game Demon can still 9-shot a max-health player.

---

## 4. Dice → Probability Translation

**The source's signature feel is the count-6s d6 pool.** It is bursty, swingy, often-miss-sometimes-massive. A 5 AD roll returns 0 hits 40% of the time and 3+ hits 3.5% of the time. This heavy left skew is the *personality* of the combat.

### Options considered

| Option | Preserves source | Fits real-time | Risk |
|---|---|---|---|
| Keep dice visible as side panel, tie to swings | High | Medium | Reads as "dice minigame next to action" |
| Hidden probability roll on every attack | Medium | High | Invisible randomness feels bad in RT |
| Rhythm-based timing attacks | Low | High | Loses the dice-pool identity |
| Pure deterministic hit-if-connected | Low | Very high | Loses the bursty feel entirely |

### Pick: **Visible dice ghost panel tied to swings — but probability applies to MULTI-HIT spray, not binary hit/miss**

This needs unpacking. Here is the model:

**On every player attack swing:**
1. Play the swing animation.
2. On contact with an enemy, roll the player's Attack Dice (5 dice for a max-gear player) in a **visible 3D physics dice tray** in the lower-right HUD corner. The dice tumble for ~0.5s and settle.
3. Count 6s. If **0 hits** rolled: the swing visually connects but produces a **Guard Spark** effect (clang, spark, no damage, enemy staggers slightly — "your blade glanced off").
4. If **1+ hits** rolled: the first 6 **kills** the enemy instantly (if in an Opening window). Extra 6s cause a **cleave** effect — the sword swings through and lethally hits nearby enemies (this is where grouped-trash moments shine).
5. If the attack is NOT during an Opening window, even a successful 6 is converted into a Guard Spark (parry-gate is upstream of the roll).

**On every monster attack that connects:**
1. Monster plays attack telegraph.
2. If the player did not dodge/parry, monster rolls its Attack Dice in an enemy-side dice tray.
3. Count 6s. **0 hits** = the blow glances off / sparks the player's shield (no pip loss, small knockback). **1+ hits** = pip loss, with the per-AD scaling from §3.
4. Defence Dice, if the player has them, **subtract from the hit count** via a simultaneous defensive dice roll visible on the player-side tray. This is the source's "player rolls defence" happening in the same tick as the monster attack roll.

**Why this works:**
- The dice are **visible, tactile, and diegetic**. The player sees the bursty rhythm with their own eyes — dice clatter, show 1s, show 1s, show 1s, then *boom* three 6s and an enemy explodes. This is the source's "often-miss-sometimes-massive" directly preserved.
- The dice do **not gate binary hit/miss** — positional play / parry gates that. Dice decide **magnitude of the result** (no hit → spark, one hit → clean kill, many hits → cleave).
- The dice tray sits at the screen edge and is **glanceable**, not demanding focus. Like a skill bar in an ARPG.
- The bursty-swingy math is literally the source math. No re-weighting. The "1/6 per die" is sacred.

**Reference touchstones:** Dicey Dungeons for the visible dice, Hades for the side-HUD attention budget, Dead Cells for the parry-gate → guaranteed-hit flow.

---

## 5. Combat Resolution Pattern

**Source pattern:** Monster attacks → player defends → player attacks → monster defends → repeat.

### Options considered

| Pattern | Fit | Preserves "monster first"? |
|---|---|---|
| Parry-riposte (Sekiro / Dead Cells) | Excellent | Yes naturally |
| Dodge-roll + punish windows (Dark Souls) | Good | Yes |
| Exchange rounds (turn-based-in-real-time) | Mediocre | Yes but stiff |
| Action-sim (both hit continuously) | Poor | No |

### **Pick: Parry-Riposte with Dodge-Roll Fallback (Dead Cells / Sekiro hybrid)**

**The core combat loop in real time:**

1. **Encounter trigger.** Player enters a room. Monster is already positioned in the room and **starts its attack animation before the player's attack is ready** — this enforces the source's "monster always attacks first" rule. The player literally cannot swing during the first 0.4s of an encounter because their character is drawing their weapon / entering combat stance.
2. **Monster attacks.** Monster telegraphs (a visible wind-up, 0.3–0.8s depending on monster type). The player has three options during the telegraph:
   - **Parry** (tight window, ~0.15s before contact) → triggers Opening, player gets a guaranteed next-swing kill window.
   - **Dodge-roll** (i-frames, ~0.3s) → avoids damage, does NOT trigger Opening. Monster recovers and the cycle continues.
   - **Block** (hold button, drains a posture meter) → passive defence. The defence-dice roll is consumed here; if the dice cancel the incoming hits, block is free. If they don't, block posture breaks and player is staggered.
3. **Player attacks.** During the Opening window (from parry or from monster post-attack recovery), the player swings. This is the moment the **attack dice pool rolls** (§4). A connecting hit = kill.
4. **Monster defends (Dracular only).** When the player's swing connects on Dracular, Dracular's 9 defence dice roll in real time. This almost always reduces the hit count to 0 → spark → the player must disengage and try again. The "monster defends" step becomes the boss's signature feel.
5. **Repeat** until monster dies or player hearts = 0.

**How "monster first" is preserved:**
The monster is *always* the aggressor. The player's first meaningful action in an encounter is always **reactive** (parry / dodge / block). The player cannot walk into a room and one-shot a monster by swinging first — their swing has a draw animation that takes longer than the monster's telegraph. This is a direct, diegetic translation of "monster always attacks first" into frame data.

**Reference touchstones:**
- **Dead Cells:** parry-into-guaranteed-kill is already proven to feel great against 1-HP style enemies.
- **Hades:** the dodge-roll recovery window as a punish window.
- **Sekiro:** "make the hit itself the hard part" via posture / guard deflect.
- **Hyper Light Drifter:** the way rooms open into combat with enemies already aggro'd before the player can react.

---

## 6. Per-Monster Feel

All 11 monsters must feel distinct even though the source only gives them one stat: AD count. In the RT version each gets a signature silhouette, attack pattern, and tell.

| # | Monster | AD | Size | RT Identity |
|---|---|---|---|---|
| 1 | **Orc** | 1 | Medium | Slow, shoulder-charge lunge. **Tell:** raises club overhead, telegraph ~0.8s. The tutorial enemy — a generous parry window designed to teach the system. |
| 2 | **Wolf** | 2 | Small | Fast, leaps in arcs. **Tell:** crouches for 0.3s before pouncing. Movement-based threat — teaches dodge-roll over parry. |
| 3 | **Skeleton** | 3 | Medium | Jerky, twitchy, reassembles after being knocked down (visual bait — it still dies on an opening hit, but the first swing of a new encounter only staggers). **Tell:** rattles before a triple-swipe. |
| 4 | **Evil Warrior** | 4 | Medium | A mirror of the player: sword + shield, parries *your* swings. **Tell:** lowers stance before a thrust. Teaches the player to bait-and-punish rather than spam. |
| 5 | **Devil Bat** | 5 | Small, airborne | Flies in figure-eights, dives to bite. **Tell:** a screech and a sudden altitude drop. Only vulnerable during the dive. Teaches vertical spatial awareness. |
| 6 | **Cyclops** | 6 | Huge | Slow, massive club sweeps with large hitboxes. **Tell:** both hands on the club, shoulder-turn wind-up ~1.2s. Opening is long but the sweeps cover half the arena. Arena-control threat. |
| 7 | **Dark Elf** | 7 | Small | Ranged — throws poisoned darts in 3-dart fans. Teleports short distances. **Tell:** glows purple before teleport, narrows eyes before volley. First real "reads" check. |
| 8 | **Skeleton Lord** | 8 | Large | Summons 2 skeleton adds (finally — grouped combat!) during the fight. Main body is armoured. **Tell:** raises staff to summon (add-phase), swings halberd in horizontal arcs (damage-phase). The first "phase-aware" enemy. |
| 9 | **Wizard** | 9 | Medium | Pure ranged, floats. Casts three different projectile types (straight bolt, homing orb, AoE zone). **Tell:** each spell has a distinct colour and vocalisation. Must be punished between cast animations — high parry risk because he doesn't melee. |
| 10 | **Demon** | 10 | Massive | The pre-boss terror. Fire breath cone + claw swipes + can grab and deal 3 pips. **Tell:** inhales (fire incoming), lifts claw (sweep incoming), lunges forward (grab incoming). Three-tell monster — the player must learn all three to survive. |
| 11 | **Dracular** | 9 AD / 9 DD | Tall, imposing | Multi-phase boss. See §7. |

Each monster also has a **signature audio cue** heard on room entry so the player knows what they're facing before they even see the enemy — a growl for the Orc, a screech for the Bat, a chant for the Wizard, organ notes for Dracular.

---

## 7. Dracular as Climax

Dracular is the only fight in the game that the source **explicitly designs as hard**. In the source, the boss's 9/9 stat line produces a ~55–60% win rate even for max-gear players. In RT, the fight needs to *feel* like a climax, not just statistically be one.

### Arena

A **gothic throne hall**, long and narrow. Three zones:
- **Entry balcony** (where the player enters)
- **Central combat floor** (open, with destructible pillars)
- **Throne dais** at the far end (where Dracular starts)

The arena has **environmental hazards** that activate per phase: chandeliers that fall, stained-glass moonlight beams that the player can knock Dracular into for bonus damage, coffins along the walls that Dracular retreats into during Phase 3 to heal.

### Phases (3, mapped to Dracular's 9 "rounds" of combat)

**Phase 1 — "The Count" (100% → 66%)**
- Dracular floats down from the throne, melee-focused.
- Attacks: a cape-sweep (3-pip horizontal AoE), a bite lunge (2 pips but fast recovery), a dark-magic palm blast (ranged, 1 pip).
- Defence: 9 DD is represented as a **crimson aura** that visibly sparks away most of the player's hits. Most swings produce spark effects, not kills.
- **Key mechanic:** parry his bite lunge to force him into a stagger where his aura drops for 1.5s — the player's only reliable damage window.
- **1 kill hit in Phase 1 knocks him to Phase 2.** (The "1 HP" rule literally applies — the fight isn't three health bars, it's three *forms*.)

**Phase 2 — "The Bat" (66% → 33%)**
- Dracular explodes into a swarm of bats, then reforms as a larger bat-form.
- Attacks: diving swoops (must dodge), bat-swarm zone denial, a bite grab that lifts the player and drops them for 2 pips.
- Defence aura is now a **swirling bat storm** that redirects player swings. The player must hit him between swoops — the aura drops at the apex of each dive as he turns.
- **Key mechanic:** the environmental stained-glass beams now matter. Dracular takes double hitbox in moonlight. Positioning him in a beam is the fastest kill.
- **1 kill hit in Phase 2 knocks him to Phase 3.**

**Phase 3 — "The Lord" (33% → 0%)**
- Dracular retreats to the throne and becomes a **two-avatar fight**: a shadow clone runs around the arena while the real Dracular casts from the throne.
- Attacks: the clone mirrors Phase 1 moves. The real Dracular at the throne channels a cross-arena lightning wave every 8 seconds (must be dodged or will one-shot via 3-pip damage).
- **Key mechanic:** the clone's aura is permanently down. Killing the clone does nothing — but the clone's death triggers the real Dracular's aura to drop for 2 seconds. The player must sprint the length of the arena and deliver the killing blow in that window.
- **1 kill hit on the real Dracular in Phase 3 = victory.**

### How "9 AD / 9 DD" translates

- **9 AD** becomes Dracular's **ability to roll more dice per attack than any other enemy** — his attack rolls in the RT dice tray spawn 9 dice clattering, and the player's heart row shudders dramatically as the count ticks up. This makes every unparried Dracular hit feel catastrophic, even when the dice roll badly.
- **9 DD** becomes his **crimson aura** — a visual wall that the player sees sparking off every non-opening swing. The aura is the reason the fight is long even though Dracular is "1 HP." Without an opening, the player cannot land a kill.

### Fight duration

Target: **90–180 seconds** total, across all 3 phases.
- Phase 1: 30–60s (depends on how fast the player learns the bite-parry).
- Phase 2: 30–60s (depends on arena positioning).
- Phase 3: 30–60s (depends on clone timing reads).

Deaths restart the *fight*, not the whole run — unless the chosen run-mode is hardcore, see §11.

### The 2–3 key tells (learnability)

1. **The bite lunge in Phase 1** — this is the "gateway" parry. The player has to learn it or Phase 1 is unwinnable. It has a clear red flash on Dracular's eyes 0.3s before contact.
2. **The swoop apex in Phase 2** — the bat-form's aura drops for 0.4s at the top of every dive arc. Visible because Dracular turns his head / wings fold back.
3. **The shadow clone death → aura drop in Phase 3** — a screen-wide audio cue (a cathedral bell toll) marks the 2-second window. The player has to *move* during that window, not just swing. This is the climactic skill check of the whole game.

---

## 8. Feel Problems

Real-time translation of a dice game introduces specific feel hazards. Each needs mitigation.

### FP-1: "I died to a 10-AD random roll with no reaction time" — UNFAIR

**Problem:** In the source, a Demon can roll 5+ hits in one round and one-shot the player. In RT, if we preserve this literally, the player gets hit once and their heart row empties with no way to react. That's not punishing, that's punishment without play.

**Mitigation:**
- **Cap per-hit damage at 3 pips** regardless of underlying dice count. Extra "hits" above 3 are converted into **knockback** and **screen shake** intensity. A catastrophic demon roll becomes a dramatic launch + massive screen effect rather than instant death.
- **Pity heart:** if the player takes damage that would drop them from >5 pips directly to 0, they instead survive at 1 pip with a brief invulnerability frame (Hades trick). This only triggers once per encounter and has a long cooldown (once per 3 monster rooms).
- **Damage is telegraphed before it's dealt** — even against a monster with high AD, the damage is resolved on an *animated* swing that the player had a chance to react to. If they failed the reaction, they chose to eat it.

### FP-2: "Monsters with 1 HP dying instantly feels ANTICLIMACTIC"

**Mitigation:** Already addressed in §2. Kills require an Opening, not just a connection. Additionally:
- **Kill flourish:** every kill triggers a 0.1s hit-stop, a satisfying crunch audio, and a pixel-burst particle effect. Even a tiny Orc feels like an event.
- **Execution animations:** on Opening-kills the camera briefly zooms and the player character performs a short execution animation (0.3s). This makes even 1-HP kills feel cinematic.

### FP-3: "No interrupt-ability means no skill expression"

**Problem:** The source combat is 100% deterministic after the dice roll — the player has no mid-combat agency. In RT, this would mean the player just watches a cutscene of dice. Unacceptable.

**Mitigation:** Skill expression is moved to the **reaction layer**, which the source simply doesn't have:
- **Parry timing** — skill-expressive, high ceiling.
- **Dodge-roll i-frame timing** — skill-expressive.
- **Positional awareness** (stained-glass beams in boss, cleave alignment against groups) — skill-expressive.
- **Gear-build-to-arena matchups** — the loadout choice matters against specific monster types.

This creates a new skill axis that doesn't exist in the source but which the source also doesn't *forbid*.

### FP-4: "The shop interrupting between fights disrupts action flow"

**Problem:** The source puts the shop at step 7 of every turn. In RT, pausing the action every 45 seconds to open a shop menu kills momentum.

**Mitigation:**
- **Shop lives at persistent "Safe Rooms"** that appear every 2–3 combat rooms (not every room). The player walks into a lit room, a shopkeeper character is there, and interaction is optional. The action loop only breaks when the player *chooses* to engage.
- **In-combat upgrades are disabled** — the player cannot shop during a fight. This preserves the source's "step 7 is after combat" constraint.
- **Safe-room ambient audio** (fire crackling, merchant humming) provides an emotional beat of rest. Darkest Dungeon town visits are a good reference — the rest beat is a *feature*, not a flaw.

### FP-5: "The player has no mid-combat healing but can die before they reach the next shop"

**Problem:** The source allows potion use only at step 6 of a turn (and only for potions not bought that turn). A literal RT port would mean the player can't heal mid-fight, but RT fights are lethal enough to need it.

**Mitigation:** Potions are usable **in combat**, bound to a quick-use hotkey, with a **0.4s drinking animation** during which the player is vulnerable (Dark Souls Estus). This preserves the "healing costs a window of vulnerability" tradeoff while giving the player the agency to make the decision mid-fight.

### FP-6: "Dice dice dice everywhere, the screen is cluttered"

**Problem:** If every swing rolls dice in a visible tray, the HUD becomes a slot machine.

**Mitigation:**
- The dice tray **only appears during an active Opening or active monster attack**. When the player is just moving / positioning, it's invisible.
- **Audio-first signalling:** a fast "clack clack clack six!" audio sting plays when a 6 rolls, even if the player isn't looking at the tray. The dice become a heard rhythm, not a watched spectacle.

---

## 9. Damage Feedback — The Hit Impact Recipe

Every combat hit (player-on-monster and monster-on-player) must feel weighty. The recipe below is non-negotiable for the project's game feel.

### Player hits monster (kill)

| Layer | Effect | Duration / Intensity |
|---|---|---|
| **Hit-stop** | Frame-freeze on contact | 0.08–0.12s (scales with monster size) |
| **Camera shake** | Small perpendicular shake | Amplitude 0.15, duration 0.15s |
| **Slow-mo** | Time scale 0.3× | 0.1s (bigger kill = longer) |
| **Particles** | Monster-type-specific (blood for flesh, bone-shards for skeletons, ichor for demons, embers for wizard) | 80 particles, dispersed radially |
| **Knockback** | Enemy ragdolls backward | 3m distance, scales with player AD |
| **Audio stinger** | Crunch + impact + death-cry | Single mixed cue, ducks ambient |
| **Damage numbers** | **NO.** This is a 1-HP game. The death *is* the number. | N/A |
| **Screen edge flash** | White radial vignette from hit point | 0.1s fade |
| **Dice tray flash** | The 6 that landed the kill pulses gold in the tray | 0.3s pulse |

### Player hits monster (spark — no Opening)

| Layer | Effect |
|---|---|
| Hit-stop | 0.03s (much shorter, conveys "it didn't count") |
| Particles | Metal sparks (yellow-white) |
| Audio | Sharp "clang" |
| Monster reaction | Tiny stagger, no damage |
| No screen shake, no slow-mo | (conveys "this wasn't a real hit") |

This difference between **kill feel** and **spark feel** is the clearest possible signal to the player: "you need an Opening, that swing didn't count, try again."

### Monster hits player

| Layer | Effect |
|---|---|
| **Hit-stop** | 0.1s |
| **Camera shake** | Larger, radial, amplitude 0.3, duration 0.25s |
| **Controller rumble** | Strong pulse (if gamepad) |
| **Screen edge** | Red blood vignette from hit direction, 0.5s fade |
| **Heart pip** | The lost pip shatters visibly with a "break glass" effect on the HUD |
| **Audio** | Impact thud + player grunt + low-frequency rumble |
| **Time scale** | 0.5× for 0.15s (helps the player process what happened) |
| **Knockback** | 1–2m backward, scales with monster AD |
| **Damage numbers** | **Small red number** above the player briefly ("–1" or "–2"). Unlike kill hits, here the number matters because damage is variable. |

### Parry (successful deflect)

| Layer | Effect |
|---|---|
| **Hit-stop** | 0.2s (longest in the game — the parry is the hero moment) |
| **Camera shake** | None (the world stops) |
| **Screen flash** | White full-screen flash, 0.05s |
| **Slow-mo** | 0.2× for 0.3s |
| **Audio** | Cathedral bell "TING" + monster hiss |
| **Monster visual** | Frozen in stagger pose, red "!" above head |
| **Dice tray** | Pulses purple to indicate Opening window is active |
| **UI** | "PARRY" text briefly in top-center, fades in 0.4s |

The parry is the most important player input in the whole game. The feedback must be extravagant.

---

## 10. PvP Opportunity

**Verdict: NO PvP.**

### Why this game refuses PvP

1. **The source is militantly solo.** There are no rules for a second player, no opposing decisions, no information asymmetry. Adding PvP would be inventing a fundamentally different game.
2. **The conflict model is PvE by identity.** The enemy is "the dungeon" — random rolls + monster types + the self-blocking pathfinding constraint. The tension is the player vs. their own luck and the Hamiltonian walk. There is no natural seat for a human opponent.
3. **The 1-HP asymmetry would be poisonous in PvP.** Who has 1 HP? Both? Both of them die in 0.3s the instant they open a parry window — unplayable. One of them? Then it's not symmetric, and designing the "monster role" against a skilled human who now has intent and strategy would require reinventing the game from scratch.
4. **The source's progression is run-based, not match-based.** There is no matchmaking unit. A "game" is 20–40 minutes of escalating tension against growing dice pools. PvP would need to compress or abstract that, losing the slow climb that is the identity of the experience.
5. **The dice burst rhythm breaks under adversarial pressure.** In a real-time PvP context, a 40%-chance-of-zero-hits swing means the loser of a coin flip is just unlucky, not outplayed. This feels bad in PvP (where fairness expectations are high), even though it feels *great* in PvE (where randomness IS the tension).

### What we will offer instead of PvP

- **Asynchronous ghost runs:** daily-seed runs where other players' paths and deaths appear as spectral overlays on the dungeon grid. You can see where other heroes fell. This is social without being competitive.
- **Leaderboards:** time-to-Dracular, monsters slain, no-hit-boss attempts, hearts remaining at victory. Competitive but not synchronous.
- **Shared cursed runs:** a weekly "cursed dungeon" with a fixed seed and modifier where everyone plays the same nightmare and scores are compared.

**PvP is not just absent, it is philosophically incompatible with this translation.**

---

## 11. Difficulty Through Reaction

The standard real-time combat model scales difficulty with reaction time demands: enemies get faster, windows get tighter, players with twitch win. This excludes cool-headed players, older players, and players with motor accessibility needs. The source Solo Dungeon Bash is explicitly a solo-and-cerebral game — it would be wrong to lose that audience.

### How we balance difficulty so cool-headed players can play

**1. Tell-duration settings (not difficulty settings)**

The game has a **Tell Duration Slider** in options: 1.0× (default), 1.25×, 1.5×, 2.0×.
- At 1.0×, Orcs telegraph for 0.8s. At 2.0×, they telegraph for 1.6s.
- Parry windows scale with this slider too.
- Dodge i-frames get a matching extension.

This is NOT called "Easy Mode." It's called "Reaction Comfort." It does not reduce damage, does not remove enemies, does not change the dice — it simply buys the player more time to *read* the fight.

**2. The Turn Button — a time-bending toggle**

Hold a trigger button: time slows to 30%. Stamina drains while held. Release: time resumes.

This is **a direct digital translation of the source game's "read the situation and decide" pacing**. In the board game, the player has *infinite* time to look at the roll and decide what to do. In RT, the Turn Button gives them a slice of that cognitive comfort back. Skilled players will use it sparingly for tight parries; cool-headed players will use it as their primary combat tool.

**3. Dice-Preview Mode**

An accessibility option: the attack-dice roll result is shown as a *prediction* 0.3s before it's committed, giving the player a chance to abort (spend stamina) or commit. Cool-headed players can use this to plan; twitch players will ignore it.

**4. Read-first, act-second enemy design**

Every enemy's attack pattern is **pattern-readable**, not **reflex-only**. No enemy has unannounced instant attacks (the Demon's fire breath has a 1.2s wind-up, not a 0.2s flash-fry). The game rewards **pattern recognition**, which scales with patience not reflexes.

**5. Pause-to-shop, pause-to-potion menus**

Any menu interaction is a **full time pause**, not a real-time menu. Mid-fight potion drinking has a 0.4s animation but menuing to the potion is instant. This removes UI-dexterity penalty from the reaction layer.

**6. Runs are forgiving of retries**

The source game is harsh — die and the run is over. The RT translation offers a **"Gentle Run" mode** as a baseline: death at a monster restarts you at the nearest Safe Room, not the start of the dungeon. A **"Bash Run" mode** is the hardcore source-faithful option where death = full reset. Both modes are valid; neither is the "real" one.

### Core principle

**Difficulty is expressed in patience, not in APM.** A careful player who uses long tell durations and the Turn Button should be able to clear the game. A twitchy player who ignores all those aids should be able to clear it faster and feel clever. Neither experience is the "right" one.

---

## 12. Flagged Translation Risks

Mechanics where the translation is genuinely uncertain and needs playtesting to resolve.

### TR-1: Dice visibility cognitive load

**Risk:** The visible dice tray might be a great feature OR it might be a distraction. It's impossible to know without playtest.
- **Test:** A/B two builds — one with dice tray, one with audio-only dice stings (no tray). Measure which version players report feeling more engaged with.
- **Resolution trigger:** >70% preference for one side in internal playtests → commit to that.
- **Fallback:** if neither wins, make the tray a toggleable option ("Hide Combat Dice" accessibility).

### TR-2: Parry window length

**Risk:** 0.15s is "Dead Cells tight." It might be too tight for the source's cerebral-player audience, or it might be perfect because the Tell Duration slider compensates. Unknown until players play.
- **Test:** instrument parry attempts, measure success rates by monster type and tell-duration setting.
- **Target:** a cool-headed player on 1.5× should hit parries 60–70%. A twitchy player on 1.0× should hit 50%+.

### TR-3: Hearts vs. bar

**Risk:** The hearts model may run out of screen space. 17 pips is a *lot* of pips for a HUD.
- **Test:** prototype the HUD at 1080p / 4K / ultrawide, confirm readability.
- **Fallback:** split into rows (10 + 7) or use heart containers (Zelda-style) where each "container" = 2 pips.

### TR-4: 1-pip-per-hit vs. 2-pip-per-hit for high-AD monsters

**Risk:** The §3 proposal to give Demon / Dracular 2-pip hits might make those fights feel unfair instead of climactic. Unclear until players fight them.
- **Test:** telemetry on deaths to each monster type. If Demon kill-rate is >40% on first attempt, it's probably too punishing.
- **Resolution:** tune pip-per-hit scalar until target death distributions are met.

### TR-5: Selective grouping of trash mobs

**Risk:** The source is explicit: "one monster per room." Putting groups of 2–3 Orcs in late rooms might break faithfulness in the eyes of source purists. Or it might be fine because no one remembers that rule.
- **Test:** show both versions to board-game-familiar playtesters, ask which feels more like SDB.
- **Fallback:** guard the grouping behind a "Pack Mode" toggle that defaults to off for "Faithful" runs.

### TR-6: Shop pacing in Safe Rooms

**Risk:** Putting the shop in safe rooms (not every combat room) changes the source's 7-step turn rhythm. Might disrupt the economy or might barely matter.
- **Test:** telemetry on treasure accumulated per safe room, potion usage per combat, death-due-to-no-heals rate.
- **Resolution:** tune safe room frequency (every 2 rooms vs. every 3 vs. every 4).

### TR-7: Monster-first initiative felt as "unfair"

**Risk:** The source rule "monster always attacks first" is translated as "player cannot swing during the first 0.4s of a room." If players read this as "my controls are locked," it will feel bad.
- **Test:** observe whether playtesters complain about "input delay" when entering rooms.
- **Fallback options:**
  1. Make the "draw weapon" animation cancellable into a parry (but not into an attack) so the player feels reactive, not locked.
  2. Give the player a "burst" window: entering a room triggers a 0.3s invulnerability + a stance-ready frame, so the monster-first rule is invisible to the player.

### TR-8: Dracular's Phase 3 clone mechanic readability

**Risk:** The "kill the clone → 2-second window → sprint to real boss" mechanic is a specific and high-ceiling idea. It might be incomprehensible in playtest.
- **Test:** first-time playtests. Measure % of players who figure out the mechanic on their first Phase 3 attempt.
- **Target:** >50% figure it out first-try. <50% means the tells are insufficient.
- **Fallback:** add an explicit tutorial prompt ("The clone is shielding the real Dracular!") the first time it's triggered.

### TR-9: Count-6s math versus real-time tolerances

**Risk:** The bursty "0 hits 40% of the time" rhythm is awesome in theory but in practice a real-time swing that does nothing 40% of the time might just feel like the game is broken. The source's rhythm is tolerated because dice are *visibly* random; if the game feels like its hit detection is broken, players won't care about the math.
- **Test:** compare two versions — one with literal source probabilities, one with a **floor** (first hit of an encounter is always guaranteed, subsequent dice are random).
- **Resolution:** probably needs a mild floor (~10–15% effective buff) to survive the "player expects swings to do something" instinct, unless the dice tray visibility is so strong that players internalize the randomness.

### TR-10: The 17-HP scale mismatch

**Risk:** 17 pips is a *lot* of hearts. On paper it translates cleanly, but a player seeing 17 hearts and seeing 1 disappear may not register the loss as significant. The feel might be "my bar of 17 went to 16, who cares."
- **Test:** measure player reported anxiety during and after low-HP moments. If low-HP anxiety is absent, the translation failed.
- **Fallback:** collapse to 10 hearts where each heart = 2 pips (the Zelda model). This changes the source number but may feel better.

---

## Conclusion — The Resolution Stance

The RT translation of Solo Dungeon Bash combat keeps **three sacred elements** from the source:

1. **Hits are lethal** — every kill is one hit. No enemy HP bars.
2. **Bursty dice** — the 1/6 success rhythm is visible and audible, directly preserved via the dice tray.
3. **Monster-first initiative** — the player is always reacting, never opening.

It adds **one new layer** the source did not have:

4. **Reactive skill expression** — parry, dodge, and positional play. This is the interpretation of "real-time" that makes the translation viable. Without it, the game is a dice-watcher. With it, the dice are a texture the player navigates.

The combat resolution pattern is **parry-riposte with dodge-roll fallback (Dead Cells / Sekiro hybrid)**, justified by the source's "monster always attacks first" rule mapping cleanly onto reactive combat: the monster's aggression is the event, the player's parry or dodge is the response, the dice decide the magnitude.

Everything else — the gear shop, the grid, the potions, the 17-HP cap, the encounter tables — remains structurally identical to the source. Only **the combat moment** is reinterpreted, because the combat moment is the only thing that *must* be reinterpreted to survive the switch to real time.

---

*End of ConflictModel.md — RealTimeForge RT-4 for Solo Dungeon Bash.*
