# Economy Model — Solo Dungeon Bash (Real-Time Translation)

> **Stage:** RT-5 Progression & Economy Translation
> **Source:** `BalanceSheet.md`, `BalanceSheet.csv`, `RulesExtraction.md`, `Mechanics.md`
> **Target format:** Real-time action roguelike (top-down, 2D)

This document translates Solo Dungeon Bash's turn-based, roll-and-write economy into a real-time progression system. The source is an extremely tight binary win/lose experience with three stockpilable resources, nine permanent upgrades, and no catch-up or metaprogression. Our job is to preserve its signature tensions — compounding upgrades, cap-limited healing, forced commitment — while giving the player the moment-to-moment feedback a real-time game demands.

---

## 1. Source Economy Diagram

```
               +----------------------------+
               |      DUNGEON ROOMS         |
               |  (90 rooms, 10 levels,     |
               |   1d6 content per room)    |
               +------+---------------+-----+
                      |               |
         17% TREASURE |    17% POTION (L1/3/6/9 only)
                      |               |
                      v               v
            +------------------+   +-----------------+
            |    TREASURE      |   |    POTIONS      |
            |   (unbounded)    |   |  (unbounded)    |
            +---------+--------+   +--------+--------+
                      |                     |
                      v                     v
            +------------------+   +-----------------+
            |    STEP-7 SHOP   |   |  STEP-6 HEAL    |
            |  (9 items, 3     |   |  1 Potion = +1  |
            |   exclusive      |   |  HP, capped 17  |
            |   slots)         |   +--------+--------+
            +---------+--------+            |
                      |                     |
                      v                     v
            +------------------+   +-----------------+
            |  STAT UPGRADES   |   |    HEALTH       |
            | (+AD / +DD,      |   |   (0..17)       |
            |  permanent)      |   +--------+--------+
            +---------+--------+            ^
                      |                     |
                      v                     |
            +------------------+            |
            |   COMBAT WIN     |------------+
            |   PROBABILITY    |  (losses feed back
            |   (compounds)    |   as HP drain)
            +---------+--------+
                      |
                      v
            +------------------+
            |  DEEPER ROOMS    |----(loops back to DUNGEON ROOMS)
            |   EXPLORED       |
            +------------------+
```

Key properties of the source loop:
- **Monotonic upgrades.** Every purchase is permanent; there is no degradation.
- **Two faucets, one sink.** Treasure (wide, 17% everywhere) and Potions (narrow, only 4 levels) both drain eventually into combat survival.
- **HP cap is the economic governor.** Without the 17-HP cap, hoarded potions would trivialize risk. The cap is why pushing deep to spend HP "in advance" of regeneration matters.
- **Compounding.** Early treasure is worth more than late treasure because upgrades apply to all subsequent combats. This is the engine-building heart of the game.

---

## 2. Resource Translation Table

| Source Resource | RT Equivalent | Regen / Gain Rate | Spend | Cap |
|---|---|---|---|---|
| **Health (17 cap)** | **Health (170 HP cap)** — scaled 10× for finer real-time grain | No passive regen. Gained via potion items or fountains. | Damage from enemy attacks, traps, boss specials. | 170 HP hard cap (same 17 "tick" count, with 10-HP granularity per old point). |
| **Treasure (unbounded)** | **Gold** — auto-pickup coin drops from kills + chests | Drops at ~1 gold per monster killed on average, ~3 gold per chest, ~6 gold per elite. Target 45-60 gold per 30-min run. | Permanent upgrade altars between rooms. | Unbounded (source-faithful). |
| **Potions (unbounded, 1 turn delay)** | **Potion Vials** — slot-based consumable bar (max 6 carried) | Dropped from rooms (~1 per 3-4 rooms in "potion zones"), or bought from altars (buys have a **20s arming cooldown** = the source's "next turn" rule). | Hotkey quick-heal for +10 HP each (1-source-HP ≡ 10 RT-HP). | **6 in inventory** (new cap, to prevent degenerate hoarding). |
| **Attack Dice (1-5)** | **Attack Power tier (1-5)** — determines damage/sec, not literal dice | Gained only via altar purchases (Big Sword, Big Axe, Spiky Armour). | Never spent — permanent. | Hard cap at 5 (1 base + 2 Big Axe + 2 Spiky Armour). |
| **Defence Dice (1-7)** | **Defence Tier (1-7)** — damage-reduction percent + iframe window | Gained only via altar purchases (Buckler, Shield, Spiky Armour, Magical Armour). | Never spent — permanent. | Hard cap at 7 (1 base + 2 Shield + 5 Magical Armour by A-3 stacking ruling, OR 1 + 1 Spiky + 5 Magical = 7 alternate path). |

**Dice-to-damage mapping (replaces "count 6s on d6"):**
- **1 AD** → 5 DPS baseline
- **2 AD** → 10 DPS
- **3 AD** → 18 DPS (Big Sword)
- **4 AD** → 28 DPS (Big Axe or Spiky Armour only)
- **5 AD** → 42 DPS (Big Axe + Spiky Armour — max build)

DPS scales super-linearly to reflect that in the source game, each +1 AD roughly halves expected time-to-kill. A 1-HP monster in the source that took ~6 rounds at 1 AD now takes ~3 seconds at 1 AD, ~0.4 seconds at 5 AD.

**Defence-to-mitigation mapping:**
- **1 DD** → 10% damage reduction
- **2 DD** → 18% (Buckler)
- **3 DD** → 25% (Shield)
- **4 DD** → 32% (Shield + Spiky OR Buckler + Spiky)
- **5 DD** → 38%
- **6 DD** → 45% (Magical Armour)
- **7 DD** → 52% (max defensive build)

Diminishing returns baked in so that max DD is survivable but not invulnerable — mirroring the source, where even max-gear players only hit Dracular ~55-60% of the time.

---

## 3. Economy Mode

**Selected: Continuous generation + burst spends** (RTS-style economy).

**Justification:** The source game has exactly this shape. Treasure drips in as a 17% trickle on every room, and potions drip in on a narrower channel, but the *spending* happens in discrete, deliberate bursts at the step-7 shop. The player is always accumulating (drip) and always deciding when to commit a big chunk (burst). A Hades-style "bursty after fights" economy would collapse the source's farming-vs-pushing tension, and a survival-game "slow-and-deliberate" feel would betray the rapid-kill energy of 1-HP monsters. RTS continuous-drip-into-burst-spend captures both the faucet tension and the upgrade discipline.

Rejected alternatives:
- **Slow-and-deliberate** — Source monsters die in ~1 hit. Nothing slow about it.
- **Fast-and-constant** — Would erase the shop deliberation moment entirely.
- **Bursty** — Would remove the "one treasure room every 6 rooms" rhythm that defines the farming loop.

---

## 4. Treasure in RT

**Collection:** Auto-pickup coin drops on enemy kill and on chest destruction. We do *not* use manual collection — it would fight the flow of an action game. Treasure rooms in the source become **treasure chests embedded in rooms** (visible on entry, smashable for 2-5 gold). Monster-kill drops are smaller (1-2 gold) and exist to keep the drip-faucet running between explicit treasure spots.

- **Auto-pickup radius:** 3 tiles. Coins magnet to the player.
- **Visual:** Gold coins spill outward with small arc; a satisfying "ting" per pickup.
- **Level treasure density:** Matches source's 17% flat rate. In a ~60-room run we expect ~10-12 treasure events.

**Spending:** **Permanent altar stations** placed at the entrance/exit of every 3-4 rooms. Altars present the current purchasable items from the 9-item shop; approaching the altar triggers a **soft pause** (game slows to 10% speed, not full pause — preserves tension) and opens a radial selector. Purchase is instant, stat applies immediately, gold deducted.

- Altars are **always-available** (source's step-7 shop appears every turn).
- All 9 shop items are present at every altar. The player just needs gold.
- Bought potions go into inventory but are **greyed out for 20 seconds** (the "next turn" delay).
- No backtracking shop — a run has enough altars that you never need to retrace.

---

## 5. Potions in RT

**Quick-heal hotkey.** The player has a dedicated button (Q on keyboard, LB on controller) that drinks one potion for +10 HP (source's +1 HP × 10). The potion bar is visible in the HUD, top-right corner, as a vertical stack of up to 6 vials.

- **Use conditions:** Usable at **any time except during boss telegraphs** (to prevent panic-chug exploits in telegraphed moments). Using a potion triggers a 0.5-second drinking animation during which the player is still mobile but cannot attack.
- **Drop rule:** Found potions go into inventory immediately and are usable next frame. **Bought** potions are greyed out for 20 seconds (this is the source's "next turn" delay, translated into wall-clock). The grey-out is visually clear: purple vials with a clock overlay and a countdown ring.
- **HP cap interaction:** Drinking a potion while at 170/170 HP is **blocked** — the game will refuse to consume the potion and flash a "HP full" prompt. This preserves the source rule "Health may not exceed 17."
- **Found potions** override the cooldown rule — rare potion-room pickups (levels 1, 3, 6, 9 equivalents) are instant-usable.

Design rationale: the source treats potions as a manual-use, player-controlled resource ("take any or all Potions you've collected"). A hotkey is the cleanest RT analog. Auto-healing would strip agency; drag-onto-character is fussy for a roguelike tempo.

---

## 6. The Permanent Upgrade Ladder

**Selected approach:** Upgrades stream continuously as rewards (via altars between rooms) but **commit immediately** — no end-of-encounter delay.

- **Source parallel:** In the source, upgrades happen at "step 7" which is end-of-turn, immediately effective for the next turn. In the RT version, altars mid-dungeon are the equivalent. The player walks up, spends gold, stat applies instantly, continues.
- **Slot exclusivity is preserved.** If you own a Big Sword and purchase a Big Axe, the game shows a confirmation dialog: "Replace Big Sword with Big Axe? (refund 3 gold for Big Sword)". This resolves source ambiguity F-3 — we allow sidegrade with partial refund, since locking the player into a bad early choice is punishing without adding tension.
- **Armour slot stacking:** We adopt A-3 literal (shields DO stack with armour). So a max build = Big Axe + Shield + Magical Armour (2+0+5 AD = 2 AD extra, 0+2+5 DD = 7 DD extra) OR Big Axe + Spiky Armour (2+2 AD, 0+1 DD). Spiky is the all-in offensive option; Magical is the all-in defensive option.

Frequency: **1 altar every ~90 seconds** of play (so roughly 20 altars in a 30-minute run). Not every altar is a meaningful purchase — early altars exist for 1-gold Bucklers and bulk-potion deals; late altars are where the big weapon/armour commits happen.

---

## 7. Item Slot System

The player's character has **three visible equipment slots**, shown as icons in the HUD bottom-left:

| Slot | Options | Visual |
|---|---|---|
| **Weapon** (Exclusive) | Basic Fists / Big Sword / Big Axe | Held weapon changes on-screen; swing animation differs. |
| **Shield** (Exclusive) | None / Buckler / Shield | Small off-hand shield prop; affects dodge-roll visual. |
| **Armour** (Exclusive) | Cloth / Spiky Armour / Magical Armour | Full character skin swap (spikes visible / glowing runes visible). |

**Presentation:**
- Each slot is a circle in the HUD; filled circles show equipped icon; greyed circles show empty.
- **On-character visuals change in real-time.** When the player buys Big Axe, the sword in their hand morphs mid-stride into an axe. When they buy Magical Armour, the armour snaps on with a particle burst. This is load-bearing for feedback.
- **Switching rule:** A player can swap to a new item in an exclusive slot, and the game **refunds half** the old item's cost (rounded down). This rewards experimentation without enabling free-rebuild farming.
  - Refund example: Own Big Sword (3g). Buy Big Axe (4g). Total spent: 4g. Refunded: 1g (floor of 3/2). Net spent: 3g for the upgrade.
- **Commit vs. freedom:** We land on "mostly committed, with costed undo." Pure commit (source-faithful) is punishing in RT because the player often realizes mid-dungeon that their playstyle wants speed or tankiness. Pure free-swap trivializes the choice. Half-refund is the compromise.

---

## 8. The Shop Interface

**Selected: Ambient altars in every room cluster** (Dead Cells style, not Hades between-room style).

**Justification:**
- The source's shop "appears every turn" — it's always there, always available. Hades-style "between rooms only" would make the player feel constrained. Dead Cells-style ambient stalls feel closer to the source's rhythm.
- The player should never have to backtrack through cleared rooms to find an altar. We guarantee altar density of **~1 every 3 rooms** procedurally.
- **Altar interaction:** Walk up → press E → world slows to 10% (not full pause; enemies still approach but slowly, so you can't shop mid-combat) → radial menu of 9 items → select → buy → world returns to 100%.
- **Visual:** A stone shrine with a floating icon showing the altar's "featured item" for flair. All 9 items are buyable at every altar — the featured item is cosmetic variety only.

**Rejected:**
- **Hades-style gated:** Too restrictive; would break the source's drip-purchase rhythm.
- **Dropped loot chests:** Would remove the decisive player agency the source grants via the shop.
- **End-of-dungeon-level altars (1 per row):** Would starve players early; source has shop access every turn.

---

## 9. Victory Condition Translation

**Selected: Hybrid — Pure binary win/lose as the primary outcome, with a "clear time" and "rooms explored" readout as flavor.**

- **Primary:** You either kill Dracular or you don't. There is no ranking, no score leaderboard on the main menu.
- **Post-run screen** shows:
  - **Outcome:** Victory / Death / Blocked
  - **Clear time:** mm:ss
  - **Rooms explored:** N
  - **Gold collected:** N
  - **Items purchased:** icon list
  - **Final HP:** N/170
- These stats are **informational**, not competitive. They exist so the player can compare runs to themselves and feel progression without warping strategy toward score optimization.

**Justification:**
- Source is strictly binary; adding a true score (time attack, combo) would warp the game away from its "push deep, commit, kill the boss" core. It would create pressure to skip rooms in a way the source never does.
- But in a real-time game, some measurable feedback on the post-run screen is essential for replayability and player satisfaction. Time and rooms-explored are neutral flavor — they describe the run without incentivizing degenerate play.
- **No leaderboards. No rating.** This is faithful to the source's "you won" or "you died" identity.

---

## 10. Match Length Design

**Selected: Single continuous session, ~30 minutes, death resets, no saves mid-run.**

- **Target length:** 28-32 minutes for a competent player, 20-45 minutes on outer tails. Matches source's 20-40 min estimate.
- **No mid-run save.** This is a roguelike — a run is a single commitment. If the player closes the game mid-run, the run is lost. (We may add a "resume in-progress run" for the current session only, cleared on quit, as a QoL against crashes.)
- **Death resets.** No continues, no revives. Faithful to source — death is the end. Players who want another run press "New Run" from the main menu.
- **No chapter breaks.** The 10 dungeon levels translate into 10 vertical "floors," but they are **seamless** — the camera pans up as the player ascends, no loading screens. This preserves the single-sitting intensity.
- **Timer-bound? No.** There is no run timer. Players can take as long as they want within a run. This matches the source's "no tempo or time limit" identity.

**Justification:** Roguelikes live on single-session commitment. Allowing saves or continues would drain the tension that makes a binary win/lose matter. 30 minutes is short enough to replay frequently and long enough for a real upgrade arc.

---

## 11. Match End Triggers

A run ends when **any** of the following is true:

| Trigger | Source-faithful? | Notes |
|---|---|---|
| **Victory: kill Dracular** | Yes | Player reaches End zone, defeats Dracular, credits roll, return to menu. |
| **Death: HP hits 0** | Yes | Dracular gibs the player; screen fades red; "You Died" stinger. |
| **Blocked: End unreachable** | Yes | If the procedural dungeon evolves in a way that ever traps the player (see section 14), the game detects it after ~10 seconds of no progress and triggers a "Dungeon Collapse" loss. |
| **Retreat / give up** | Added for RT | ESC → "Abandon Run" button in pause menu. Confirms twice. Counts as a loss. Exists for player agency and player mental health — a 30-min stuck run should have an exit. |
| **Time limit** | **Not added** | The source has no timer and we preserve that. Adding a time limit would reshape player behavior entirely. |

Of the four source loss conditions (HP 0, blocked, maze trap, intentional stop), all are faithfully carried over. Victory is the boss kill, unchanged.

---

## 12. Catch-up Mechanics

**Selected: None — faithful to source.**

This is a hard opinion. The source has zero catch-up and is *designed* around the consequence of early losses. The death spiral is the game. Adding comeback potions or rubber-banded enemies would undermine the core identity.

**Counter-arguments we considered and rejected:**
- **"Real-time games need catch-up because skill variance is higher."** True in competitive RT games, but this is a solo PvE experience. Variance is learning, not matchmaking.
- **"A bad early run feels unfair."** It feels unfair *because it is unfair*. That's the source's point. Every roll matters; every room entry matters. Catch-up dilutes every decision.
- **"But 30 minutes is a long time to waste on a dead run."** Mitigated by the retreat option (section 11) and by the short-enough runtime. A 30-min failure is acceptable; a 30-min failure where you also feel robbed by comeback mechanics is worse.

**What we DO keep as "implicit catch-up":**
- **Altars are always available.** A player on their last leg can still buy potions at any altar (assuming they have gold). This is source-faithful — the shop is available every turn regardless of HP.
- **The potion faucet exists.** If the RNG is kind, a floundering player can find potions in levels 1/3/6/9. This is luck, not rubber-banding.
- **The retreat option.** Players choose to end a dead run rather than grind through it.

That's it. The game is punishing. Lean into it.

---

## 13. Metaprogression

**Selected: Bestiary / cosmetic unlocks only. No stat-based metaprogression.**

- **Bestiary:** Each monster type (Orc, Wolf, Skeleton, Evil Warrior, Devil Bat, Cyclops, Dark Elf, Skeleton Lord, Wizard, Demon, Dracular) has a bestiary entry. First-time kills unlock lore text and the monster's stat card. Zero mechanical impact.
- **Cosmetic unlocks:** Completing milestones (first win, first 10-room run, first altar purchase, etc.) unlocks **character skins** and **weapon trail colors**. All purely visual.
- **What we explicitly REJECT:**
  - **Permanent stat bonuses between runs (Hades-style).** This would invalidate the source's "you start weak, you buy your way up" engine. A run should always begin at 1 AD / 1 DD / 17 HP.
  - **Unlockable starting items.** Same reason.
  - **Unlockable difficulty modes.** Fine as a menu option, but not as a progression gate.
  - **Currency that persists between runs.** There is one currency, gold, and it vanishes on death.

**Justification:** The source has *no* between-run progression and is a complete experience without it. Our RT version adds cosmetic hooks (because roguelikes live and die on return-for-unlocks psychology) but refuses to dilute the core loop. A win feels like a win *because* it's earned fresh every time. Hades-style metaprogression would shift the difficulty curve across runs in a way that betrays the source's pure binary identity.

The bestiary gives the player a reason to see Dark Elves and Skeleton Lords and Wizards, even on losing runs. Cosmetic skins give long-term reward. Both are zero-stakes and faithful.

---

## 14. Degenerate Strategies to Prevent

Based on the BalanceSheet red flags and the RT translation surface, the following exploits are likely:

### Exploit A: Treasure farming by never committing to boss
- **Risk:** Player stays on lower floors indefinitely, racking up gold and max gear, then walks to Dracular with perfect build. The source naturally prevents this with the no-revisit rule and blocking loss condition (if you spiral too hard, you lose). In RT, we need an analog.
- **Counter-mechanic:** **Dungeon decay.** After ~10 minutes in a single floor zone, enemies on that floor begin to respawn with +20% HP and +10% damage, stacking per minute. This is *not* rubber-banding (which responds to player weakness); it's **anti-farming pressure** (which responds to player stalling). It preserves the source's "you must push forward" vibe.
- **Secondary:** **Treasure chest depletion.** A given room has finite treasure chests. Once looted, they don't respawn. Monster drops still scale per kill, but the big chest payouts are one-shot. This matches the source rule that each room is rolled once and never revisited.

### Exploit B: Potion hoarding trivializes combat
- **Risk:** Buy 6 potions for 3 gold (6-pack), spam-heal through any boss. The source guards against this with the "next turn" delay. In RT, we add the 20-second arming delay and the **6-potion cap**.
- **Counter-mechanic 1:** **Potion inventory cap: 6.** This is new; the source is unbounded. 6 is enough to survive a bad sequence of rooms but not enough to win a boss fight on healing alone.
- **Counter-mechanic 2:** **Potion lockout during boss telegraphs.** Dracular's big-wind-up attacks (cast bars) disable potion drinking for the duration of the telegraph. Panic-chug is a skill loss, not a lifesaver.
- **Counter-mechanic 3:** **HP cap enforced.** Drinking at 170/170 is blocked. No "preload before boss" because the cap exists.

### Exploit C: Dominant Magical Armour rush
- **Risk:** Rush to 6 gold, buy Magical Armour, ignore offense, turtle through everything. The BalanceSheet flags this as Red Flag #3 — Magical Armour strictly dominates Shield per-die.
- **Counter-mechanic:** **Kill-time DPS thresholds.** Certain rooms (elite encounters around floor 5+) have monsters that **regenerate HP** if not killed within 8 seconds. A no-offense build literally cannot clear these rooms. This forces every build to maintain *some* attack power. Pure defensive turtle builds lose to regen.
- **Additional:** The final boss Dracular has a **defense phase** where he becomes briefly invulnerable and summons adds; you need to kill the adds on a timer or be overwhelmed. Low-DPS builds fail this phase.

### Exploit D: Altar refund abuse
- **Risk:** Player buys Shield for 2g, gets to next altar, buys Buckler for 1g, refund 1g from Shield. Now owns Buckler, lost 0g net, spent 2g on a mistake. Scales if player farms refunds.
- **Counter-mechanic:** **Refund is one-time per slot.** Once you sidegrade a slot, further sidegrades give **no refund** at all. This lets players fix one mistake per slot but prevents refund farming.
- **Additional:** **Refund is half (rounded down).** Already priced in.

### Exploit E: Corner-camping at altars
- **Risk:** Because the world only slows to 10% (not 0%) at altars, a player could park at an altar and use it as a safe shopping zone.
- **Counter-mechanic:** **Altar radius is tiny.** You must be within 2 tiles of the altar for the slow effect. Any enemy can walk into range. Also: altars have a **one-purchase-per-visit** limit — after one transaction, the altar's slow effect ends and the altar goes on a 30-second cooldown.

### Exploit F: Infinite backtracking
- **Risk:** Unlike the source's no-revisit rule, an RT dungeon might let the player walk back into cleared rooms freely, circumventing push-forward tension.
- **Counter-mechanic:** **Collapsing corridors.** Each floor, once the player ascends to the next, seals behind them (with a brief dramatic tremor). No going back. This is source-faithful — the no-revisit rule forbids re-entering a cleared square.

---

## 15. Scoring System

We chose "no true score" in section 9. The post-run screen shows:

- **Clear time** (informational, not ranked)
- **Rooms explored** (informational)
- **Gold collected** (informational)
- **Items purchased** (icon list)
- **Final HP** (informational)

There is **no formula combining these into a score**. There is no leaderboard. The "gold star" for the player is the Victory/Death text and nothing else.

**Play-decision implications:** Because there is no score optimizer, the player's incentives are always "reach the boss alive with enough power." There is no pressure to skip content, no pressure to speedrun, no pressure to collect max gold. The player explores exactly as much as they feel they need — matching the source's "no tempo or time limit" identity.

If a future update adds a scoring system (e.g., weekly seeded runs), the recommended formula would be:

```
Score = (Victory ? 10000 : 0)
      + max(0, 3000 - clear_time_seconds * 2)
      + rooms_explored * 20
      + gold_unspent_at_end * 5
      - potions_used * 10
```

This formula rewards winning, penalizes slowness, and slightly rewards efficiency (unspent gold, fewer potions used). But it is **not part of the base game**. We flag it here as a possible future extension only.

---

## 16. Resource Rhythms per Minute

Over a target 30-minute run, the economy should flow as follows. These numbers are derived from the source's expected-value analysis in BalanceSheet.md and adapted to real-time pacing.

### Room throughput
- **Target rooms cleared:** ~60 rooms in 30 minutes → **2 rooms per minute average**.
- **Per-room time:** ~30 seconds average (fast empty rooms ~10s, combat rooms ~20-45s, treasure/potion pickups ~15s, altar visits ~25s).
- **Source comparison:** source runs ~30 turns in 20-40 minutes = 0.75-1.5 turns/min. RT is faster (2 rooms/min) because there's no paper bookkeeping.

### Gold flow
- **Gold per room (expected):** 0.8 (17% chest × 3 gold + ~60% monster × 1 gold drop ≈ 0.51 + 0.6 ≈ 1.1, round down).
  Let's re-check: 17% chance of treasure chest at 3g = 0.51 expected from chests. 60% chance a room has a monster averaging 1g = 0.60. Total ≈ 1.1 gold/room.
- **Total gold per 30-min run:** ~60 rooms × 1.1 = **~66 gold**. Source expectation was ~5-6 treasure over 30 turns; we 10× scale (since RT gold is 10× finer).
- **Gold per minute:** ~2.2 gold/min.
- **Source sanity check:** 5-6 source treasure ÷ 30 turns = 0.17 treasure/turn → 0.34 treasure/min at 2 rooms/min → 3.4 RT gold/min. Our ~2.2 is slightly stingier; we err toward source-tight.

### Potion flow
- **Potions per run from rooms:** Source gets ~2 found potions per run (1/6 on 4 rows). In RT, we place equivalent "potion altar rooms" on floors 1, 3, 6, 9 with ~15-17% chance per room on those floors. Expected found potions per run: **2-3**.
- **Potions per run from purchases:** Players typically buy 1-2 bulk potion packs (the 6-pack is 3 gold and efficient). Expected purchased potions per run: **~6-9**.
- **Total potion draws per run:** ~9-12 per 30 minutes. Player uses maybe 6-8 of those (roughly 60-80% efficiency).
- **Potions per minute:** ~0.3 found + ~0.2 bought = 0.5 potion events/min. Usage: ~0.2-0.3 drinks/min.

### Stat upgrades (altar purchases)
- **Altars visited per run:** ~20 (1 every 90 seconds).
- **Altars with a meaningful purchase:** ~6-8 (most altars the player just walks by with no gold).
- **Upgrade milestones per run:**
  - Minute 3-5: First Buckler (1g) or first Shield (2g).
  - Minute 8-12: First weapon upgrade (Big Sword 3g or save for Big Axe 4g).
  - Minute 15-20: Weapon commitment and armour path begins.
  - Minute 22-28: Final armour piece (Magical Armour 6g or Spiky Armour 5g).
  - Minute 28-30: Boss fight.
- **Total stat upgrade purchases per run:** **4-6** out of the 9 shop items. Source is same order (~5-6 items bought per winning run).

### Kills
- **Total monster kills per run:** ~40-50 monsters. Source: ~60% of 30 turns = 18 monster rooms at 1 monster each → 18 kills. RT is higher because we have 2 rooms/min and more monsters per room.
- **Kills per minute:** ~1.5 kill/min.

### Budget summary

| Metric | Per 30-min run | Per minute |
|---|---|---|
| Rooms cleared | ~60 | 2 |
| Gold collected | ~66 | 2.2 |
| Gold spent | ~60 | 2.0 |
| Potions found | ~2-3 | 0.1 |
| Potions bought | ~6-9 | 0.25 |
| Potions consumed | ~6-8 | 0.22 |
| Altar interactions | ~20 | 0.67 |
| Stat-upgrade purchases | 4-6 | 0.15 |
| Monster kills | ~40-50 | 1.5 |
| HP lost to combat (gross) | ~180 HP | 6 HP/min |
| HP regained from potions | ~70 HP | 2.3 HP/min |
| Net HP loss over run | ~110 HP | (starts 170, ends ~60 going into boss) |
| Boss fight HP budget | 60 HP available | ~15-20s fight |

**Reading the budget:** The player spends most of their gold (60 of ~66), buys 4-6 items, loses more HP than they regain (source-faithful — you arrive at the boss wounded), and has roughly 60 HP going into the Dracular fight. With max gear, Dracular does ~3-4 HP/second (after mitigation), so the fight lasts ~15-20 seconds and is genuinely a coin-flip. This mirrors the source's 55-60% win-rate-on-max-gear finding.

---

## 17. Flagged Issues

### Issue 1: The 17-HP cap translation grain
**Concern:** Source HP is 17 integer. RT HP is 170. This 10× scaling lets us show damage numbers like "7 damage" for what was "0.7 source HP." But if we ever want to show HP as "17/17" for nostalgia, the translation breaks. **Recommendation:** Commit to 170/170 RT-native display. Do not show source-scale HP.

### Issue 2: Potion timing — the "next turn" rule
**Concern:** The source's "bought potions usable next turn" rule is clean in turn-based but awkward in RT. We chose a 20-second arming delay. This may feel arbitrary to players ("why can't I drink this?"). **Recommendation:** Make the delay visually obvious (clock overlay on greyed vial) and add a sound cue on the moment the potion becomes ready. Consider shrinking to 15s if playtests show it feels punitive.

### Issue 3: Self-blocking pathfinding (M10) has no RT analog
**Concern:** The source's "you can paint yourself into a corner" is a turn-based puzzle constraint. In RT, players move continuously, so "blocked from reaching End" doesn't trigger naturally. **Recommendation:** We replaced it with **collapsing corridors** (floor seals behind you) and **dungeon decay** (anti-farming). Neither is the same as the source constraint, and players who loved the pathfinding puzzle in the source will miss it. Flagged as a significant departure from source feel.

### Issue 4: No-score-ness may disappoint modern roguelike players
**Concern:** Modern roguelikes (Hades, Dead Cells, Binding of Isaac, Slay the Spire) all have some form of post-run scoring or metaprogression. Our "bestiary + cosmetics only" may feel thin to players expecting more reward loops. **Recommendation:** Monitor playtest sentiment. If retention drops, the safest addition is **more bestiary depth** (lore pages, monster-specific quests) rather than stat-based metaprogression.

### Issue 5: Altar refund system is new ground
**Concern:** The source does not allow selling items (ambiguity F-3 in RulesExtraction.md). Our half-refund-one-time-per-slot system is a designer-made choice. **Recommendation:** Playtest two variants — (A) half refund, (B) no refund at all — to see which feels better. If (B) is acceptable, it's more source-faithful.

### Issue 6: Shield/Buckler slot economics
**Concern:** BalanceSheet red flag — Shield strictly dominates Buckler per-die. In source, this is OK because Buckler is the early-affordable option. In RT, with altars every 90 seconds, the player might skip Buckler entirely and save for Shield. That makes Buckler effectively dead content. **Recommendation:** Accept this — Buckler fills the "1-gold emergency purchase" slot, which is a valid role even if not optimal. Alternatively, buff Buckler to +1 DD and 5% movement speed to give it a unique identity.

### Issue 7: Dungeon decay may feel punitive
**Concern:** The anti-farming decay system (section 14) adds +20% monster HP per minute of stalling. Players who don't realize this exists will feel cheated when rooms get harder. **Recommendation:** Show a visible indicator — after 5 minutes on the same floor, a skull icon appears in the HUD; after 10 minutes, it pulses red and a warning appears: "The dungeon is stirring." Telegraph clearly.

### Issue 8: The 6-potion cap is new
**Concern:** Source has unbounded potions. We added a 6-cap to prevent hoarding exploits. This changes strategy — the 6-pack purchase (3 gold for 6 potions) is suddenly "buy only when your current stack is low." **Recommendation:** The cap is load-bearing for balance. Accept the deviation. Alternative: raise cap to 8 if playtest shows 6 is too tight.

### Issue 9: The boss fight length
**Concern:** Dracular in source takes multiple rounds of d6 rolls. In RT, our DPS math says a max-gear fight is ~15-20 seconds. That might feel short for a "final boss." **Recommendation:** Give Dracular phase transitions — at 66% HP and 33% HP, he enters invulnerable phases and summons adds. This extends the fight to ~45-60 seconds and creates multi-phase boss spectacle. Phase math must be tuned so total HP remains equivalent to source's 9-DD tanking.

### Issue 10: Altar density may overwhelm
**Concern:** 20 altars per 30-minute run = altar every 90 seconds. Some players may feel spammed with shop prompts. **Recommendation:** Differentiate altar tiers visually. Tier-1 altars (small stone shrines) appear every ~45 seconds and only sell potions and Buckler/Shield. Tier-2 altars (glowing shrines) appear every ~3 minutes and sell everything including the big-ticket weapons/armour. This reduces decision fatigue on minor purchases.

---

## Summary of Key Design Choices

| Decision | Choice | Faithful? |
|---|---|---|
| Economy mode | Continuous drip + burst spend (RTS-style) | Yes |
| Treasure collection | Auto-pickup coins + smashable chests | Adaptation |
| Treasure spend | Ambient altars every 3 rooms, soft-pause | Adaptation |
| Potions | Quick-heal hotkey, 6-cap, 20s arming on buys | Mostly (cap is new) |
| Upgrades | Continuous stream, immediate effect | Yes |
| Item slots | 3 exclusive visible slots, half-refund-once | Mostly (refund is new) |
| Shop location | Ambient in-world altars (Dead Cells style) | Adaptation |
| Victory | Binary win/lose, post-run stats (no score) | Yes |
| Match length | ~30 min, single session, no saves, no continues | Yes |
| End triggers | Victory, death, blocked, retreat | Yes |
| Catch-up mechanics | **None** | Yes |
| Metaprogression | **Bestiary + cosmetics only** | Yes |
| Degenerate strategy prevention | Dungeon decay, potion lockout, kill-time thresholds | Adaptation |

**Design thesis:** The source game is a complete, balanced, punishing solo experience. Our RT translation preserves the punishing binary outcome, the compounding upgrades, and the faucet-and-burst economic rhythm. Where we deviate — potion cap, half-refund, altar density, collapsing corridors, dungeon decay — it is always to translate a turn-based mechanic into RT-legible feedback, never to soften the difficulty. Players who win a run will feel they earned it; players who lose will know why.
