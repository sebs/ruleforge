# Playtest Scripts — Solo Dungeon Dash

**Project:** Solo Dungeon Dash (RealTimeForge RT-A7)
**Phase:** Alpha playtesting
**Owner:** Design / Production
**Last updated:** 2026-04-10

This document defines three structured playtest sessions used to validate Solo Dungeon Dash from the atom of combat up through the full 20-minute run arc. Each session is a self-contained protocol — a new facilitator should be able to run it with only this document, a build, and a participant.

The three sessions are intentionally staged against the three base prototype prompts:

| # | Script | Build target | Question answered |
|---|---|---|---|
| 1 | Core Loop Validation | Prompt 1 — Core Loop Prototype | Does the 30-second loop teach itself? |
| 2 | Combat Feel Depth | Prompt 2 — Multi-Enemy Conflict Prototype | Is combat satisfying and varied? |
| 3 | Economy & Pacing | Prompt 3 — Full Vertical Slice / Alpha | Does a full 20-minute run arc land? |

A final section, **Overall Playtest Protocol**, lists the procedural, ethical, and logistical rules that apply to all three scripts.

---

## Playtest Script 1 — Core Loop Validation

### Purpose

Does the 30-second core loop work without any instruction? Can a new player figure out "move, parry, attack, kill" organically, purely from the affordances of the prototype? This test is the single most important gate in the whole project: if the core loop does not teach itself, nothing else we build will save it.

Specifically we want to answer:

1. Do players discover movement, attack, parry, and dodge in that order (or any functional order) without being told?
2. Does the Dice Tray communicate "this is what the combat math is doing" without explanation?
3. Does the first successful parry produce a visible "aha!" moment?
4. Is 5–10 minutes of unguided play enough to reach competent play?
5. Does the raw core loop feel fun enough that a player would voluntarily continue?

### Participants

**Count:** 5 people, run individually (not in groups).

**Profile mix:**

- 1 casual player — primarily plays mobile or cozy games, rarely touches action games.
- 2 moderate players — plays PC/console games a few hours a week, familiar with keyboard-and-mouse but not a genre specialist.
- 2 heavy action-game players — has logged serious hours in at least one of: Hades, Dead Cells, Elden Ring, Devil May Cry, Sekiro, Hollow Knight.

Age range: 18–45. No professional game designers, no friends who have already been told about Solo Dungeon Dash.

**Recruitment channels:**

- Friends-and-family network (with the explicit "have you heard anything about this project?" screen).
- Local indie dev community Discord (`#playtesting` channels).
- Indie playtest networks (e.g., PlaytestCloud lite tier, itch.io playtesting threads).
- University gaming clubs for the casual / moderate slots.

**Screening questions before booking:**

- "How many hours a week do you play video games?"
- "Name the last 3 games you put more than 5 hours into."
- "Have you heard about a game called Solo Dungeon Dash or Solo Dungeon Bash?" (reject yes)
- "Do you own a gamepad?" (not required, just logged)

### Duration

**30 minutes per participant.**

- 5 min: arrival, consent, webcam setup.
- 15 min: play session (hard cap: stop at 15 minutes even if player wants more).
- 10 min: debrief interview.

Book in 45-minute slots to allow changeover.

### Build target

**Prompt 1 — Core Loop Prototype** (from the base prototypes deck).

Required scope in the build:

- 1 room, 1 enemy type (Orc with telegraphed overhead).
- WASD move, mouse-aim, LMB attack, RMB parry, Space dodge.
- Visible Dice Tray with current attack die and defense die.
- Hit-stop, parry flash, and basic VFX/SFX already tuned to "first pass acceptable".
- Placeholder art is OK (flat shapes + primitive sprites).
- **Tutorial text disabled.** No tooltip popups, no input glyphs, no help screen.
- Debug logging to CSV (see Quantitative metrics).

Not required for this build:

- Shop, multiple rooms, map, additional enemies, boss.

### Pre-session setup

**Per session:**

1. Laptop or PC with the build pre-launched on the title screen.
2. Mouse and mechanical keyboard (consistent hardware across all 5 sessions).
3. Webcam framed on the participant's face (for expression coding), NOT on the screen.
4. Screen recording running (OBS, 1080p, 30fps).
5. Audio captured from room mic, not game.
6. Observer's note sheet printed (the 15-item checklist).
7. Stopwatch or timer app on facilitator phone.
8. Signed consent form ready on clipboard.
9. $20 gift card in envelope to hand over at the end.

**Environment:**

- Quiet room, neutral desk, chair at comfortable height.
- Lighting lit from the front so webcam captures expression.
- No other screens visible that might distract.
- Facilitator sits behind and to the side of the participant, out of their peripheral vision.

### Facilitator script

Read the following verbatim. Do not improvise. Do not answer questions mid-play. Do not hint.

> "Thanks for coming in today. Before we start, I'll ask you to sign this consent form — we're recording the screen and your face so we can study how people learn the game. You can stop any time for any reason and we'll still pay you.
>
> **[wait for signature]**
>
> Here's how this will work. I'm going to start a game on this computer. I want you to play it. There are no instructions. I'll be sitting quietly behind you and taking notes. If you ever feel stuck, try something else — I won't give you hints. If you truly need to ask me something, you can, and I'll say 'I can't answer that, try it and see.' When you feel done with the game, or want to quit, just tell me and we'll stop.
>
> One more thing: if you have a habit of thinking out loud, do it — we love hearing what you're wondering. If you don't, that's totally fine too. Just play normally.
>
> Ready? **[start screen recording, press Play]** Go ahead."

From this moment the facilitator does not speak until the participant quits, the 15-minute hard cap hits, or a red flag triggers.

If the participant asks a direct question, respond only with: **"I can't answer that during play — try it and see, and we can talk about it after."**

At the 15-minute mark (or earlier stop):

> "Okay, let's pause there. Great. I just have a few quick questions."

### Observer checklist

The observer silently time-stamps each item against the session clock. Goal: 15 items logged in order.

- [ ] **1. Time to first movement attempt** — when the player first presses a movement key.
- [ ] **2. Time to first attack attempt** — when the player first triggers LMB or any attack input.
- [ ] **3. Time to first parry discovery** — when the player first presses RMB or whatever they think parry is.
- [ ] **4. Time to first successful parry** — first enemy attack blocked with visible parry flash.
- [ ] **5. Time to first kill** — first enemy defeated.
- [ ] **6. Time to first death** — first time the player's HP reaches zero.
- [ ] **7. Does the player notice the Dice Tray?** (eyes dart to it, comments on it, or points at it)
- [ ] **8. Does the player *understand* the Dice Tray?** (verbalizes, e.g., "oh, that's my damage roll")
- [ ] **9. Does the player discover dodge?** (intentionally uses Space to avoid an attack)
- [ ] **10. Expression during combat** — laugh, focused frown, confused squint, bored slump. Code every 30 seconds.
- [ ] **11. Does the player speak out loud?** Log exact quotes verbatim when possible.
- [ ] **12. Does the player attempt to rage-quit?** (slams input, audibly swears, asks to stop)
- [ ] **13. Does the player ask for help, and about what?** Log the exact question.
- [ ] **14. Does the player restart after first death without prompting?** (Y/N + reaction)
- [ ] **15. Does parry success rate visibly improve?** Compare first 5 min vs last 5 min qualitatively.

### Post-session questions

Ask each question exactly as written. If the participant asks "what do you mean?", repeat the question verbatim — do not clarify or leading-question them. Write down answers word-for-word if possible.

1. "Describe what you were trying to do in that game."
2. "What did the dice on the screen mean to you?"
3. "Did combat feel fair? Why or why not?"
4. "If you played again, what would you do differently?"
5. "What did you NOT understand?"
6. "Would you want to play this game for 20 more minutes?"
7. "What does this game remind you of?"
8. "If you had to describe this to a friend in one sentence, what would you say?"
9. "Scale of 1 to 10, how much did you enjoy it?"

### Quantitative metrics

These should be automatically logged by the build where possible. The rest the observer logs by hand.

**Automatically logged (CSV, flushed per event):**

- `t_first_move` — seconds from start to first movement input.
- `t_first_attack` — seconds from start to first attack input.
- `t_first_parry_attempt` — seconds from start to first RMB press.
- `t_first_parry_success` — seconds from start to first successful parry.
- `t_first_kill`
- `t_first_death`
- `parry_attempts_per_minute` (rolling)
- `parry_success_rate_first_5min`
- `parry_success_rate_last_5min`
- `total_kills`
- `total_deaths`
- `session_length_sec` (until voluntary quit or hard cap)
- `input_events_per_minute` (proxy for confidence)

**Hand-logged by facilitator in post-session:**

- Enjoyment score (1–10, from Q9).
- Did the player understand the Dice Tray? (binary).
- Did the player discover dodge? (binary).
- Would play 20 more minutes? (binary from Q6).

### Red flags (halt or escalate)

These are conditions that, if observed, should trigger an immediate team discussion before running the next participant.

- **Parry not discovered in 5 minutes of active play** → teach-by-play is failing. The prototype may need floor prompts or an input-glyph overlay.
- **Dice Tray still not understood at 10 minutes** → UI is unclear. Reconsider its visual language (icons, animation, position).
- **Player enjoyment ≤ 5 at debrief** → core loop is not fun. Halt session and investigate before booking more participants.
- **Repeated deaths in first 30 seconds with no learning curve** → telegraphs too tight, hitboxes broken, or build is simply unfair. Stop the build and debug.
- **Player rage-quits before 5 minutes in** → the fail state feels punitive instead of playful. This is a structural concern, not a tuning concern.

### Pivot triggers

These are outcomes that would force a documented design change, not just a tuning tweak.

- **3+ of 5 players fail to discover parry** → add explicit floor-prompt tutorials in Prompt 2. Do not proceed to Prompt 2 playtesting until this is in place.
- **3+ of 5 players rate enjoyment ≤ 4** → redesign combat feel (hit-stop duration, screen shake, SFX punch, enemy telegraph length) before Prompt 2 is shown to anyone.
- **Average parry success rate < 25% by end of session** → parry window too tight. Loosen by 50 ms and re-test internally before next participant round.
- **0 of 5 players understand the Dice Tray** → visualization is fundamentally broken; redesign the dice tray from scratch, possibly drop it to a secondary display.

### Output format

**Per participant (1 page):**

- Participant ID (anonymized, e.g., P1-C, P1-M1, P1-H1)
- Profile tag (casual / moderate / heavy)
- Key timestamps (from quant metrics)
- 3 most important verbatim quotes
- Notable expression moments
- Facilitator narrative: 2–3 sentences describing the arc of the session
- Red flags triggered (Y/N + which ones)

**Summary report (1 document, shared with team):**

- Aggregate quant table (5 rows × ~12 columns).
- Trend line chart: parry success rate first 5 vs last 5 min, per participant.
- Enjoyment score distribution.
- List of pivot triggers hit (if any).
- Top 5 recommendations for Prompt 2, ranked.
- 1-paragraph "go / no-go / tune" decision on the core loop.

---

## Playtest Script 2 — Combat Feel Depth

### Purpose

Does combat feel satisfying? Do the three enemy types feel meaningfully distinct? Does the Dice Tray enhance or distract from the moment-to-moment experience? Is the hit-stop / VFX / audio recipe landing? And does gamepad play feel as good as mouse-and-keyboard?

Specifically we want to answer:

1. When asked to name what feels different about each enemy, do players describe them in different words?
2. Does parry timing feel learnable per enemy, or does it feel like one rhythm that sometimes works and sometimes doesn't?
3. Does the Dice Tray earn its place on screen, or does it get ignored / get in the way?
4. Which input method produces higher satisfaction?
5. Does combat hold up for 30 straight minutes without boredom?

### Participants

**Count:** 8 people.

**Profile mix:**

- 4 roguelite-experienced — have put serious hours into at least one of: Hades, Dead Cells, Slay the Spire, Enter the Gungeon, Hollow Knight, Returnal, Risk of Rain 2. These players can articulate "feel" with comparisons.
- 4 casual players — play occasionally, not action-specialists. Their job is to flag anything that feels unfair to non-experts.

**Input mix:**

- Ensure at least 3 participants actively prefer gamepad.
- Ensure at least 3 participants actively prefer keyboard-and-mouse.
- The remaining 2 are "whichever's handy".

**Demographic mix:** balanced gender where possible, age 18–45.

**Recruitment channels:** same as Script 1 plus roguelite subreddits and Discord servers for specific games (e.g., `r/HadesTheGame`, Dead Cells community Discord).

### Duration

**45 minutes per participant.**

- 5 min: arrival, consent, input method choice.
- 30 min: play session (with a soft break at 15 min to switch input methods).
- 10 min: debrief interview.

Book in 60-minute slots.

### Build target

**Prompt 2 — Multi-Enemy Conflict Prototype** (from the base prototypes deck).

Required scope in the build:

- All three enemy types:
  - **Orc** — medium speed, telegraphed overhead, parry window medium.
  - **Wolf** — fast, dash-bite, parry window short.
  - **Cyclops** — slow, heavy club, parry window long but big damage on fail.
- Dice Tray fully visible with per-enemy attack/defense dice.
- Full feel polish pass: hit-stop, screen shake, parry flash, SFX layer, VFX particles.
- **Gamepad fully supported** (XInput + DualSense bind map).
- Wave spawner: waves of increasing composition, no shop, no progression.
- Simple HP bar, no hunger, no shop, no boss.

Not required:

- Full dungeon layout, shop, map overlay, Dracular, economy.

### Pre-session setup

**Per session:**

1. PC with build pre-launched at the arena scene.
2. Both a mouse+keyboard station and an Xbox-style gamepad connected at the same time.
3. Webcam on face.
4. Screen recording.
5. Audio recording.
6. Input telemetry logging (which device is active per second).
7. Observer sheet with the 15-item checklist.
8. Timer.
9. Consent form, gift card ($20).

**Environment:**

- Same quiet desk setup as Script 1.
- Keyboard+mouse and gamepad both within easy reach, no cable tangles.
- Facilitator seated behind and slightly to the side.

### Facilitator script

Read verbatim.

> "Thanks for coming in today. Before we start, consent form please.
>
> **[wait for signature]**
>
> Here's how this will work. You'll see waves of enemies. Your job is simple: fight through as many waves as you can. This is not the whole game — it's a combat test. There are three different enemy types, see if you can handle all of them.
>
> You have both a mouse-and-keyboard and a gamepad in front of you. I want you to try both at some point in the next 30 minutes. You can switch whenever. I don't care which you prefer, I just want to see both.
>
> Here's the thing I'm really interested in: **talk to me out loud while you play.** Tell me if something feels great, if something feels frustrating, if something feels broken, if something just feels weird. Don't worry about being polite — I want the honest reaction. I'll listen but I won't interrupt.
>
> Around 15 minutes in I'll just quietly tap the desk — that's a reminder to switch inputs if you haven't yet. You don't have to switch immediately, just sometime after that.
>
> Ready? Go ahead."

At ~15 minutes, the facilitator taps the desk once. No verbal prompt.

At 30 minutes:

> "Okay, let's pause there. I have some questions."

### Observer checklist

- [ ] **1. Does the player switch inputs organically, or only when prompted?**
- [ ] **2. Does the player settle on one input method strongly?** (Log which.)
- [ ] **3. When does the player smile, laugh, or react positively?** Log timestamp + trigger.
- [ ] **4. When does the player groan, sigh, or react negatively?** Log timestamp + trigger.
- [ ] **5. Does the player adapt parry timing between Orc / Wolf / Cyclops?** (Does parry success rate against each improve?)
- [ ] **6. Does the Dice Tray cause any "whoa!" moments?** (Visible reaction, comment, pointing.)
- [ ] **7. Does the player understand the Guard / Opening cycle?** (Do they wait for the opening or mash?)
- [ ] **8. Does the player ever verbalize "feeling cheated" by a roll?** Log quote.
- [ ] **9. Does the player use dodge as a *conscious choice* or only reflexively?**
- [ ] **10. How does the player react to hit-stop?** (Doesn't notice / notices positively / complains it's too much.)
- [ ] **11. Does the player notice VFX polish?** (Comments on particles, flash, screen shake.)
- [ ] **12. Does the player notice audio?** (Comments on SFX, music, parry chime.)
- [ ] **13. Average wave duration.** (How long to clear a typical wave at steady state.)
- [ ] **14. Does boredom set in, and at roughly what minute?**
- [ ] **15. Does the player express a preference for a specific monster type?** (Which and why?)

### Post-session questions

1. "Of the three enemies, which one felt most distinct? Which one felt the most generic?"
2. "Describe the feeling of a good parry — like, the physical sensation."
3. "Did the dice on screen matter to you? How?"
4. "Which input method did you prefer, and why?"
5. "Were there moments you felt the game wasn't being fair? Talk me through one."
6. "What would make combat feel even better?"
7. "What would you remove from this build if you could remove one thing?"
8. "How long could you play this in a stretch — 5 minutes, 20, an hour?"
9. "Compare this to Hades, Dead Cells, and Enter the Gungeon. Which does it feel closest to?"
10. "Scale of 1 to 10, how satisfying is combat?"

### Quantitative metrics

**Automatically logged:**

- `waves_cleared_first_attempt`
- `parry_success_rate_by_enemy` — separate counters for Orc, Wolf, Cyclops.
- `time_to_kill_by_enemy` — mean TTK per enemy type.
- `dodge_usage_rate` — dodges per minute.
- `dodge_success_rate` — percentage of dodges that avoid damage.
- `input_device_share` — percentage of wall-clock time on M+K vs gamepad.
- `input_switch_count`
- `damage_taken_per_wave`
- `parry_to_attack_ratio` — are players parrying and attacking, or just attacking?

**Hand-logged from debrief:**

- Combat satisfaction score (1–10, from Q10).
- Preferred input (from Q4).
- Most-distinct enemy (from Q1).
- Comparative anchor (from Q9).

### Red flags

- **Average combat satisfaction < 6** → combat feel is unfinished. Do not move to Script 3 until this is addressed.
- **All three enemies feel "the same" to 4 or more players** → enemy distinction is unclear. The problem is at the behavior layer, not the art layer.
- **Dice Tray ignored entirely by 5 or more players** → the visualization is not earning its space.
- **Gamepad users consistently struggle** (>25% lower TTK on gamepad) → rebind or rework gamepad input before next iteration.
- **Player reports physical discomfort** (thumb fatigue, eye strain after 30 min) → input is too demanding or contrast is wrong.

### Pivot triggers

- **Combat satisfaction < 5** → redesign the hit-stop / telegraph / feedback recipe. This is a "feel sprint" before anything else moves.
- **Enemy distinction fails on the behavior level** → differentiate enemies by speed, attack shape, and commit time — not just parry window timing. Ship a new Prompt 2 variant.
- **Dice Tray perceived as clutter** → reduce visual prominence, move to HUD edge, or demote to a togglable overlay.
- **Gamepad is clearly worse than M+K** → redesign aim assist, reconsider twin-stick versus lock-on, or prioritize M+K as the "canonical" control.

### Output format

**Per participant (1 page):**

- Participant ID, profile, preferred input.
- Waves cleared.
- Parry success per enemy.
- Combat satisfaction score.
- Most distinct / least distinct enemy.
- 3 most important verbatim quotes.
- One "delight moment" timestamp.
- One "frustration moment" timestamp.

**Summary report:**

- Parry success rate matrix (enemy × participant).
- Combat satisfaction histogram.
- Input device share bar chart.
- Comparative anchor word cloud (Hades / Dead Cells / etc.).
- Top 5 feel-polish recommendations.
- Top 3 enemy-distinction recommendations.
- "Feel score" roll-up vs target (target: median 7/10).

---

## Playtest Script 3 — Economy & Pacing

### Purpose

Does a full 20-minute run arc work end-to-end? Is 20 minutes the right length? Does the Shop Shrine economy create meaningful decisions? Is the 90-second Dracular boss fight climactic — not anticlimactic, not unfair?

Specifically we want to answer:

1. Do players finish a run inside the 12–35 minute target window?
2. Does the Shop Shrine create real "which of these do I want?" decisions, or do players buy reflexively?
3. Does slot exclusivity surface as an interesting constraint?
4. Does the reachability warning actually help players avoid dead-ended runs?
5. Is the Dracular fight the emotional climax of the run, or is it just another room?
6. Does the player feel a "one more run" pull at the end?

### Participants

**Count:** 6 people.

**Profile:** roguelite audience — each participant has played at least one of Hades, Slay the Spire, Into the Breach, Dead Cells, Risk of Rain 2, or Inscryption for more than 10 hours. They have vocabulary for run pacing, meta progression, and build-around items.

**Demographic mix:** balanced gender where possible, age 18–45, mix of employment statuses so we don't only get the "I have 4 hours free on a Tuesday" profile.

**Recruitment channels:**

- Roguelite-focused Discord servers and subreddits.
- PlaytestCloud targeted to the genre.
- Word of mouth from Script 2 participants who self-identified as roguelite players (only if they did not see content from a later stage).

### Duration

**60 minutes per participant.**

- 5 min: arrival, consent, brief framing.
- Up to 50 min: play window — as many runs as they want.
- After each run: 3 quick questions (2 min).
- 10 min (approximate): final debrief interview.

Book in 75-minute slots.

### Build target

**Prompt 3 — Full Vertical Slice** or an Alpha build that includes:

- Full 9×10 procedurally seeded dungeon.
- All three enemy types + elite variants.
- Shop Shrine with full item pool and slot exclusivity rules.
- Map overlay toggle.
- Reachability warning system.
- 90-second Dracular boss fight with all phases.
- Run result screen (win / death / blocked).
- Full audio, music, ambient.
- Telemetry logging for every run.

### Pre-session setup

**Per session:**

1. PC with build pre-launched at main menu.
2. Participant's preferred input method (ask in advance, set up accordingly).
3. Webcam on face.
4. Screen recording.
5. Audio recording.
6. Telemetry log CSV per run.
7. Observer's checklist sheet.
8. Per-run 3-question card.
9. Consent form, gift card ($20/hr, so $20 for this session).
10. Water and a snack (hour-long sessions = be hospitable).

### Facilitator script

Read verbatim at session start.

> "Thanks for coming in. Consent form please.
>
> **[wait for signature]**
>
> This is the full game — well, an alpha, but it plays end to end. It's a roguelite dungeon crawler. Your goal is simple: try to beat the boss at the bottom of the dungeon. His name is Dracular. He is not forgiving.
>
> You can play as many runs as you want in the next 50 minutes. Most runs should take about 15 to 25 minutes, so you'll probably get through 2 or 3. That's fine. You can quit a run early if you want. You can also keep playing until the timer runs out.
>
> After each run — win, loss, or quit — I'll tap you on the shoulder and ask you 3 short questions. About 30 seconds. Then you can jump back in.
>
> Like last time: **think out loud while you play.** Strategy thoughts, frustrations, surprises — I want to hear it. I'll be quiet unless you ask me something.
>
> Final debrief at the end. Ready? Go."

Between runs, read the 3 inter-run questions verbatim (see below).

At the 50-minute mark (or when the participant declares they're done):

> "Alright, let's wrap that and do a final debrief."

**Inter-run questions (ask after every run, timestamped):**

1. "How do you feel about that run — win, loss, draw?"
2. "What was the single best thing that happened in it?"
3. "What's your plan for the next run?"

### Observer checklist

- [ ] **1. How many runs does the player complete?**
- [ ] **2. Time (in runs) to first Dracular kill.** (Null if they don't win.)
- [ ] **3. What routing strategy does the player use?** Code as: greedy farmer / straight runner / explorer / chaotic. Watch across runs — does it evolve?
- [ ] **4. Does the player visit the Shop Shrine on first encounter?** (Y/N per run.)
- [ ] **5. What items does the player buy first?** Log first 3 purchases per run.
- [ ] **6. Does the player understand slot exclusivity?** (Do they express surprise at being blocked, or do they plan around it?)
- [ ] **7. Does the player use the map overlay?** (Y/N + frequency.)
- [ ] **8. Does the player trigger the reachability warning?** Log when and why.
- [ ] **9. Does the player heed or ignore the reachability warning?** And does ignoring it backfire?
- [ ] **10. Run outcome distribution per participant:** death / blocked / victory.
- [ ] **11. Does the player express a sense of progression within a single run?** (Quotes like "I'm getting stronger" or "I have a build now.")
- [ ] **12. Does the player find Dracular challenging / trivial / impossible?** Log word choice.
- [ ] **13. Does the 20-minute run length feel right, too long, or too short?** (From behavior and comments, not just debrief.)
- [ ] **14. Does the player express the "one more run" impulse?** Especially right before they hit the time limit.
- [ ] **15. Does the player discover any exploits?** Infinite loops, stuck enemies, shop re-roll abuse, etc. Log exact repro.

### Post-session questions

1. "What was your strategy for picking rooms?"
2. "Did you feel like you understood the shop?"
3. "How did you decide what to buy?"
4. "Describe the Dracular fight. What was going through your head?"
5. "Did the run feel paced well? Too long? Too short? Uneven?"
6. "Was the ending climactic?"
7. "What felt most rewarding in a run?"
8. "What felt most frustrating?"
9. "Would you play this again later today, if I handed you the build?"
10. "Would you recommend this to a friend? Who?"
11. "Compare the feeling to Hades' Zagreus runs, Slay the Spire ascents, or Into the Breach islands. Where does this sit?"
12. "Pricing — what's a fair price for this game as a finished product?"

### Quantitative metrics

**Automatically logged per run:**

- `runs_played` (session total)
- `run_duration_sec` (per run, distribution target: median ~1200s, 80th pct 720–2100s)
- `run_outcome` (win / death / blocked / quit)
- `dracular_reach_rate` — % of runs that make it to the boss.
- `dracular_kill_rate` — % of reached runs that result in win.
- `dracular_kill_rate_overall` — % of all runs that end in win (target 20–40%).
- `shops_visited_per_run`
- `shop_items_bought_per_run`
- `shop_items_skipped_per_run`
- `slot_exclusivity_blocks_experienced`
- `map_overlay_toggles_per_run`
- `reachability_warnings_triggered_per_run`
- `reachability_warnings_heeded_rate`
- `rooms_explored_vs_rooms_skipped`

**Hand-logged from debrief:**

- Net Promoter Score derived from Q10 (0–10 scale).
- Would-pay-for price (Q12).
- Pacing rating (Q5, coded as too long / right / too short / uneven).
- "Climactic?" (Q6 as Y/N).

### Red flags

- **Dracular kill rate < 10% across all 6 participants** → boss too hard, game feels unwinnable. Do not ship this tuning.
- **Dracular kill rate > 75% across all 6 participants** → boss too easy, run is anticlimactic. Re-tune phases.
- **Median run duration > 35 min** → too slow, pacing is broken. Restructure dungeon size or enemy density.
- **Median run duration < 12 min** → too fast, player never feels the arc. Add friction or expand mid-dungeon.
- **NPS < 6** → the economy or progression is not landing emotionally. This is a design problem, not a polish problem.
- **Shop visited on fewer than 50% of possible occasions** → the shop isn't registering as interesting or isn't visible enough.
- **Player discovers an exploit that trivializes the run** → stop session, log exploit, patch before next participant.

### Pivot triggers

- **Pacing off by 30% or more in either direction** → restructure dungeon size (drop 9×10 to 7×8, or expand to 11×12) or adjust monster density.
- **Shop not used** → rethink shop placement (maybe guaranteed on every floor), pricing (maybe lower), or visibility (maybe the shrine animates/calls attention).
- **Boss too hard** → rebalance Dracular phase thresholds, damage output, or add more tells in Phase 2.
- **Boss too easy** → add a Phase 3 mechanic, or raise HP / add adds.
- **Routing ignored — players just go straight** → increase treasure/upgrade density in non-critical-path rooms to reward exploration. If that fails, reduce dungeon width so exploration and critical path overlap more.
- **Slot exclusivity frustrates players rather than interests them** → reconsider slot model; maybe reduce slot count or introduce a "swap" mechanic.
- **"One more run" impulse absent in 4+ players** → the run loop lacks a hook; add a per-run unlock or meta progression thread.

### Output format

**Per participant (2 pages):**

- Participant ID, profile, runs played.
- Per-run timeline table: outcome, duration, dracular reached, items bought.
- Routing strategy evolution.
- Key quotes (at least 5).
- NPS, pricing, pacing verdict.
- Red flags / pivot triggers hit.

**Per-session summary (1 doc):**

- Aggregate quant dashboard: run duration histogram, outcome pie, kill rate, shop behavior.
- NPS average + distribution.
- Pacing verdict distribution.
- Top 5 design decisions the team needs to make, ranked by evidence weight.
- Dracular tuning recommendation.
- Shop tuning recommendation.

**Team debrief meeting:**

- Held within 7 days of completing all 6 sessions.
- Attendees: design, code, art, audio leads.
- Output: a decision log with specific owners for each accepted pivot.

---

## Overall Playtest Protocol

The following rules apply to all three scripts and any future playtest added to the Solo Dungeon Dash program.

### Cadence

- Each script is run approximately every 2 weeks during the Alpha phase.
- The 3 scripts cycle in order: Core Loop → Combat Feel → Economy & Pacing → (repeat with fresh participants).
- If a script triggers a pivot, the cycle pauses for that script until the pivot is implemented and re-validated internally.

### Recording

With informed consent, every session captures:

- **Video:** webcam on face (not screen) for expression coding.
- **Audio:** room mic (not game audio) for verbal reactions.
- **Screen capture:** 1080p 30fps OBS recording of the game.
- **Player-click log / telemetry:** CSV of all inputs and key game events, timestamped.

Recordings are retained for 90 days then deleted. Aggregate metrics are retained indefinitely.

### Participant compensation

- **$20 gift card per hour** of the participant's time, rounded up.
  - Script 1: $20.
  - Script 2: $20 (45 min rounds up).
  - Script 3: $20.
- Paid at the end of the session regardless of whether they completed or quit.
- Gift cards are sourced from a neutral vendor (Amazon, local supermarket) — no game-store gift cards that might bias feedback.

### Ethics

- **Informed consent form** signed before any recording starts. The form covers: what's recorded, what it's used for, retention period, right to withdraw, right to anonymization.
- **Right to quit any time.** No social pressure. Explicit "you can stop whenever" delivered verbally in addition to the form.
- **Anonymized reports.** Participants are referred to by ID (e.g., P1-C, P2-H3). Real names never appear in reports.
- **No dark patterns.** No deception about what's being tested.
- **Accessibility.** If a participant has a disability or accessibility need we should support, ask in advance and accommodate (adjustable seat, screen reader if applicable, etc.).
- **Minors.** No participants under 18.
- **Content warning.** Solo Dungeon Dash contains fantasy combat and death — disclose on recruitment.

### Team review

- Playtest findings are reviewed in the **Monday team meeting** the week following each session.
- Format: 10-minute summary by facilitator, 10-minute discussion, 5-minute decision log.
- Any accepted pivot becomes an explicit work item in the sprint that follows.

### Backlog

- All issues, risks, and observations go into a shared **Playtest Findings** document (Google Doc or equivalent), **not** JIRA.
- This is intentional: we want low-friction capture during the Alpha phase.
- Items that become committed work items then get promoted into the real task tracker.
- The Playtest Findings doc is structured as:
  - Date / script / participant.
  - Observation or quote.
  - Severity (blocker / major / minor / delight).
  - Proposed action (or "no action").
  - Owner.
  - Status (open / accepted / closed / deferred).

### Facilitator standards

- Facilitators must be trained on these scripts before running a session.
- Facilitators never hint, never lead, and never defend the design during a session.
- Facilitators log everything they notice, not just what confirms their hypothesis.
- If the facilitator is also a designer on the feature being tested, a second person on the team should co-observe to counter bias.

### Session hygiene

- Never playtest on build day. Use a build frozen at least 24 hours before the first session of a round.
- Never change the build mid-round. If a bug is discovered, log it and keep the build stable.
- Never show the participant telemetry or quant metrics until after the full debrief.
- Always provide water and a bathroom break for sessions over 45 minutes.

### When to stop a session early

A facilitator must stop a session immediately if:

- The participant asks to stop.
- The participant appears visibly distressed.
- The build crashes and cannot be recovered.
- A red flag is triggered that invalidates the remaining session (e.g., Script 3 participant discovers a game-breaking exploit on Run 1).
- The participant exceeds their booked time — don't let sessions run long out of courtesy, it corrupts data.

---

**End of document.**
