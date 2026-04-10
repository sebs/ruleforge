# Multiplayer & Competitive Design — Solo Dungeon Dash

> RealTimeForge Stage RT-A5
> Source docs: `RTGDD.md`, `GenreCrystallization.md`, `ConflictModel.md`, `EconomyModel.md`
> Status: **Exploratory / contingency planning.** Primary launch is strictly single-player.

---

## 1. Verdict: Ship Single-Player First

**Solo Dungeon Dash 1.0 is a single-player-only game. There is no multiplayer in the shipping product. This document exists to plan what multiplayer *could* look like if — and only if — post-launch data justifies it.**

The name of the game contains the word **Solo**. The source board game, *Solo Dungeon Bash* (Felbrigg Herriot, 2007), is explicitly a solitaire ruleset with no multiplayer variant, no team rules, no versus mode, not even a hot-seat alternative. Its entire design pressure — the alternating lethal dice pool, the 17-HP cap, the 9×10 commitment grid, the lone Dracular duel at the End square — is tuned for one player making one sequence of decisions.

### Why the default is "no multiplayer"

1. **Source fidelity.** The source is solo. Adding multiplayer is not translation; it is invention. Invention is fine, but it is a different product.
2. **Scope multiplication.** Every multiplayer mode roughly triples engineering scope: netcode, lobbies, matchmaking, anti-cheat, server infra, account linking, desync handling, support load. A six-month solo project becomes an eighteen-month multiplayer project.
3. **Balance breakage.** The dice math in Solo Dungeon Dash is tuned for a single attacker and a single defender. A shared dice tray or parallel trays fundamentally changes the success curve for every encounter. Bosses re-tune, enemy HP re-tunes, reward rates re-tune. See `EconomyModel.md` — the economy is calibrated for a 17 HP hard cap and a specific encounter frequency. Doubling the players roughly halves the effective enemy threat unless enemies are rebuilt.
4. **Audience mismatch.** The target audience is the Hades / Dead Cells / Into the Breach solo-roguelite crowd. That audience prefers quiet, introspective, solitary runs. Surveys in that genre consistently show that under 20% of players ask for multiplayer, and the ones who do rarely play it for more than a few hours.
5. **Brand identity.** "Solo" is in the product name. Marketing co-op or versus muddles the proposition.
6. **Opportunity cost.** Every engineering hour spent on netcode is an hour not spent on content: more bosses, more biomes, more enemy variety, deeper metaprogression. For a roguelite, content depth is the primary retention driver.

### What this document is *not*

- Not a commitment to build multiplayer.
- Not a feature list for 1.0.
- Not a stretch goal anyone should depend on.

### What this document *is*

- An honest exploration of three possible multiplayer modes.
- A ranked recommendation.
- Network architecture notes in case Mode B is ever built.
- Monetization, toxicity, and risk considerations.

**If the team ships 1.0, sees stable sales, and six months later has headroom, this document exists so someone does not have to start from zero.**

---

## 2. Three Multiplayer Modes to Consider

Three candidate modes, ordered from lowest to highest cost and (not coincidentally) from highest to lowest identity faithfulness.

### Mode A — Asynchronous Daily Seed Ghost Race

> Think: Trackmania's ghost overlays, Spelunky's daily challenge leaderboard, Dead Cells's Boss Cell leaderboards.

**Concept**
Every calendar day, the game generates a single procedurally seeded dungeon — the **Daily Dungeon**. All players worldwide play the same seed, with the same enemy layout, the same treasure rolls, the same shop contents, the same Dracular. Runs are recorded as a stream of inputs (or a compact path + combat log) and uploaded on completion. Other players can then watch "ghost" overlays of other runs — silhouetted dashes, parries, and room transitions — while attempting their own run.

**Interaction model: none real-time.**
- No two players are ever in the same match at the same time.
- No state is shared.
- The multiplayer is purely asynchronous observation.

**Competitive axes**
- **Speed** — shortest clear time wins.
- **Cleanliness** — fewest hearts lost (ties broken by time).
- **Purity** — no shop purchases, minimalist runs.
- **Gold** — most treasure looted before Dracular dies.

Players pick which leaderboard they care about. The daily seed rotates at 00:00 UTC.

**Social layer**
- Global top-100 per axis.
- Friends-only leaderboard (opt-in; fetched from Steam friends or Game Center).
- "Share your run" button — copies a text summary (seed code, clear time, hearts lost, build choices) for posting elsewhere.

**Technical complexity: LOW**
- No real-time netcode.
- Server is essentially a key-value store: `(date, player_id, seed) → run_recording`.
- Replay is deterministic because the game's RNG is seeded and the simulation is fixed-timestep.
- Server-side verification: re-simulate the replay using the seed and check that the reported outcome matches. Cheaters who edit their save are rejected automatically.
- Ghosts are lightweight: store player position samples at 10 Hz plus combat events. A 20-minute run fits in under 200 KB uncompressed, well under 50 KB gzipped.

**Faithfulness: HIGH**
- Gameplay is identical to the single-player experience.
- The player is still alone in the dungeon. They see translucent ghost silhouettes of other runners dashing through rooms, but the ghosts cannot interact, block, or aggro.
- The meditative solo feel is preserved — you can turn ghosts off entirely and it's just the base game.

**Timeline**
- Can ship with 1.0 if server infra is ready.
- More likely: ships as a **free 1.1 update** six to twelve weeks after launch, giving the team runway to stabilize the base game first.

**Risks**
- Server cost: modest. A daily seed means only ~365 seeds a year; run uploads are small. Should cost under $100/month at 10k active players on a budget cloud provider.
- Leaderboard cheating: mitigated by deterministic replay verification.
- Cold start: first day, first hour, leaderboard is empty. Mitigated by pre-recording developer and beta-tester ghosts so the leaderboard is seeded.

### Mode B — Synchronous Two-Player Co-op

> Think: Hades II's planned co-op, Enter the Gungeon's co-op DLC, Cult of the Lamb's co-op patch.

**Concept**
Two players, one dungeon, same seed. Both characters are on screen simultaneously. They move together through the 9×10 grid, fight shared enemies, open shared shops, and face Dracular together at the End.

**Key design decisions**

**HP pools: separate.** Each player has their own 17-HP cap. Sharing a pool would eliminate the tension of individual commitment and create blame-games when one player dies.

**Dice Tray: combined.** When combat starts in a shared room, the Attack Dice of both players merge into a single Dice Tray roll. Defensive rolls happen per-player, because each player defends only their own hearts. This preserves the "dice on the right side of the screen" visual spectacle while making both players feel that their gear contributes.

**Treasure and Potions: shared stockpile.** Both players draw from the same Treasure pile and the same Potion pool. Shop purchases come from the shared pile and the buying player chooses what gets bought (with obvious coordination pressure).

**Shop Shrines: shared.** One Shop Shrine per level, both players browse simultaneously, either player can spend. Upgrades apply to the player who claims them — no duplicates.

**Revive mechanic.** If Player A falls to 0 HP, they drop to the ground as a "bleeding out" token. Player B has 10 seconds to reach them and tap the revive button. A successful revive brings Player A back at 1 HP (mirrors the "hard reset but a sliver of hope" source feel). If the 10 seconds expire, both players lose the run. This is deliberately unforgiving — the source is brutal, and co-op should feel brutal too.

**Dracular fight.** Both players fight Dracular together. Dracular's 9 Defence Dice are re-tuned to ~13 DD to account for combined Attack Dice, preserving the boss's "wall of denial" feel.

**Routing.** Both players must agree on which door to take. Implementation: each doorway requires both players to stand near it for half a second. This naturally forces conversation and commitment without blocking progress.

**What co-op changes for the game's identity**
- Routing becomes a conversation instead of a meditation.
- The "alone in the dark" feel is replaced by "two against the dark" — less existentially lonely, more heroic-buddy-flavor.
- Dice rolls become shared celebrations.

**Technical complexity: HIGH**
- Real-time netcode with two remote clients.
- Lockstep or authoritative-host model (see §10).
- Deterministic simulation from a shared seed.
- Lag compensation and desync handling.
- Reconnect logic when one player drops.
- State sync on join.
- Separate matchmaking flow (invite codes).
- Separate balance pass on all encounters and Dracular.
- Testing load: co-op bugs are notoriously hard to reproduce.
- Estimated: **6 months of dedicated dev for two engineers**, plus design and QA overhead.

**Faithfulness: MEDIUM**
- Room-to-room routing feel: preserved.
- Dice Tray spectacle: preserved, even enhanced.
- Solo meditative feel: lost.
- 1-vs-1 combat intimacy: lost.
- The source is explicitly solitaire; adding a partner is a branching from, not a translation of, the source.

**Timeline**
- Not before **1.5**, and only if player base is asking for it.
- Sold as a paid expansion DLC (see §9), $5–$8.
- Bundled with 2–3 new bosses and a new biome to justify the price.

### Mode C — Competitive Split-Screen Duel

> Think: Puyo Puyo's garbage-sending, Tetris 99's attack mechanics.

**Concept**
Two players each get their own identical-seed dungeon, displayed side-by-side in split-screen. The first to kill Dracular wins. Losing all hearts is an instant loss regardless of the opponent's state.

**The twist: harass mechanic.**
Every sixth 6 that Player A rolls spawns a **Ghost Enemy** in Player B's current room, and vice versa. Ghost Enemies are weak (AD 2, 1 HP) but they interrupt flow, force parries, and drain hearts. High-rolling players harass their opponents; low-rolling players race for clean runs.

**Why this mode exists in this document**
Because some stakeholder will eventually ask: "have you considered a versus mode?" The answer is yes, here it is, and here is why we shouldn't build it.

**Technical complexity: MEDIUM**
- State sync is minimal: each player's dungeon runs independently on their own machine. Only two things need to cross the wire: (1) periodic "rolls to send" counts, (2) keepalive and end-of-game confirmation.
- No shared-world collision or interaction.
- Matchmaking needed (friends-only or ranked).

**Faithfulness: LOW**
- This turns a quiet, thoughtful, solitary run into a hyper-competitive race.
- It adds harass mechanics that do not exist in the source.
- It replaces the "me vs. the dungeon" conflict model with a "me vs. another player" conflict model, which `ConflictModel.md` explicitly rejects as out of scope.
- A player who likes Solo Dungeon Dash probably does not want this.
- A player who wants this probably does not want Solo Dungeon Dash.

**Faithfulness verdict: fundamentally off-brand.**

**Timeline**
- **Do not build.** Revisit only if a specific publisher or platform contract demands a versus mode and the money is sufficient to justify forking the identity of the game.

---

## 3. Recommended Path

In priority order:

1. **Ship 1.0 single-player.** Strict focus. Polish the solo experience. Earn the audience.
2. **Mode A — Ghost Races as free 1.1 update.** Six to twelve weeks post-launch. Low risk, high engagement uplift, preserves identity, gives reviewers a reason to re-cover the game, gives the community a shared daily conversation ("did you see the top time today?").
3. **Mode B — Co-op as paid 1.5+ DLC, only if the community asks for it.** Evaluate at the one-year mark. Criteria: does the Discord / Reddit / store page reviews have a steady drumbeat of "I want to play this with a friend"? If yes, consider. If no, spend that engineering time on a new biome instead.
4. **Mode C — Do not build.** Ever. It betrays the brand.

### Decision gates

| Gate | Signal | Action |
|---|---|---|
| 1.0 launch + 30 days | Stable reviews, < 2% crash rate, daily active users trending up or flat | Begin Mode A development |
| 1.1 launch + 90 days | Daily active users up or flat, leaderboard engagement > 15% of runs | Consider Mode B planning |
| 1.0 launch + 12 months | Recurring "please add co-op" sentiment in ≥ 20% of top-voted reviews/forum threads | Commit to Mode B dev |
| Any time | A stakeholder asks for Mode C | Show them this document |

---

## 4. Matchmaking — Mode A (Ghost Races)

Minimal and stateless.

**Leaderboards**
- Daily seed = daily leaderboard. At 00:00 UTC the seed advances and a new empty board starts.
- Sort keys: time, cleanliness (hearts lost), purity (items unused), gold.
- Global, regional (based on IP country), and friends-only views.
- Top 100 globally; your own position and neighbors (±5) shown regardless of rank.

**No rating system**
- No ELO, no MMR, no seasons, no tiers.
- Just a simple ranked list per seed per day.
- Historical leaderboards archived for 30 days then pruned.

**Cold-start strategy**
- Developer and beta-tester ghosts pre-recorded for every daily seed in the first three months.
- Ensures the leaderboard is never empty on day 1 of the feature.
- Beta ghosts are clearly marked as such ("Beta Ghost — Felbrigg") so players do not feel the board is fraudulent.

**Anti-cheat**
- Every submitted run is a deterministic replay. The server re-simulates the replay from the seed and the recorded inputs and checks that the final state matches the claimed outcome.
- Runs that fail verification are silently rejected. No ban, no notification, just not listed.
- This catches >99% of save-editing and memory-hacking attempts.
- Assumption: the game is fully deterministic given (seed, input stream). This must be enforced as a core engineering requirement during 1.0 development — no `Time.deltaTime` in gameplay code, no non-seeded RNG, no float drift from physics. This is a hard constraint.

**Friends**
- Platform-native friends (Steam, Game Center, Epic).
- Optional: "seed code" sharing for private challenges — a player can generate a non-calendar seed and send the code to friends; only ghosts from players who entered that code are visible.

**Infrastructure**
- Single small backend: REST API for run submission and leaderboard queries, one relational DB table per seed, one object store bucket for replays.
- No persistent connections needed.
- Costs scale with active players, not with server-tick rate. Cheap.

---

## 5. Matchmaking — Mode B (Co-op, if built)

Deliberately simple. No grand matchmaking architecture.

**Invite-code only**
- Host creates a private lobby and gets a 6-character code.
- Guest enters the code and joins.
- **No random matchmaking.** This is a deliberate choice — random matchmaking invites toxicity, griefing, and support load that a small team cannot handle. Friend co-op or nothing.

**Lobby flow**
1. Host clicks "Create Co-op Run" → gets code.
2. Host selects difficulty, seed (daily or random), optional modifiers.
3. Guest enters code, sees lobby, clicks Ready.
4. Host clicks Start → both transition into the same dungeon.

**No rating, no progression gating**
- No co-op ranked ladder.
- Co-op achievements are separate from solo achievements (so a solo player is never pressured into co-op to 100% the game).
- Co-op unlocks (cosmetic) are tracked per-player, not per-session.

**Drop-in / drop-out: not supported**
- If a player disconnects mid-run, the remaining player sees a "waiting for partner" pause screen.
- 30-second reconnect window.
- If reconnect fails, the run ends as a mutual loss.
- Why not drop-in: it triples complexity around mid-run state sync, balance for a single player, partial-progress credit. Not worth it for 1.5 DLC scope.

**Friend discovery**
- Platform-native friends list integration.
- Optional direct-join from friends list ("Join Amy's game") if Amy has created a lobby and enabled friends-can-join.

**Max party size**
- 2. Never more. The source is a 1-player game. 2-player is a stretch. 3+ is a different game.

---

## 6. Ranked / Competitive Structures — N/A

**None, across any mode.**

- No ranked ladder.
- No seasons.
- No competitive tiers (Bronze / Silver / Gold / etc.).
- No MMR or ELO.
- No placement matches.
- No season rewards.
- No tournaments (team-run; third parties may organize informally and that is fine).
- No leaderboard prizes.

Daily seed leaderboards in Mode A are informal bragging rights only. The game never awards in-game currency, cosmetics, or meaningful progression based on leaderboard position.

**Why**
Ranked systems generate stress, churn, cheating pressure, matchmaking complexity, and a treadmill that is at odds with the "20 minutes, one more run, just for fun" vibe.

---

## 7. Social Systems — Minimal

The philosophy: **social features should enhance the solo experience without creating obligation or toxicity vectors.**

**Implemented (if Mode A ships)**
- Platform-native friends list (Steam friends, Game Center friends, Epic friends).
- Daily leaderboard with friends view.
- Ghost overlays on/off toggle.
- "Share run" button that copies a text summary to the clipboard for social media posting. Format example:
  ```
  Solo Dungeon Dash — Daily 2026-04-10
  Clear: 14:32 · Hearts lost: 2 · Gold: 118
  Build: Big Sword +3, Chain Mail, Lucky Coin
  Play the same seed: [link]
  ```

**Low-priority / maybe**
- "Leave a note on your ghost run" — max 50 characters, shown as a chat-bubble next to your ghost silhouette when another player passes it. Prone to abuse (profanity, slurs), so: pre-moderated wordlist filter, player reporting, no note = default. Ship only if community asks.
- Emoji reactions on top-100 runs. Maybe.

**Not implemented, ever**
- In-game text chat (Mode A or B). Short-run games do not benefit from chat and the toxicity vector is significant.
- In-game voice chat. Platforms already provide party chat; we do not need to build our own.
- Clans, guilds, teams. Not the right scale.
- Friend gifting of in-game items. Complication with no upside.
- Public profiles with run history. Privacy implications, moderation load, minimal player value.

---

## 8. Toxicity Mitigation

A solo game with minimal social features is already a low-toxicity environment. The goal is to *keep* it low, not to build a moderation empire.

**Core principle: remove the vectors, not the players.**

**Vectors removed by design**
- No text chat → no chat harassment.
- No voice chat → no voice harassment.
- No direct PvP → no griefing via gameplay.
- No trade → no scamming.
- No guilds → no guild politics.
- No public profiles → no stalking.

**Vectors that remain**
- Player display name: platform-managed, platform-moderated. We show whatever Steam / Game Center provides.
- Ghost-run "notes" (if implemented): wordlist filter, optional reporting, one-tap mute.
- Daily seed leaderboard entries: the only player-visible content is a name and a time. Low surface area.
- Co-op (if Mode B ships): friend-only matchmaking means a player has to actively invite their abuser, which is an unusual self-harm pattern we can accept the residual risk of.

**Reporting**
- One-tap "report this name / this ghost note" button on any leaderboard entry or ghost overlay.
- Reports go to a simple review queue; repeat offenders get their names auto-mangled (e.g. replaced with "Player#####") and their ghost notes blanked.
- No ban system — since there is no account economy, a ban would just mean making a new Steam account, which is wasted effort on everyone's part. Name-mangling is sufficient.

**Moderation load estimate**
- With 10k DAU, expect fewer than 5 reports per day.
- Can be handled by one community person for an hour a week.
- If reports spike, add a third-party automated moderation service (e.g. Modulate, Community Sift) to triage.

---

## 9. Monetization Model Recommendation

**Primary launch: premium single purchase.**

| Tier | Price (USD) | What it includes |
|---|---|---|
| Base game | $7–$12 | Full single-player Solo Dungeon Dash, all bosses, all biomes, all metaprogression. Daily seed with Mode A Ghost Races (if bundled). No expiring content. |

**Justification for $7–$12 range**
- Hades: $24.99 (premium roguelite benchmark, content-deep)
- Dead Cells: $24.99 (premium roguelite benchmark, content-deep)
- Into the Breach: $14.99 (shorter-runtime indie roguelite)
- Downwell: $2.99 (very short indie roguelite)
- Solo Dungeon Dash is closer to Into the Breach in scope, with ~20-minute runs and a single campaign arc. $9.99 is the landing price, with $7.99 for sales and $11.99 for platforms that round up (consoles).

**Post-launch cosmetic DLC (optional, light touch)**

| Item | Price (USD) | Notes |
|---|---|---|
| Dice skin packs | $2–$3 | Visual-only reskins of the Dice Tray. E.g., "Bone dice," "Star dice," "Cursed dice." No gameplay effect. |
| Character palette packs | $2–$3 | Alternate colors/outfits for the player character. |
| Dungeon theme packs | $3–$5 | Visual reskins of a biome (e.g. "Snowbound Dungeon," "Undersea Dungeon"). Identical layout/enemies, different art. |

**Key constraints on cosmetic DLC**
- Purely cosmetic. Never affects damage, HP, dice odds, or any gameplay value.
- No currency, no microtransactions, no in-game store. Buy once from Steam/platform page.
- No loot boxes. No gacha. No random drops.
- No FOMO. Once released, stays available forever.
- Launch base game with at least two character palettes and two dice skins **free** to set expectations.

**Mode B co-op expansion (optional, only if demanded)**

| Item | Price (USD) | Contents |
|---|---|---|
| "The Second Adventurer" expansion | $5–$8 | Unlocks co-op (Mode B). Bundled with 3 new bosses, 2 new dungeon biomes, 10+ new enemy types, 5+ new gear items, co-op achievements. |

**Content ratio**: the co-op netcode alone would not justify $5+. Bundling it with substantial new solo content ensures solo-only players still get value if they buy it, and gives the expansion a real content story for marketing.

**What we will NOT do**
- No battle pass. No seasonal pass. No season reset. No "miss it and it's gone forever."
- No ads. None. Not even rewarded video.
- No energy systems. No wait timers.
- No premium currency. No loot boxes. No gacha.
- No pay-to-win. No pay-to-progress.
- No "day one DLC" that was cut from the base game to sell back.

**Rationale**
The solo-roguelite audience is vocally hostile to aggressive monetization. Hades, Dead Cells, Into the Breach, Slay the Spire, Balatro — all shipped as premium titles with no microtransactions and were commercially successful. The audience rewards respect for their time and money. The audience punishes FOMO mechanics. We follow the established successful pattern.

---

## 10. Network Architecture Sketch — Mode B Co-op

Only relevant if Mode B is ever built. Kept at high level since the decision to build is uncertain.

### Topology: Peer-to-Peer with Host Authority

**Decision: P2P with host-authoritative simulation.**

Alternatives considered:
- **Dedicated servers.** More reliable, lower latency for geographically-distant players, but requires running and paying for server fleet, which is massive overhead for a 2-player co-op game. Rejected.
- **Full peer-to-peer (no authority).** Each player simulates locally and hopes to stay in sync. Fragile. Rejected.
- **Host-authoritative P2P.** One player (the host) is the source of truth. The guest sends inputs to the host, the host simulates, and the resulting state is echoed back. Rollback for local responsiveness. **Selected.**

### Netcode Model: Deterministic Lockstep

**Decision: Lockstep with seeded deterministic simulation.**

- Both clients simulate the exact same dungeon from the exact same seed.
- Each tick, both clients send their inputs (movement vector, parry input, dodge input, attack input, interact input) to the other via the host.
- Once both clients have both sets of inputs for tick N, they both advance the simulation to tick N+1.
- No visual rollback, no prediction — the game is turn-paced enough that a ~60 ms input delay is acceptable and imperceptible during exploration, and combat is telegraph-driven so minor delay is fine.

**Why lockstep for this game**
- The dice math is already deterministic once seeded (core engineering requirement for Mode A anti-cheat).
- Only 2 players, so the "worst-case latency = slowest player" problem is bounded.
- Bandwidth is trivial — a few bytes of input per tick per player.
- Simpler than rollback, simpler than state sync, simpler than snapshot interpolation.

**Tick rate**
- Simulation: **30 Hz** (33 ms per tick).
- Render: **60 Hz** (or device native, decoupled from simulation).
- Input sample: 60 Hz locally, sent at simulation rate.

### State Sync & Desync Handling

- Every 5 seconds, both clients compute a 64-bit checksum of the authoritative game state (player positions, HP, dice tray contents, enemy states, inventory).
- The host compares its checksum against the guest's.
- If mismatched for two consecutive checks → **desync detected**.
- On desync: both clients pause, display "Synchronizing…", host sends a full state snapshot, guest applies it, both resume.
- If desync recurs three times in one run: end the run with a graceful error message and a "please report this" button.
- Logs of desync events go to a crash-reporting service for analysis.

### Lag Compensation

- **None at the simulation layer** — lockstep makes it unnecessary. Both clients advance at the pace of the slower client's network.
- **Some at the render layer** — for the duration while waiting for the next input pair, clients interpolate animation so the game does not freeze visually.

### Drop Handling

- If a client stops sending inputs for > 2 seconds → show "waiting for partner" overlay on the still-connected client.
- Attempt reconnection for 30 seconds.
- On reconnect success: host sends full state snapshot, guest applies, simulation resumes.
- On reconnect failure: end the run with a mutual-loss notification. No partial progress credit (simpler than adjudicating who was "winning").

### Bandwidth Budget

- Input packet: ~12 bytes per player per tick (movement vector, button states, timestamp). At 30 Hz, ~360 B/s per player, under 1 KB/s round trip. Trivial.
- Checksum packet: ~16 bytes every 5 seconds. Trivial.
- Snapshot (only on desync or reconnect): ~4 KB. Rare.
- Total: **< 2 KB/s per player in steady state.** Works on hotel wifi, works on 4G.

### Latency Tolerance

- **< 100 ms RTT:** smooth, feels local.
- **100–200 ms RTT:** playable, occasional "why did my parry feel late?"
- **200–300 ms RTT:** noticeable input lag on combat, but telegraph-driven enemies still allow success.
- **> 300 ms RTT:** warning shown, encourage players to play with geographically closer friends.

We do not implement regional matchmaking (no random matchmaking anyway). Friends bring their own latency.

### Reference Stack

- Transport: UDP via a proven middleware (e.g., Steamworks Networking Sockets on PC, GameKit on Apple, an off-the-shelf library like GameNetworkingSockets on cross-platform).
- Relay: Steam's relay network when both players are on Steam; otherwise NAT punchthrough via platform-provided services; otherwise fall back to a minimal relay we host.
- Serialization: manual bit-packing of inputs (they're small), Protocol Buffers or FlatBuffers for snapshots.

---

## 11. Cross-Platform Play

Only relevant for Mode B. Platform matrix:

| Host \ Guest | PC | Steam Deck | Mobile (iOS) | Mobile (Android) |
|---|---|---|---|---|
| **PC** | yes | yes | no | no |
| **Steam Deck** | yes | yes | no | no |
| **Mobile (iOS)** | no | no | yes | yes |
| **Mobile (Android)** | no | no | yes | yes |

**Rationale**
- **PC ↔ Steam Deck:** effectively the same platform (Steam Deck runs the Steam PC build). Trivial.
- **Mobile ↔ Mobile:** both iOS and Android use the same touch-based input paradigm. Build layouts are identical.
- **PC ↔ Mobile:** rejected because of input asymmetry. A PC player with mouse-and-keyboard or controller has precise parry timing; a mobile player on touchscreen has a slightly slower, slightly less precise input method. Mixed lobbies would feel unfair. Better to keep co-op within input paradigms.

**Account linking**
- Not required. Each platform's native account is sufficient.
- Cross-platform friends list would require a unified account system, which is out of scope.
- Result: you can co-op with friends on the same platform family. That is enough.

---

## 12. Risks of Adding Multiplayer

Ordered by severity.

### Identity dilution (highest)
**Risk:** The name contains "Solo." The marketing promise is a meditative single-player roguelite. Adding multiplayer — especially co-op — confuses the brand and the audience.
**Mitigation:** Ship 1.0 strictly solo. Only ever add multiplayer as an optional mode, clearly marked. Never lead marketing with multiplayer.

### Scope creep
**Risk:** Multiplayer always takes longer than estimated. Mode B at a six-month estimate will likely become nine.
**Mitigation:** Do not commit to multiplayer during 1.0 development. Bring it up only after 1.0 is stable and the team has bandwidth.

### Balance drift
**Risk:** Combined dice trays, shared HP decisions, and 2× player damage output all require rebalancing every encounter and every boss. A naive port will make co-op trivially easy or frustratingly hard.
**Mitigation:** Dedicated balance pass for co-op as a separate project. Treat co-op as its own balance problem with its own playtesting cycle.

### Netcode bugs
**Risk:** Lockstep is deterministic but *fragile* — any float drift, any unseeded random call, any `Time.deltaTime` in gameplay code causes desync. Desync is hard to diagnose and reproduce.
**Mitigation:** Enforce determinism as a core engineering requirement from day one of 1.0 development (already required for Mode A anti-cheat). Write determinism tests. Add a "replay from seed" CI check that runs every build.

### Server and infra costs
**Risk:** Even P2P needs some matchmaking infra (invite codes, relay fallback). This is ongoing operational cost.
**Mitigation:** Use platform-provided services (Steamworks, GameKit) wherever possible. Avoid running custom servers.

### Toxicity concerns
**Risk:** Even without chat, ghost-run notes, names, and invite-only co-op still have some toxicity surface.
**Mitigation:** Described in §8. Keep moderation load manageable.

### Support load
**Risk:** Multiplayer bug reports are disproportionately hard to investigate: "my partner and I desynced in the boss fight." We need logs, repro steps, network traces.
**Mitigation:** Build strong client-side logging and a one-click "send diagnostic bundle" feature. Budget support time accordingly. Consider dropping Mode B entirely if support load exceeds what the team can handle.

### Fan disappointment
**Risk:** A subset of the solo audience may feel that multiplayer is a betrayal or a sign that future content will go multiplayer-only.
**Mitigation:** Communicate early and often: "1.0 is solo. All future solo content will continue. Multiplayer is an optional mode, not a new direction."

### Partner ghosting
**Risk:** In co-op, one player may need to stop playing mid-run (work, family, bathroom). The other player either waits or loses progress.
**Mitigation:** Pause-on-disconnect with 30-second reconnect, graceful mutual-loss as described in §10. Set expectations: "co-op runs take 20 minutes, plan accordingly."

### Opportunity cost
**Risk:** Every hour on multiplayer is an hour not on content.
**Mitigation:** Rigorously evaluate at decision gates (§3). Prefer content.

---

## 13. Final Recommendation

**Build a great solo game first. That is the entire job for 1.0.**

The recommendation in one paragraph: ship Solo Dungeon Dash 1.0 as strict single-player, polish it to a shine, earn a stable audience, then add **Mode A (Daily Seed Ghost Races)** as a free update around 1.1 because it is cheap, low-risk, and identity-preserving. Evaluate **Mode B (Co-op)** at the one-year mark based on sales, community feedback, and a credible drumbeat of co-op requests — if built, ship it as a paid expansion bundled with solo content so solo-only players still get value. **Never ship Mode C (Competitive Duel)** — it is fundamentally off-brand and would damage the solo identity for no upside.

### One-line summary per mode

| Mode | Ship it? | When? | Price |
|---|---|---|---|
| 1.0 solo | **Yes** | Launch | $7–$12 premium |
| Mode A — Ghost Races | **Yes** | 1.1, ~2–3 months post-launch | Free |
| Mode B — Co-op | **Maybe** | 1.5+, ~12+ months post-launch, only if community asks | $5–$8 DLC with content |
| Mode C — Versus Duel | **No** | Never | N/A |

### The brand line

**Be proud of being a solo game.** That is the brand. That is the promise. The source is solo. The name is "Solo" Dungeon Dash. The audience came here for a solo experience. Everything in this document is contingency planning around that central truth; the truth itself is: we are making a solo game, and it is going to be great because of that, not in spite of it.

---

*End of RT-A5 — Multiplayer & Competitive Design*
