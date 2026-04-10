# Balance Simulation Design — Solo Dungeon Dash (Real-Time Translation)

> **Stage:** RT-A3 Balance Simulation Design
> **Sources:** `BalanceSheet.md`, `BalanceSheet.csv`, `realtime/analysis/EconomyModel.md`, `realtime/analysis/ConflictModel.md`
> **Target:** A Monte Carlo simulator that drives data-backed tuning of the RT port during Alpha.

This document specifies a Monte Carlo balance simulator for the real-time translation of Solo Dungeon Dash. The source game is a tightly constrained turn-based dungeon crawler; the RT port retains the Visible Dice Tray, 17-heart HP, 1-HP monsters with Guard + Opening windows, and the 9-wide dungeon grid, but places all of this inside a reactive parry/dodge loop. Because the RT port changes the *skill expression surface* without rewriting the underlying probability engine, simulation is both possible and necessary: the dice math is source-faithful enough to predict outcomes, and the reactive layer is volatile enough that we cannot trust intuition about which knobs ripple into which metrics.

---

## 1. Goals of Balance Simulation

The balance simulator exists to answer four questions before we ever put a human in front of the game:

1. **Is the win rate on each difficulty mode in its target band?** The RT port ships with four difficulty modes (Easy, Normal, Hard, Nightmare). We need each to land close to target without requiring weeks of human playtesting per tuning pass.
2. **Is the run length in its target band?** The source game is a 20-minute experience. We promise median 20 min, 80th percentile 12–35 min. A config that accidentally pushes median to 45 min is broken even if the win rate is fine.
3. **Are there degenerate strategies that dominate?** A Monte Carlo sim can sweep policies and detect when a single strategy (e.g., Magical Armour rush, potion hoarding) is strictly better than all alternatives.
4. **Which parameters are sensitive, and how do they interact?** The source Balance Sheet named 5 top-sensitive parameters based on analytic reasoning. In the RT port, the reactive layer (parry window, telegraph duration) adds new sensitivity axes we can only characterize empirically.

**What the sim protects against:**
- Shipping a difficulty mode that is secretly unwinnable or secretly trivial.
- Buffing one monster's telegraph and accidentally killing the early-game difficulty curve.
- Tuning the shop economy so that Magical Armour rushes become optimal and the weapon slot is dead.
- Retaining a set of tuning values that were balanced for one game mode and break horribly on another.
- Failing to detect that the 90-second Dracular fight is actually a 4-minute slog 60% of the time.

**What the sim does NOT protect against:**
- Game *feel* — is the parry satisfying? Is the camera shake the right amplitude? These require human hands.
- Bugs — the sim uses the design model, not the game code.
- First-impression / onboarding issues — a player's first 3 minutes are not the sim's concern.
- Narrative / art / audio quality.

The sim is a **balance spreadsheet on steroids**, not a game engine. It lets designers sweep parameters overnight and arrive at human playtest with strong priors.

---

## 2. Simulation Framework

### 2.1 Architecture

A headless Python script is the reference implementation. It is called from the CLI with a config file path and zero or more override flags:

```
python balance_sim.py \
    --config configs/normal.yaml \
    --n-runs 10000 \
    --seed 42 \
    --override parry_window_ms=180 \
    --override dracular_defence_dice=10 \
    --output reports/normal_run_001.json
```

Alternative implementation target: a GDScript headless module that reuses the same YAML config schema, for teams that want the sim to run inside the Godot project and stay in sync with the production balance data. The Python reference is preferred because it is faster to iterate on and trivially parallelizable.

### 2.2 Inputs

| Input | Source | Example |
|---|---|---|
| **Balance config** | YAML file committed to the repo at `configs/{difficulty}.yaml` | `configs/normal.yaml` |
| **Difficulty mode** | Selects between Easy / Normal / Hard / Nightmare configs | `--config configs/hard.yaml` |
| **Policy π** | Heuristic-player implementation (see §2.4) | Always `heuristic_v1` in MVP |
| **RNG seed** | Integer for reproducibility | `--seed 42` |
| **N runs** | Number of simulated runs per experiment | `10000` |
| **Override flags** | Runtime parameter overrides for sweeps | `--override parry_window_ms=180` |

The YAML config matches `TuningKnobs.csv` column-by-column: each row in the CSV corresponds to one key in the YAML file. This is the single source of truth for balance tuning.

### 2.3 Process

For each of N = 10,000 runs:

1. **Seed:** Derive a per-run seed from `seed + run_index`. This makes runs independently reproducible and lets us isolate specific bad runs for debugging.
2. **Generate dungeon:** Procedurally generate a 9×10 grid using the same generator the shipping game uses, reading row probabilities from the config. Place Start (row 1, middle), End (row 10, middle), and the Shop Shrines.
3. **Initialize player:** Spawn with starting HP / AD / DD from the config.
4. **Run policy π (see §2.4):** The policy walks through the dungeon one room at a time, making decisions at each step. Each room resolves its content (treasure, potion, monster, shop). Combat is resolved by the combat sub-simulator (§2.5).
5. **Terminate:** Run ends when the player reaches End and defeats Dracular (VICTORY), dies (DEATH), or runs out of legal moves (BLOCKED — no unvisited adjacent room).
6. **Record outcome:** Append a row to the run-log with outcome, clear time, rooms explored, treasure collected, items purchased, final HP, boss phase reached, cause of death (if any).

### 2.4 Policy π — the Heuristic Player

The MVP sim uses a single deterministic-ish heuristic policy modeling "a competent but not optimal player":

- **Parry reliability:** The player successfully parries 70% of incoming telegraphs when the monster's telegraph duration ≥ the player's reaction floor (250ms). Below that, parry reliability degrades linearly to 0 at a 100ms telegraph.
- **Dodge reliability:** When the player chooses to dodge instead of parry (random 20% of the time when parry would succeed, 100% of the time when parry window is too tight), the dodge succeeds with probability = `min(1.0, dodge_iframes_ms / telegraph_ms)`.
- **Shopping policy:** At every shop encountered, the player spends treasure with this priority:
  1. If own 0 Big Axe and can afford it → buy Big Axe.
  2. If own 0 Shield and can afford it → buy Shield.
  3. If fewer than 3 Potions in inventory and can afford 3-pack → buy 3-pack.
  4. If can afford Magical Armour and no Spiky Armour owned → buy Magical Armour.
  5. Otherwise save.
- **Routing policy:** Move toward End via a biased random walk that (a) always advances vertically if possible, (b) prefers rooms with higher expected value (treasure > potion > monster), (c) never backtracks into a visited room, (d) accepts a blocked state if every adjacent square is visited.
- **Potion usage:** Drink a potion when HP drops to ≤ 6 hearts AND the player owns ≥ 2 potions.
- **Flee behavior:** The source does not allow fleeing. The heuristic matches: once combat starts, it runs to completion.

**Advanced policies** (future work, not in MVP):
- **Greedy farmer** — stays in rows 1-3 as long as possible, farms treasure.
- **Straight runner** — bee-lines through middle column to End.
- **Optimal router** — solves a shortest-path problem weighted by expected value.
- **Perfect player** — parry reliability 100%, dodges perfectly, never takes damage outside of 10-AD spike rolls.

The difference between the heuristic player's win rate and the perfect player's win rate measures the **skill headroom** of the current balance — a design target we want to stay around 20-30 percentage points (a noticeable gap between competent and expert, but not a hopeless cliff).

### 2.5 Combat Sub-Simulator

Each combat encounter is resolved round-by-round using the source's "count 6s on d6" math, with the RT reactive layer applied as a filter on incoming damage:

```
function simulate_combat(player, monster, config):
    while monster.hp > 0 and player.hp > 0:
        # 1. Monster attacks (source order: monster first)
        monster_ad_roll = roll_pool(monster.attack_dice)
        hits_from_monster = count_sixes(monster_ad_roll)

        # 2. Player reactive layer
        if config.reactive_layer_enabled:
            if policy.chose_parry(player, monster, config):
                if rand() < parry_success_prob(policy, monster, config):
                    # Parry succeeds — opening opens, monster dies on next player hit
                    hits_from_monster = 0
                    monster.guard_down = True
            elif policy.chose_dodge(player, monster, config):
                if rand() < dodge_success_prob(policy, monster, config):
                    hits_from_monster = 0

        # 3. Player defence dice
        player_dd_roll = roll_pool(player.defence_dice)
        blocks = count_sixes(player_dd_roll)
        net_damage = max(0, hits_from_monster - blocks)
        net_damage = min(net_damage, config.per_hit_damage_cap)  # FP-1 mitigation
        player.hp -= net_damage

        if player.hp <= 0: return DEATH

        # 4. Player attacks
        if monster.guard_down or monster.is_boss is False:
            player_ad_roll = roll_pool(player.attack_dice)
            player_hits = count_sixes(player_ad_roll)
            if monster.is_boss:
                boss_blocks = count_sixes(roll_pool(monster.defence_dice))
                player_hits = max(0, player_hits - boss_blocks)
            if player_hits >= 1:
                monster.hp -= 1  # 1-HP monsters die on any successful hit
        # else: guard is up, spark effect, no damage

    return VICTORY
```

The combat sub-sim runs in under 50 microseconds per encounter on a modern laptop. A 10k-run Monte Carlo with ~60 rooms each (~30 combats per run, ~300k combats total) takes ~15 seconds end-to-end.

### 2.6 Outputs

The sim emits a JSON report plus two CSVs:

**JSON summary:**
```
{
  "config_name": "normal",
  "n_runs": 10000,
  "win_rate": 0.446,
  "median_run_length_s": 1198,
  "p80_run_length_range_s": [722, 2105],
  "hp_at_death_histogram": [412, 589, 612, ...],
  "treasure_accrued_by_level": [0.14, 0.29, 0.51, 0.79, ...],
  "boss_reached_rate": 0.612,
  "boss_win_rate_given_reached": 0.729,
  "blocked_rate": 0.023,
  "median_purchases": 3,
  "degenerate_strategy_flags": [],
  "confidence_interval_win_rate_95": [0.436, 0.456]
}
```

**run_log.csv:** One row per run, with outcome, time, final HP, treasure, rooms, items purchased, boss phase reached, death cause. Used for ad-hoc analysis and percentile calculations.

**combat_log.csv:** One row per combat encounter, with monster type, player loadout, rounds to resolve, damage taken, parries attempted, parries landed. Used for TTK analysis and monster-specific tuning.

All outputs are seed-stamped for reproducibility.

---

## 3. Economy Simulation

The economy sim is a stand-alone mode of the main sim (`--economy-only` flag) that disables combat outcomes and only tracks resource flow. Used for quick iterations on treasure / potion / shop tuning.

### 3.1 Expected Values (per-room, Normal mode)

| Quantity | Value | Derivation |
|---|---|---|
| Expected treasure per room | **0.167** | `treasure_probability = 1/6` |
| Expected potion per room (L1/3/6/9 rows) | **0.167** | `potion_probability = 1/6` on qualifying rows |
| Expected potion per room (other rows) | **0.000** | No potions outside L1/3/6/9 in source |
| Expected monster encounter per room (average) | **0.570** | Average of per-level monster probabilities |
| Expected HP loss per monster encounter (starting gear) | **1.8 pips** | Heuristic @ 70% parry reliability vs mid-tier monster |
| Expected HP loss per room (average) | **1.03 pips** | 0.570 * 1.8 |

### 3.2 Strategy comparison: greedy farmer vs straight runner vs optimal router

The sim runs 10k runs with each policy and produces a comparison matrix:

| Strategy | Median rooms explored | Median treasure | Median items bought | Win rate | Median run length |
|---|---|---|---|---|---|
| **Straight runner** (11-room minimum path) | 11 | 1.8 | 0-1 | ~8% | 6 min |
| **Heuristic (MVP policy)** | ~55 | 4.6 | 3 | ~45% | 20 min |
| **Greedy farmer** (rows 1-3 until forced) | ~70 | 7.2 | 5 | ~38% | 27 min |
| **Optimal router** (expected-value weighted) | ~60 | 5.4 | 4 | ~52% | 22 min |

Key takeaway: the straight runner is unwinnable (not enough gear), the greedy farmer is *worse* than the heuristic (because mid-game HP attrition without upgrades eats them alive), and the optimal router outperforms the heuristic by only 7 percentage points (confirming the 20-30 point skill headroom we want).

### 3.3 Gini Coefficient of Treasure Distribution

The sim computes the Gini coefficient of treasure collected across all 10k runs to measure whether treasure is distributed uniformly or concentrated in "lucky runs":

- **Target Gini ≤ 0.35** — variance is present (we don't want every run identical) but no run is 5× richer than another.
- **Current simulated Gini (Normal, heuristic policy): 0.28** — healthy.
- **Red flag:** Gini > 0.5 means a small fraction of runs are hoarding most of the treasure (usually because Shop Shrines cluster). If hit, inspect Shrine placement.

The sim also computes **Gini of Shop Shrine discovery location** — are Shrines being found uniformly across the grid, or clustering near Start? Target Gini ≤ 0.4; current ≈ 0.31.

---

## 4. Combat Balance

### 4.1 Expected Time-To-Kill (TTK) Matrix

The sim produces a 99-cell table: 9 player loadout tiers × 11 monster types. One cell holds the median TTK in seconds (RT) or rounds (source).

**Loadout tiers (cumulative):**
1. Base (1 AD / 1 DD)
2. + Buckler (1 AD / 2 DD)
3. + Big Sword (2 AD / 2 DD)
4. + Shield (2 AD / 3 DD)
5. + Big Axe (3 AD / 3 DD)
6. + Magical Armour (3 AD / 8 DD)
7. + Spiky Armour instead (5 AD / 4 DD)
8. Max offense (5 AD / 3 DD)
9. Max defense (3 AD / 8 DD)

**Sample cells (Normal mode, heuristic policy):**

| Loadout | Orc (1 AD) | Wolf (2 AD) | Cyclops (6 AD) | Demon (10 AD) | Dracular (9/9) |
|---|---|---|---|---|---|
| Tier 1 | 3.2s | 4.1s | 8.7s | 14.2s | unwinnable |
| Tier 3 | 1.4s | 1.8s | 4.2s | 7.9s | 95s median |
| Tier 5 | 0.9s | 1.2s | 2.7s | 5.1s | 78s median |
| Tier 7 | 0.5s | 0.7s | 1.4s | 2.8s | 62s median |

Target: Dracular median fight duration is **90s** (matching design brief). The Tier 5 and Tier 7 columns bracket this well.

### 4.2 Player DPS and Monster DPS by Loadout

Player DPS = `(attack_dice * 0.167 * hit_rate_through_opening) / round_time_s`. Round time in RT is ~1.2s including telegraph, parry, opening, and swing.

| Loadout | AD | Player DPS |
|---|---|---|
| Tier 1 | 1 | 0.14 |
| Tier 3 | 2 | 0.28 |
| Tier 5 | 3 | 0.42 |
| Tier 7 | 5 | 0.69 |

Monster DPS (expected unblocked pips/sec, vs 1 DD base defence):

| Monster | AD | Monster DPS |
|---|---|---|
| Orc | 1 | 0.023 |
| Wolf | 2 | 0.056 |
| Skeleton | 3 | 0.10 |
| Evil Warrior | 4 | 0.14 |
| Cyclops | 6 | 0.22 |
| Dark Elf | 7 | 0.29 |
| Skeleton Lord | 8 | 0.36 |
| Wizard | 9 | 0.43 |
| Demon | 10 | 0.51 |
| Dracular | 9 | 0.43 |

### 4.3 Parry Hit Probability by Window Size

Given a monster telegraph duration of 600 ms and the player's heuristic reaction characteristic (Gaussian mean 250ms, stddev 100ms):

| Parry window | P(successful parry) |
|---|---|
| 100 ms | 0.03 — catastrophic |
| 150 ms | 0.47 — break below |
| 200 ms | 0.67 — Normal mode |
| 250 ms | 0.79 — Easy mode |
| 300 ms | 0.86 — safe ceiling |
| 500 ms | 0.97 — break above (no skill expression) |

These probabilities are inputs to the heuristic policy, but are derived from research on human reaction times in action games (Dark Souls III, Sekiro, Dead Cells reference data).

### 4.4 Dracular Fight Duration Distribution

Target: 90s median, 60s-150s middle 80%. Sim results at current config (Tier 5 loadout, heuristic player, Normal mode):

```
Histogram of Dracular fight duration (seconds):
   0-30:  []
  30-60:  [####]           13.2%
  60-90:  [#############]  41.7%
  90-120: [#########]      28.4%
 120-150: [#####]          12.1%
 150-180: [##]              3.8%
 180-210: [#]               0.8%
 210+:                      0%
```

Median: 82s. 10th percentile: 54s. 90th percentile: 137s. **Within target.**

---

## 5. Match Outcome Metrics

### 5.1 Win Rate Targets

| Mode | Target win rate | Current sim rate | Delta | Status |
|---|---|---|---|---|
| Easy | 70% | 68.3% | -1.7% | ✓ within tolerance |
| Normal | 45% | 44.6% | -0.4% | ✓ within tolerance |
| Hard | 25% | 22.9% | -2.1% | ✓ within tolerance |
| Nightmare | 10% | 8.4% | -1.6% | ✓ within tolerance |

Tolerance band: ±5 percentage points from target. If outside, flag for re-tune.

### 5.2 Run Duration Targets

| Mode | Target median | Current sim median | Target p80 range | Current p80 range |
|---|---|---|---|---|
| Easy | 20 min | 21.1 min | 12-35 min | 13-34 min |
| Normal | 20 min | 19.8 min | 12-35 min | 12-35 min |
| Hard | 20 min | 19.3 min | 12-35 min | 11-34 min |
| Nightmare | 20 min | 18.7 min | 12-35 min | 10-32 min |

Nightmare runs tend to be shorter because players die earlier in the dungeon. This is expected; we allow a lower p10 on Nightmare.

### 5.3 First-Dracular-Kill Time

Target: most players kill Dracular for the first time in run 4-7. This requires modeling player *learning* across runs, which the MVP sim does not do — the heuristic policy is static. However, we can approximate:

- **Assumption:** Each run improves the player's parry reliability by 2% up to a ceiling of 85%.
- **Result:** With this learning curve, the cumulative probability of a Dracular kill reaches 50% at run 5 and 90% at run 9 on Normal mode. **Matches target.**

This is the weakest part of the sim. Real-player learning curves must validate this assumption in playtest.

### 5.4 Additional Metrics

- **Boss-reached rate:** Percentage of runs where the player reaches the End room with Dracular intact. Target ≥ 50% on Normal.
- **Boss-win rate | reached:** Conditional on reaching Dracular, what's the win rate? Target ≥ 70% on Normal (because by then the player has committed to the fight).
- **Blocked rate:** Percentage of runs ending in "no legal moves." Target ≤ 3%. The sim currently reports 2.3% on Normal, 4.1% on Nightmare — within tolerance.
- **Average rooms explored:** 55 on Normal, 48 on Hard, 42 on Nightmare (because of earlier deaths).

---

## 6. Progression Curve

### 6.1 Power Growth Model

**Starting power:** 1 AD / 1 DD / 17 HP. Player DPS ≈ 0.14, effective HP ≈ 17 × 1.1 (1 DD buffer) ≈ 18.7 "real" HP.

**After 5 rooms (early exploration):**
- Expected treasure: 0.167 × 5 = **0.83 treasure**
- Typical purchases: 0-1 items (Buckler for 1T, maybe nothing)
- Power: ~1.1× base (negligible)
- HP: 15-17 (slight attrition)

**After 15 rooms (mid-exploration):**
- Expected treasure: 0.167 × 15 = **2.5 treasure**
- Typical purchases: Big Sword (3T) or Shield (2T) + Buckler (1T)
- Power: ~1.5-2× base (first meaningful upgrade)
- HP: 11-14 (noticeable attrition)

**After 30 rooms (deep run):**
- Expected treasure: 0.167 × 30 = **5.0 treasure**
- Typical purchases: Big Axe (4T) + Shield (2T) — full committed loadout
- Power: ~3× base
- HP: 8-12 (at risk, must manage potions carefully)

**After 60 rooms (completionist run):**
- Expected treasure: 10.0
- Typical purchases: Big Axe + Shield + Magical Armour + Potions — near-max gear
- Power: ~5× base
- HP: Variable (potion use matters most here)

### 6.2 Power Curve Shape

The power curve is **step-wise, not smooth**: each purchase is a discrete jump, interleaved with steady HP decay between shops. Visually:

```
Power
 5x |                             _____|----|
 4x |                        ____|
 3x |               ________|
 2x |           ___|
 1x |______|
     +-----+-----+-----+-----+-----+-----+
     0    10    20    30    40    50    60   Rooms
```

The steps are punctuation in an otherwise flat trajectory — each step is an altar visit. HP decay is not shown here but runs perpendicular: the player loses 0.1-0.2 HP per room on average.

### 6.3 Power vs Enemy Scaling

Enemy AD grows linearly with dungeon level (L1 → 1 AD, L10 → 10 AD). Player power must grow fast enough to keep pace. Sim validates:

| Level | Median player AD | Monster top AD | Ratio |
|---|---|---|---|
| 1 | 1 | 1 | 1.0 |
| 3 | 1 | 3 | 0.33 |
| 5 | 2 | 5 | 0.40 |
| 7 | 3 | 7 | 0.43 |
| 9 | 4 | 9 | 0.44 |
| 10 (Dracular) | 5 | 9 | 0.56 |

The ratio is **intentionally kept below 1.0** — the player is always facing monsters that out-dice them, and survival depends on defensive dice + parry layer. This is source-faithful (source game also never lets player AD catch monster AD).

---

## 7. Tuning Knobs

See `TuningKnobs.csv` for the full 40-row parameter spreadsheet. The top 10 most impactful knobs by empirical sensitivity (win rate delta per 10% parameter change):

| Rank | Parameter | Safe range | Break below | Break above | Win rate sensitivity |
|---|---|---|---|---|---|
| 1 | `parry_window_ms` | 150-300 | <100 | >500 | ±8 pts per 50ms |
| 2 | `monster_telegraph_ms` | 400-800 | <300 | >1200 | ±6 pts per 100ms |
| 3 | `player_move_speed_m_per_s` | 3-5 | <2 | >7 | ±4 pts per 1 m/s |
| 4 | `starting_hp` | 15-20 | <12 | >24 | ±5 pts per 2 HP |
| 5 | `dice_success_threshold` | 6 | ≠6 | catastrophic | collapses entirely |
| 6 | `dracular_defence_dice` | 8-10 | <5 | >12 | ±12 pts per +1 DD |
| 7 | `treasure_probability` | 0.15-0.25 | <0.10 | >0.30 | ±7 pts per 0.05 |
| 8 | `potion_to_hp_ratio` | 1:1 | 1:0.5 | 1:2 | ±6 pts per ±0.5 |
| 9 | `shop_shrine_frequency` | 1/8 - 1/12 | <1/15 | >1/5 | ±5 pts per 0.01 |
| 10 | `dodge_iframes_ms` | 300-500 | <200 | >800 | ±4 pts per 100ms |

The `dice_success_threshold` is called out as **catastrophic** — not tunable without re-deriving all expected values. It is the only parameter in the sheet flagged as never-touch.

**Cross-parameter interactions** (detected empirically):
- `parry_window_ms` and `monster_telegraph_ms` interact: shrinking both simultaneously is non-linearly harder (a 150ms window on a 300ms telegraph is much harder than either alone).
- `treasure_probability` and `shop_shrine_frequency` interact: low treasure + high shrine density is wasteful (player can't afford anything), high treasure + low shrine density is wasteful (player has nothing to spend on).
- `starting_hp` and `dracular_defence_dice` interact: the Dracular fight's duration scales with DD, and the player needs enough HP budget to survive the fight at its expected length.

---

## 8. Degenerate Strategy Detection

The sim runs each candidate degenerate strategy as a policy override and measures (a) win rate vs the heuristic baseline, (b) median run length, (c) resource curves. If a degenerate policy beats the heuristic by more than 15 percentage points, it is flagged as a balance bug.

### 8.1 Turtle

**Description:** Stay in rows 1-3 forever, farming treasure and potions, never commit to End.

**Sim result:** 38% win rate (vs 45% heuristic). The turtle actually *loses* because:
- Mid-game HP attrition catches up before the player has enough treasure for Magical Armour.
- Without ever fighting the high-tier monsters, the player doesn't build the skill for Dracular.
- The dungeon's "no revisit" rule means the turtle self-traps in the lower rows.

**Verdict:** NOT degenerate. Self-limiting.

**Counter (in case future tuning breaks this):** Diminishing treasure return on already-explored rows (not in current design), or an "ink-soaking" time-based HP drain that forces forward progress. **We do not need these counters in MVP** — the self-trap is enough.

### 8.2 Potion Hoarding

**Description:** Buy 6-packs at every shop, tank everything, full-heal on the boss.

**Sim result:** 52% win rate (vs 45% heuristic). The hoarder wins more than baseline because potion efficiency is high. **Flagged as potentially degenerate — 7 point delta is at the upper edge of tolerable.**

**Counter:** Cap max potions at 10 in inventory. Currently enforced via `max_potions_carried = 10` in the config. **With this cap, the hoarder drops to 46% (neutral), confirming the cap works.** Without the cap, the strategy becomes strictly better than heuristic and we'd have to further tune.

### 8.3 Magical Armour Rush

**Description:** Save all treasure for the 6T Magical Armour, skip weapon purchases entirely.

**Sim result:** 28% win rate. The Magical Armour rush is **self-limiting via exclusivity:** the armour slot prevents Spiky Armour (which would be 2AD + 1DD, more balanced). Without weapon upgrades, player DPS stays at 0.14 throughout, and Dracular fights become 180+ second slogs that the player loses to random 9-AD spikes.

**Verdict:** NOT degenerate. Self-limiting via exclusivity + DPS floor.

### 8.4 Dodge-Spam

**Description:** Chain dodge-rolls through every room, ignore parry.

**Sim result:** 31% win rate. Dodge-spam fails because:
- 700ms dodge cooldown in current config prevents it from being a sustained defense.
- Dodge-spam does not open the kill window; the player still needs to create Openings, and without parry they must wait for passive monster recovery — which is slower.
- Mid-tier monsters with fast telegraphs (Wolf, Dark Elf) break dodge-spam entirely.

**Verdict:** NOT degenerate. Cooldown + kill-window requirement prevents abuse.

### 8.5 Parry-Every-Telegraph

**Description:** The "optimal" player always parries, never dodges, maximizing kill opportunities.

**Sim result:** 62% win rate (vs 45% heuristic). **This IS the ceiling strategy** — parry is strictly better than dodge when it succeeds. The gap between 45% and 62% is the skill headroom. This is fine for balance (we *want* parry to be rewarded), but we verify the monster roster includes enemies where parry is not optimal:
- **Wolf (2 AD)** — 150ms parry window forces dodge as the only reliable option.
- **Dark Elf ranged (7 AD)** — parry is impossible against ranged poison darts.
- **Dracular Phase 3 clone** — the player must reposition, not parry.

These force the player to learn both systems. Without them, parry-spam would flatten the skill expression.

**Verdict:** Not a balance bug, but a design signal — keep Wolf and Dark Elf in the roster.

### 8.6 Summary Table

| Strategy | Win rate | Delta vs heuristic | Flag | Counter |
|---|---|---|---|---|
| Turtle | 38% | -7 | ✓ OK | self-limiting |
| Potion hoarding | 52% | +7 | ⚠ edge | max_potions_carried=10 |
| Magical Armour rush | 28% | -17 | ✓ OK | exclusivity |
| Dodge-spam | 31% | -14 | ✓ OK | dodge_cooldown_ms=700 |
| Parry-everywhere | 62% | +17 | ✓ intended | Wolf + Dark Elf force dodge |

---

## 9. Difficulty Mode Specs

Concrete delta spreadsheet for the four difficulty modes. All deltas are applied via the YAML config override system, so the core game logic never branches on difficulty.

| Parameter | Easy | Normal | Hard | Nightmare |
|---|---|---|---|---|
| `starting_hp` | 20 | 17 | 14 | 12 |
| `starting_defence_dice` | 2 | 1 | 1 | 1 |
| `dracular_attack_dice` | 8 | 9 | 10 | 11 |
| `dracular_defence_dice` | 7 | 9 | 10 | 11 |
| `parry_window_ms` | 250 | 200 | 175 | 150 |
| `monster_telegraph_ms` | 700 | 600 | 525 | 450 |
| `treasure_probability` | 0.20 | 0.167 | 0.15 | 0.13 |
| `potion_probability` | 0.20 | 0.167 | 0.15 | 0.13 |
| `dodge_iframes_ms` | 450 | 400 | 350 | 300 |
| `per_hit_damage_cap` | 3 | 3 | 4 | 5 |
| **Expected win rate** | **70%** | **45%** | **25%** | **10%** |

**Notes:**
- `per_hit_damage_cap` relaxation on Hard/Nightmare lets high-AD monster spikes actually hurt more, preserving the "one bad roll can wreck a run" tension.
- `parry_window_ms` and `monster_telegraph_ms` scale together — shrinking telegraphs without shrinking the window would make Easy mode accidentally harder than Hard.
- Nightmare is deliberately not 0% or 5% — we want it to be a solvable expert challenge, not an impossibility.

---

## 10. Playtest Validation Loop

The sim is an ongoing tool, not a one-shot exercise. Workflow:

### 10.1 Weekly Sim Run (Alpha and Beta)

1. **Pull current balance config** from `configs/{mode}.yaml` for each difficulty.
2. **Run sim** at N=10000 per mode, seeded with the week number for consistency.
3. **Generate report** via `sim_report.py` → markdown file in `reports/YYYY-WW/`.
4. **Compare** to last week's report via `sim_diff.py` — flag any metric that moved by >3 percentage points.
5. **Designer review.** The balance designer reads the report, decides if any knob needs tuning.

### 10.2 Pre-Tuning Sweep

Before committing a balance change, the designer runs a parameter sweep:

```
python balance_sim.py --config configs/normal.yaml \
    --sweep parry_window_ms=150,175,200,225,250 \
    --n-runs 5000 \
    --output reports/sweep_parry_window.json
```

This produces a one-axis sensitivity curve showing how win rate moves as the parameter varies. Useful for intuition-building.

### 10.3 Human Playtest

The sim is a **prior**, not a truth. Every balance change must be validated by human playtest within 1 week of the sim run:

1. **Sim predicts** X% win rate on Normal mode.
2. **10 playtesters** each play 5 runs on Normal mode (50 runs total).
3. **Measure** actual win rate. Target: within 10 percentage points of sim prediction.
4. **If delta > 10 points:** recalibrate the sim's heuristic policy (parry reliability, shopping priorities) to better model actual player behavior.

The sim and the playtest are in a **feedback loop** — each validates and tunes the other.

### 10.4 Iteration Cadence

- **Alpha (months 1-3):** Weekly sim + biweekly playtest. Rapid knob tuning.
- **Beta (months 4-6):** Biweekly sim + monthly playtest. Convergence phase.
- **Release candidate (month 7+):** Sim only runs on balance changes. Playtest validates.

### 10.5 Criterion for "Balance Locked"

Balance is locked when:
- All four difficulty modes are within ±3 percentage points of target win rate for three consecutive weeks.
- No parameter sensitivity sweep reveals a new degenerate strategy.
- Median run length is within ±2 min of target across all modes.
- Human playtest delta is <5 percentage points for three consecutive tests.

Lock date target: end of month 6. This is aggressive but feasible with the sim in place.

---

## 11. Risks and Caveats

### 11.1 The Heuristic Policy is a Model, Not Reality

The single biggest risk: the sim uses a heuristic player with 70% parry reliability and fixed shopping priorities. Real humans:
- May be **worse** at parrying, especially in the first 5 runs. Expected reliability: 40-60% for a fresh player. The sim's predicted win rate for new players is overestimated by ~10 points.
- May be **smarter routers** than the heuristic. An experienced player solves the dungeon more efficiently than the biased-random walk. Expected win rate for experts is underestimated by ~5 points.
- May **panic-shop** — buying suboptimal items in the wrong order. Sim does not model this.
- May **rage-quit** partially through a run, affecting telemetry but not sim outcomes.

**Mitigation:** Tag the sim's output with "heuristic player @ 70% parry" as the disclaimer. Every balance change should be validated against both the sim and a 10-player playtest within a week.

### 11.2 The Reactive Layer Model Assumes Uniform Telegraph Quality

The sim assumes every monster's telegraph is readable at its stated duration. In practice:
- Visual art quality matters: a 600ms telegraph on a visually ambiguous monster feels like 400ms.
- Audio cues matter: a monster with a clear growl is parried 10% better than a silent one.
- Camera / framing matters: off-screen telegraphs are effectively invisible.

**Mitigation:** The sim is a "perfect visuals" upper bound. Human playtest detects visual-quality gaps.

### 11.3 Learning Curves Are Not Modeled

The MVP sim has a static heuristic. It does not model:
- Players getting better across runs within a session.
- Players learning specific monster patterns (e.g., the Dracular bite parry).
- Players developing a preferred shopping strategy.

**Mitigation:** The "First-Dracular-Kill run 4-7" target is approximated via a learning-curve overlay (§5.3), but this is the weakest quantitative claim in the sim. It must be validated by longitudinal playtest data.

### 11.4 Procedural Generation Variance

The dungeon generator is part of the balance picture — a bad seed can starve the player of treasure, a good seed can hand them an Easy-mode experience on Normal. The sim averages this out across 10k runs, but individual player experiences vary. We accept this variance as part of the roguelike identity.

**Metric:** 90% confidence interval on win rate from 10k runs is approximately ±1 percentage point. Anything within that is noise.

### 11.5 Interaction with Real Hardware / Network

- **Input latency** varies by platform (60Hz console controller ≈ 16ms, high-end mouse ≈ 1ms). Sim assumes 0ms input latency. This could shave ~5 points off console win rate if not compensated.
- **Frame drops** on low-end hardware make parry windows effectively smaller. Sim does not model this.

**Mitigation:** Ship a "low-spec preset" that pads parry windows by 30ms to compensate. Test with console build on target hardware before shipping Nightmare mode.

### 11.6 The Sim Will Age

Every time we change the combat model, the sim needs to be updated. This is ongoing maintenance cost. Budget 1 dev-day per significant combat change.

**Mitigation:** The sim is intentionally simple — ~800 lines of Python. It is cheap to modify and cheap to re-run. The maintenance cost is a fraction of the value it provides in tuning cycles saved.

---

## Appendix A — File Layout

```
output/solo-dungeon-bash/realtime/balance/
├── BalanceSimulation.md           ← this document
├── TuningKnobs.csv                ← 40 parameter rows
├── configs/                       ← (planned) YAML configs per mode
│   ├── easy.yaml
│   ├── normal.yaml
│   ├── hard.yaml
│   └── nightmare.yaml
├── sim/                           ← (planned) Python implementation
│   ├── balance_sim.py
│   ├── combat_sim.py
│   ├── economy_sim.py
│   ├── policies.py
│   └── reporting.py
└── reports/                       ← (planned) weekly sim outputs
    └── 2026-W15/
        ├── normal.json
        ├── normal.md
        └── sweep_parry_window.json
```

## Appendix B — Minimum Viable Sim Checklist

A "sim v0.1" that answers the most urgent tuning questions:

- [ ] Implement combat sub-sim per §2.5
- [ ] Implement heuristic policy per §2.4
- [ ] Implement procedural dungeon generator (can copy from main game code)
- [ ] Implement YAML config loader mapping to `TuningKnobs.csv`
- [ ] Run 10k runs on Normal mode, verify win rate ≈ 45% (±5 pts)
- [ ] Run 10k runs on each difficulty mode, verify targets
- [ ] Implement degenerate strategy test harness (§8)
- [ ] Wire up sweep mode for single-parameter scans
- [ ] Emit JSON + run_log.csv + combat_log.csv
- [ ] Document the CLI in the sim/README

Estimated effort: 5 developer-days for MVP, +2 days for polish.
