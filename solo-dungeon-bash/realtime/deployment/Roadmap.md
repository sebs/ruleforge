# Development Roadmap — Solo Dungeon Dash

> RealTimeForge Stage RT-A8
> A conservative, honest development plan for a small indie team. Every phase carries a 25% buffer. Risks that could double the timeline are called out explicitly. If a phase slips more than 50%, the plan is to descope content — never quality.

---

## 1. Executive Summary

**Solo Dungeon Dash** is a 2D top-down action roguelite for PC and Steam Deck (primary) with mobile as a stretch goal. It ships in Godot 4, built by a small indie team of roughly 1 programmer + 1–2 artists + part-time audio + part-time design.

- **Prototype** (Phase 0): ~4 weeks from start. Internal only.
- **Alpha / Vertical Slice** (Phase 1): ~14 weeks after Prototype. Private pitch build, no sale.
- **Beta / Content Complete** (Phase 2): ~18 weeks after Alpha. Private beta.
- **Launch** (Phase 3): ~7 weeks after Beta. Steam + itch.io, PC + Steam Deck.
- **Post-Launch Support** (Phase 4): ~10 weeks after Launch. Free updates + cosmetic DLC.

**Realistic total timeline**: **~43 weeks (~10 months)** from Phase 0 start to launch, **ending roughly February 2027** for a Phase 0 start in mid-April 2026. Under adverse conditions, expect **up to 14 months**.

**Price point at launch**: $14.99 USD for PC/Steam/itch.io. No Day-1 mobile release — mobile is a Year-2 consideration.

**Target audience**: solo roguelite players, dice-game fans, people who loved *Inscryption*, *Slay the Spire*, *Hades*, and want a tighter session-length single-player roguelite with a handcrafted look.

---

## 2. Phases at a Glance

| Phase | Nominal | With 25% Buffer | Team Size | Deliverable | Gate Criteria |
|---|---|---|---|---|---|
| **0 — Prototype** | 3 weeks | **4 weeks** | 1 dev | Playable core loop (Prompts 1 + 2) | Combat feels good to 3 of 5 first-time testers |
| **1 — Alpha** | 10 weeks | **14 weeks** | 1 prog + 1 artist + 0.25 audio | Vertical slice end-to-end (Prompt 3) | 5-min pitch trailer possible from footage |
| **2 — Beta** | 14 weeks | **18 weeks** | 1 prog + 2 artists + 0.5 audio + 0.5 design + 0.5 QA | Feature-complete, all content in | Full playthrough, no crashes, playtest ≥ 7/10 |
| **3 — Polish & Launch** | 5 weeks | **7 weeks** | Beta team + marketing + external QA | Launched on Steam + itch | Steam cert passed, zero P0 bugs |
| **4 — Post-Launch** | 10 weeks | 10 weeks | 1 prog + 0.5 artist | 3 updates (1.1, 1.2, 1.3) | Community-driven |

**Cumulative buffered schedule** (Phase 0 → Phase 3 complete): **43 weeks ≈ 10 months**.

---

## 3. Phase 0 — Prototype

**Window**: Weeks 1–4 (weeks 1 & 2 = Prompt 1, weeks 3 & 4 = Prompt 2).
**Nominal**: 3 weeks. **With buffer**: 4 weeks.
**Team**: 1 programmer (full-time or focused weekends). Part-time designer (0.25 FTE) for playtest scripts and tuning sheets.

### Goal
Prove the 30-second core loop of Solo Dungeon Dash feels good. Build Prompts 1 and 2 from `PrototypePrompts.md`:

1. **Prompt 1 — Core Loop Prototype** (week 1–2): single enemy, player parry-attack-dodge, dice tray roll-and-count. Colored-square placeholder art. Acceptance: ≤ 50 ms input latency, parry window landable, clean victory/defeat loop.
2. **Prompt 2 — Conflict Prototype** (week 3–4): three enemy types (Orc/Wolf/Cyclops), hit-stop, screen shake, particles, gamepad, 7-wave arena, difficulty slider. Combat feel polish.

### Deliverable
A Godot 4 project that boots in under 5 seconds, lets a new player fight 7 waves of mixed enemies with readable telegraphs, and produces a standout "WHOA" multi-six moment at least once per 10 minutes of play.

### Gate Criteria
> **"Does combat feel good to 3 out of 5 first-time testers?"**

Take the prototype to 5 friends, family, or local devs. Ask them to play for 10 minutes each. Without leading, ask: "Did that feel good?" If 3 or more say yes without qualification, advance to Phase 1. If fewer, return to tuning and re-test.

### Activities
- Set up Godot 4 project, source control (Git + LFS for later art), CI build pipeline placeholder.
- Implement Prompt 1 scenes exactly (player, enemy, dice tray, HUD, main scene).
- Playtest Prompt 1 internally (dev + 1 friend, 3 sessions).
- Implement Prompt 2 on top (add enemy types, juice, gamepad, wave spawner).
- Run external playtest with 5 fresh eyes.
- Document every tuning tweak in a `TuningLog.md`.

### Risks
- **Combat doesn't feel good.** Mitigation: do not advance. Iterate on hit-stop, parry window, telegraph readability until the playtest gate passes. Expect 1–2 extra weeks of iteration — this is normal and is why Phase 0 has its own buffer.
- **Programmer-only team loses morale on placeholder art.** Mitigation: part-time designer provides "verbal art direction" at key moments so the dev can visualize where the polish is going.
- **Godot 4 upgrade friction.** If the team is new to Godot, add 3–5 days for ramp-up.

### What this phase does NOT include
- Hand-drawn art (colored squares only).
- Procedural dungeon.
- Shop.
- More than 3 enemy types.
- Save system.
- Menus beyond start/restart.

---

## 4. Phase 1 — Alpha / Vertical Slice

**Window**: Weeks 5–18. **Nominal**: 10 weeks. **With buffer**: 14 weeks.
**Team**: 1 programmer (1.0 FTE) + 1 artist (1.0 FTE) + part-time audio (0.25 FTE) + part-time designer (0.25 FTE).

### Goal
Build the Full Vertical Slice (Prompt 3 from `PrototypePrompts.md`) plus an AI/NPC Stress Test session. This is the pitch-demo build — the thing you show publishers, streamers, and the first press.

**Scope of the slice** (exactly per Prompt 3):
- Main Menu scene
- Tutorial Room (5-room scripted micro-run)
- 3×3 miniature dungeon (3 rooms + Shop Shrine + Dracular Phase 1)
- End Screen with run stats
- Hand-drawn ink-on-parchment visual style (finally — no more placeholders)
- 3 monster types (Orc, Wolf, Skeleton)
- 1 shop item (Big Sword, +1 AD, 3 Treasure)
- Dracular simplified to 1 phase, ~30 seconds
- Keyboard + gamepad + Steam Deck verified profile
- Zero crashes in 20 consecutive runs

### Deliverable
A 5-to-10-minute complete playable experience, end-to-end, in real art, shippable as a password-protected private build. A publisher, watching over the dev's shoulder or via a recorded trailer, can see the entire pitch: core loop, combat variety, art direction, shop, boss, win/loss. Download size ≤ 200 MB.

### Gate Criteria
> **"Does a 5-minute trailer cut from this footage show a complete, pitch-worthy vertical slice?"**

Record 30 minutes of gameplay. Cut it into 5 minutes. Show it to 3 external people (indie dev friends, publishers, streamers). If the response is "I'd play that" or "show me more," you pass. If it's "what genre is this?" you do not pass — return to polish.

### Activities

**Weeks 5–8 — Foundations in parallel**
- Programmer: migrate Prompt 2 code into a clean architecture (state machines, event bus, scene manager). Implement Main Menu, Tutorial Room framework, room-to-room transitions, seeded RNG, camera pan.
- Artist: start IMMEDIATELY at Week 1 (= project week 5). Produce style exploration: 5 reference ink-on-parchment scenes, then the player character (6 sprites), then the first monster (Orc ×4 sprites).
- Audio (part-time): source or compose the 3 music tracks (main menu, dungeon ambient, boss theme).
- Designer (part-time): lock `RT-BalanceSheet.md` numbers for the slice (HP, AD, DD, parry windows, telegraph durations, shop prices).

**Weeks 9–12 — Vertical slice integration**
- Programmer: Dungeon scene (3×3 grid, seeded), Shop Shrine interaction, Dracular Phase 1 fight, End Screen.
- Artist: finish Wolf and Skeleton sprites, Dracular sprites (higher detail), room tileset (octagonal chamber + door variants), particles, UI chrome, Main Menu parchment background.
- Audio: complete ~20 SFX pack, test crossfades.
- Designer: write and walk-through the Tutorial Room script from `RT-OnboardingDesign.md`.

**Weeks 13–14 — AI/NPC stress test + polish (buffer weeks)**
- Run the equivalent of an AI/NPC stress test session: spawn 20+ enemies in an arena, profile for 60 Hz on Steam Deck, fix bottlenecks.
- Steam Deck verified profile submission dry-run.
- Internal playtest (5 people) of the full slice, bug fix, polish.
- Record the pitch trailer.

### Risks

- **CRITICAL — Art bottleneck.** The artist has ~80 sprites in ~10 weeks. That is a tight but feasible pace for one full-time artist who has style nailed. If style iteration takes > 3 weeks, this phase slips by 2–4 weeks. **Mitigation**: start art at week 1 in parallel with code — do not wait for gameplay to "be ready first." Lock art style by end of week 6, no later.
- **Scope creep.** The team sees the slice coming to life and wants to add a second shop item, a second boss phase, a fourth monster. **Mitigation**: the producer/designer must enforce the Prompt 3 "Do not implement" list. Add to backlog, not to this slice.
- **Tutorial room complexity.** Teaching parry, dodge, dice tray, and shop in 5 minutes is hard. Expect to rewrite the script twice.
- **Audio delivery risk.** Part-time contractors are unreliable. Have 1 week of placeholder SFX ready as a fallback.

### Doubling risk
Alpha is the single biggest doubling risk in the entire plan. If the art style can't be nailed in 3 weeks OR the programmer has to rewrite the Prompt 2 code from scratch for architecture reasons, Phase 1 could balloon to 20–24 weeks. This is why the buffer is already +40%.

---

## 5. Phase 2 — Beta / Content Complete

**Window**: Weeks 19–36. **Nominal**: 14 weeks. **With buffer**: 18 weeks.
**Team**: 1 programmer (1.0 FTE) + 2 artists (2.0 FTE) + part-time audio (0.5 FTE) + part-time designer (0.5 FTE) + part-time QA (0.5 FTE).

### Goal
Scale the vertical slice into a full, content-complete game. Everything in the RTGDD that is in scope ships now.

**Content to add on top of the Alpha slice**:
- **All 11 monster types** (Orc, Wolf, Cyclops, Skeleton, Orc Warrior, Witch, Mummy, Vampire Bat, Plant Crawler, Red Dragon, Ghost). Alpha had 3 — Beta adds 8.
- **Full 9×10 seeded dungeon** instead of the 3×3 slice. Seeding, king-adjacency movement, biome variety if affordable.
- **Dracular 3-phase boss fight** instead of Phase 1 only. All 3 phases tuned to ~90 seconds total for a competent player.
- **All shop items** (~9 per the source material). Alpha had 1 (Big Sword) — Beta adds the rest.
- **Multiple Shop Shrines** per dungeon if the design calls for it.
- **Full Tutorial polish** based on Alpha playtest feedback.
- **Save / resume mid-run** (was explicitly descoped in Alpha).
- **Settings menu**: volume, fullscreen, brightness, input remapping, accessibility (colorblind, text size, motion reduction).
- **Cosmetics framework** (no content yet — just the plumbing).
- **Bestiary** unlocked by first-kill.
- **Achievements** (internal only; Steam achievements plug in at Phase 3).
- **Daily Seed** run mode.
- **Run History** and stats screen.
- **Analytics opt-in** plumbing.
- **Localization hooks** (English only, but strings externalized for future translation).

### Deliverable
A feature-complete Solo Dungeon Dash, all content present, all UI polished, ready for a private closed beta with 20–50 external playtesters.

### Gate Criteria
> **"Does the full game play through without crashes, and does the average playtest rating hit ≥ 7/10 across at least 10 testers?"**

Run a closed beta for 2 weeks. Collect NPS, ratings, qualitative feedback, bug reports. If average rating < 7/10 OR NPS < 4, delay launch until fixable. Do not ship a 6/10 game.

### Activities

**Weeks 19–24 — Content production (monsters + dungeon)**
- Programmer: full 9×10 dungeon generator, biome variety, room template library, save/resume system, settings menu scaffolding.
- Artist 1 (original): remaining 8 monster sprites, Dracular Phase 2 & 3 visuals, new biome tilesets.
- Artist 2 (new): UI pass (all menus, HUD polish, achievement icons, cosmetics framework art), 9 shop item icons, run stats screen illustrations.
- Audio: ~30 more SFX (new monsters, new biomes, UI), 2 more music tracks (biome variations, Dracular Phase 2/3 intensification).
- Designer: re-tune balance from Alpha playtest data, lock progression curve.

**Weeks 25–30 — Systems + polish**
- Save/resume system, daily seed mode, bestiary, run history, analytics plumbing, localization pipeline, accessibility features (from `/accessibility-audit` checklist — colorblind palette, text scaling, motion reduction, input remapping).
- QA (part-time): write regression checklist, begin daily smoke testing.
- Steam Deck verified badge: submit application.

**Weeks 31–34 — Closed beta**
- Ship beta build to 20–50 external testers via Steam key distribution.
- Daily patch cadence during beta.
- Collect feedback, fix P0/P1 bugs, rebalance content.
- Designer runs a playtest telemetry analysis from the analytics opt-in data.

**Weeks 35–36 — Stabilization buffer**
- Final bug fix pass.
- Lock content.
- Prepare Phase 3 launch assets (Steam page, trailer, screenshots).

### Risks

- **CRITICAL — Content fatigue on the art side.** 8 additional monsters × 4 sprites each + biome tilesets + UI polish is a LOT of art in 14–18 weeks. **Mitigation**: tight scope discipline; the second artist is hired specifically to absorb UI/icons so Artist 1 can stay on monsters and environments; no feature creep permitted.
- **Save system regressions.** Adding mid-run save late is notoriously bug-prone. **Mitigation**: the save system must land by week 24, not week 30. Plan it before you plan cosmetics.
- **Beta tester recruitment.** Getting 20–50 engaged beta testers is harder than it sounds. **Mitigation**: start building a Discord community during Phase 1 and recruit from it now.
- **Localization scope creep.** English-only at launch is a hard rule. Mitigation: strings are externalized for future translation but no other languages ship.
- **Playtest reveals fundamental design flaw.** If the 9×10 dungeon paces poorly where the 3×3 slice felt tight, you may need to revisit dungeon generation rules. Budget 1 week of buffer for this.

### Doubling risk
If the beta playtest is harsh (< 6/10 average), Phase 2 could extend by 4–8 weeks. This is survivable. If the extension exceeds 8 weeks, descope content (remove 2 monsters, remove 2 shop items) rather than delay launch further.

---

## 6. Phase 3 — Polish & Launch

**Window**: Weeks 37–43. **Nominal**: 5 weeks. **With buffer**: 7 weeks.
**Team**: Full Beta team + marketing help + external QA (1.0 FTE during this phase).

### Goal
Ship Solo Dungeon Dash on Steam and itch.io. Make it stable, marketable, and legally clear. Nothing new ships in this phase — only fixes, polish, and release infrastructure.

### Deliverable
- Launched game on Steam (PC, Steam Deck verified)
- Launched game on itch.io (PC)
- Steam page live with trailer, screenshots, description, press kit
- Press & streamer keys distributed
- Launch trailer on YouTube
- Discord community active
- Day-1 patch queued

### Gate Criteria
> **"Has the game passed Steam's certification process and hit zero P0 (crash / data loss) bugs in the final smoke test?"**

No P0 bugs. No data loss. No crashes in a 20-run smoke test on a Steam Deck AND a mid-spec PC. If either fails, delay launch by 1 week and fix.

### Activities

**Weeks 37–38 — Release engineering**
- Steam store page finalization: description, tags, trailer embed, screenshots (10 required), capsule art.
- itch.io store page finalization.
- Press kit on presskit() or custom static page.
- Age rating submission (likely PEGI 7 / ESRB E10+).
- Steam build upload, cert submission.
- External QA pass on current branch.

**Weeks 39–40 — Bug fix + polish sprint**
- Address every P0/P1 bug from external QA.
- Polish pass: screen transitions, audio balance, menu feel, controller glyphs per platform.
- Translate (optional, scope permitting) UI strings into 1 additional language (likely German or French) — only if time allows.
- Day-1 patch testing: build a known-bug patch in advance so it can ship the moment Steam is live.

**Weeks 41–42 — Marketing ramp**
- Key distribution: ~200 keys to press/streamers via Keymailer or similar.
- Launch trailer on YouTube, embedded on Steam page.
- Social media posts scheduled across launch week.
- Discord announcements, wishlist push emails.
- Localization status: English first, expand later.

**Week 43 — Launch week**
- Launch day: daily hotfix cadence, monitor reviews, respond to critical feedback.
- Stream the launch if morale permits.

### Risks
- **Steam certification delays.** Plan for 5–10 business days for Steam review and possible revision loops.
- **Day-1 patch mismatch.** Ensure day-1 patch is built AGAINST the same branch Steam certified.
- **Marketing underperform.** If wishlists are < 3000 at launch week, push the launch 2 weeks and run a "free demo on Steam Next Fest" marketing beat.
- **Single-threaded marketing person.** Part-time marketing is risky. Consider a 4-week full-time contractor if budget allows.

---

## 7. Phase 4 — Post-Launch Support

**Window**: Weeks 44–53 (and beyond). **Nominal**: 10 weeks.
**Team**: 1 programmer + part-time artist (0.5 FTE) + part-time marketing (0.5 FTE).

### Goal
Keep the community happy. Ship three planned updates over 3 months after launch.

### Deliverables

- **Update 1.1 — "Ghost Races"** (Week 46): Daily Seed leaderboards, Ghost Race mode (race your own or friends' best runs), run replay viewer. Free.
- **Update 1.2 — "Quality of Life"** (Week 49): accessibility improvements based on launch feedback, input remapping polish, performance fixes, bestiary entries reworked based on community favorites, new "hard mode" difficulty. Free.
- **Update 1.3 — "Cosmetic Parchment Pack"** (Week 52): paid cosmetic DLC ($2.99): 10 alternate parchment backgrounds, 10 dice skins, 5 hat cosmetics for the player. First revenue bump beyond base sales.

### Activities

**Week 44 (Launch + 1)**: daily hotfix cadence. Monitor Steam reviews. Respond to P0/P1 bugs only.

**Weeks 45–46**: build 1.1, playtest internally, ship.

**Weeks 47–49**: build 1.2 based on launch feedback priorities (not pre-planned features — let the community dictate).

**Weeks 50–52**: build 1.3 cosmetic pack with artist, set up Steam DLC SKU.

**Week 53+**: evaluate whether to extend support, build a major content update (Month 6), or move on to the next project.

---

## 8. Total Timeline

**From Phase 0 start to Phase 3 Launch (buffered, conservative):**

| From | To | Weeks |
|---|---|---|
| Phase 0 start | Phase 0 end | 4 |
| Phase 1 start | Phase 1 end | 14 |
| Phase 2 start | Phase 2 end | 18 |
| Phase 3 start | Launch | 7 |
| **Total** | **Launch** | **43 weeks ≈ 10 months** |

**Plus Phase 4 (post-launch) 10 weeks → ~53 weeks ≈ 12 months from start to 1.3 DLC.**

### Honest range
- **Best case** (everything clicks): **9 months** to launch.
- **Realistic case** (the estimate above): **10 months** to launch.
- **Adverse case** (art style iteration slow, Alpha playtest fails once, beta playtests harsh): **14 months** to launch.

The plan assumes the ideal team stays intact. If the programmer or lead artist leaves mid-project, add 6–10 weeks for replacement + onboarding.

**Assuming Phase 0 starts 2026-04-15, realistic Launch date is around 2027-02-10. Adverse-case Launch date is 2027-06-10.**

---

## 9. Team Composition Timeline

FTE = Full-Time Equivalent. 0.5 = half-time, 0.25 = quarter-time / one day a week.

| Phase | Programmer | Artist 1 | Artist 2 | Audio | Designer | QA | Marketing |
|---|---|---|---|---|---|---|---|
| **0 — Prototype** | 1.0 | 0 | 0 | 0 | 0.25 | 0 | 0 |
| **1 — Alpha** | 1.0 | 1.0 | 0 | 0.25 | 0.25 | 0 | 0 |
| **2 — Beta** | 1.0 | 1.0 | 1.0 | 0.5 | 0.5 | 0.5 | 0.25 |
| **3 — Launch** | 1.0 | 1.0 | 1.0 | 0.25 | 0.25 | 1.0 | 1.0 |
| **4 — Post-Launch** | 1.0 | 0.5 | 0 | 0.1 | 0 | 0.25 | 0.5 |

### Observations
- Artist 2 enters at Beta, not Alpha. This is deliberate: one artist establishes the style; the second absorbs UI/icons/cosmetics later.
- QA ramps at Beta (0.5) and peaks at Launch (1.0 — external).
- Designer is part-time throughout. No single full-time designer role — the programmer and lead artist share design ownership.
- Marketing ramps hard only at Launch.
- Post-launch shrinks the team — this is intentional and sustainable.

---

## 10. Budget Rough Estimate

**For transparency. All numbers are USD. Ranges reflect contractor-rate uncertainty and team structure (founder-led vs. fully-paid).**

### Fully-paid scenario (no founder subsidies; 9 months at 2–3 average devs)

| Line item | Low | High | Notes |
|---|---|---|---|
| Labor (programmer, full team) | $120,000 | $200,000 | 10 months at $12k–$20k/month blended rate |
| Art contractors (Artist 1 + Artist 2 part-time) | $15,000 | $40,000 | Depends on hourly rate + sprite count |
| Audio contractor (part-time) | $5,000 | $15,000 | ~50 SFX + 5 music tracks |
| Tools / engine | $500 | $1,500 | Godot free; misc plugins, asset packs, Aseprite, Reaper |
| Marketing | $10,000 | $30,000 | Keymailer, trailer edit, festival fees, ads |
| Steam fee | $100 | $100 | Steam Direct one-time |
| Localization (optional, launch+) | $0 | $5,000 | English only at launch |
| Legal / LLC / accounting | $2,000 | $5,000 | One-time |
| Contingency (15%) | $23,000 | $43,000 | For the inevitable |
| **TOTAL** | **~$175,000** | **~$340,000** | |

### Founder-led scenario (1 founder-programmer unpaid; contractors paid)

| Line item | Low | High |
|---|---|---|
| Founder labor | $0 | $0 |
| Art contractors | $10,000 | $25,000 |
| Audio contractor | $3,000 | $10,000 |
| Tools | $500 | $1,500 |
| Marketing | $3,000 | $10,000 |
| Steam + legal | $2,100 | $5,100 |
| **TOTAL** | **~$19,000** | **~$52,000** |

**Recommendation**: if this is a founder-led hobby project, the founder-led scenario is realistic. If this is VC-backed or publisher-funded, plan for $200k+. Do not start the project without clarity on which scenario applies.

### What each phase costs (fully-paid scenario, midpoint)

| Phase | Weeks | Cost midpoint |
|---|---|---|
| Phase 0 | 4 | ~$15,000 |
| Phase 1 | 14 | ~$75,000 |
| Phase 2 | 18 | ~$120,000 |
| Phase 3 | 7 | ~$40,000 |
| Phase 4 | 10 | ~$25,000 |
| **Total** | **53** | **~$275,000** |

---

## 11. Day-1 Launch Checklist

**Must be done the week of launch. Check each box before shipping.**

### Store & presence
- [ ] Steam page live with final trailer, 10+ screenshots, description, tags, capsule art
- [ ] itch.io page live with matching assets
- [ ] Press kit uploaded to presskit() or custom static page
- [ ] Launch trailer on YouTube (public, unlisted removed)
- [ ] Social media posts scheduled for launch day + launch week
- [ ] Discord server set up, rules posted, mods assigned

### Build & platform
- [ ] Build passes Steam certification (all checks green)
- [ ] Day-1 patch tested on the cert'd branch, ready to push hour-1 if needed
- [ ] Save system tested: no data loss across app restart, across Steam Cloud sync, across OS update
- [ ] Cross-platform save sync (Steam Cloud) tested on at least 2 machines
- [ ] Steam Deck verified badge obtained
- [ ] Windows, Linux, macOS builds all green (only PC+Deck ships, but Linux is near-free via Godot)
- [ ] Installer size ≤ 300 MB

### Community & press
- [ ] ~200 press & streamer keys distributed via Keymailer or manual
- [ ] Top 20 streamer list identified and direct-DM'd
- [ ] Review embargo (if any) communicated
- [ ] Launch-day livestream scheduled (founder or partner)

### Compliance
- [ ] Age rating obtained (PEGI 7 / ESRB E10+, or similar)
- [ ] Privacy policy & terms of service live
- [ ] Analytics opt-in flow working (GDPR-compliant if EU)
- [ ] Refund policy reviewed (Steam auto-applies — just confirm no issues)
- [ ] Localization status: English confirmed; other languages "Coming Soon" if promised

### Operations
- [ ] Bug tracker ready for post-launch reports (GitHub Issues or similar)
- [ ] Support email live and monitored
- [ ] Discord moderation coverage for launch day
- [ ] Backup of the final gold master on 2 separate drives

---

## 12. Post-Launch Support Plan

**The 3-month support window after launch. Discipline is more important than ambition here.**

- **Week 1** (Launch): **daily hotfixes**. P0 and P1 bugs only. No new features, no balance tweaks unless game-breaking.
- **Weeks 2–4**: **weekly patches**. Continue P0/P1 fixes. Start integrating the most-requested QoL ideas from the community.
- **Week 6** (Month 2): ship **Update 1.1 — Ghost Races**. Daily Seed leaderboards + replay viewer.
- **Weeks 5–12**: **monthly updates**. Pace sustainable. Respond to community, no feature-fatigue on the team.
- **Week 9** (Month 3): ship **Update 1.2 — Quality of Life**. Community-dictated improvements.
- **Week 12** (Month 3): ship **Update 1.3 — Cosmetic Parchment Pack** ($2.99 DLC).
- **Month 6**: evaluate whether to ship a first major content update (new boss? new biome?). If sales justify it, proceed. If not, shift to maintenance mode.
- **Month 12**: evaluate co-op DLC as a separate project. This was explicitly scoped OUT of the base game (single-player only). Only build it if Year 1 revenue + community demand justify it.

---

## 13. Success Metrics

**Measured honestly, reported publicly to the team. These are small-indie targets, not AAA.**

| Metric | Minimum | Target | Stretch |
|---|---|---|---|
| Steam wishlists at launch | 3,000 | **5,000** | 10,000 |
| Week 1 sales | 800 | **1,500** | 3,000 |
| Month 1 sales | 3,000 | **5,000** | 10,000 |
| Year 1 revenue | $30,000 | **$50,000** | $150,000 |
| Steam review score | 75% positive | **80% positive** | 90% positive |
| Median playtime per owner | 2 hours | **3 hours** | 6 hours |
| % players who defeat Dracular at least once | 30% | **50%** | 70% |
| Daily Seed engagement (Month 2+) | 5% DAU | **10% DAU** | 20% DAU |

**Small indie success** = hitting the Target column. If most Targets hit, the next game has a runway. If most fall to Minimum, the team can continue but should reconsider scope. If most fall below Minimum, see Section 14.

---

## 14. When to Cancel — Honest Criteria

**The hardest section. Read it before Phase 0 starts, not after Phase 2.**

### Prototype phase (Phase 0)
**Trigger**: Combat doesn't prove fun after 2 playtest rounds and 1 iteration cycle.
**Response**: **Pause and re-scope.** Do not advance to Alpha. Either (a) find what combat change unlocks the fun, or (b) re-examine whether Solo Dungeon Dash's core premise is viable as a digital real-time game. It is honest and cheap to stop here. It is extremely expensive to stop later.

### Alpha phase (Phase 1)
**Trigger**: Alpha slips by more than 50% (21 weeks instead of 14) AND the vertical slice is not gate-passable.
**Response**: **Descope content, not quality.** Drop the Tutorial Room to a single scripted fight. Drop to 2 monster types. Drop the Shop Shrine to a single item list. Ship the vertical slice SMALLER rather than LATER. The pitch trailer just needs to show that the game is fun and looks distinctive.

### Beta phase (Phase 2)
**Trigger**: Beta playtests show NPS < 4 or average rating < 6/10.
**Response**: **Delay launch until fixable.** Do not ship a mediocre game. Identify the single biggest issue (usually pacing, difficulty curve, or tutorial). Fix that one thing. Re-test. Repeat. If after 3 iteration cycles the rating does not rise above 7/10, accept that the game ships as a modest niche title and adjust marketing + price accordingly (drop to $9.99, lean hard into the "handcrafted roguelite" niche, manage expectations).

### Launch phase (Phase 3)
**Trigger**: Launch flops — fewer than 500 Week-1 sales and fewer than 1,500 Month-1 sales.
**Response**: **Shift to maintenance mode.** Ship 1.1 and 1.2 as planned to honor the community that did buy. Do NOT ship 1.3 cosmetic DLC — it will not pay back its dev cost. Do NOT build the co-op DLC at Month 12. Move on to the next project; treat Solo Dungeon Dash as a credential-builder rather than a revenue-driver. This outcome does not mean the game was bad or the team failed — it means the market was not there, and that's information for the next project.

### Any phase
**Trigger**: Core team member departure (programmer or lead artist).
**Response**: Pause the phase. Budget 6–10 weeks for replacement + onboarding. Re-evaluate whether to continue or sunset the project based on the new team's capacity.

### What "cancel" really means
Cancellation is not failure. It is the honest response to information. The expensive failure mode is knowing the project is in trouble and spending another 6 months hoping things improve. Set the gate criteria, honor them, and move with speed when they are not met.

---

## Appendix A — File Dependencies

The roadmap assumes these files from earlier pipeline stages exist and are referenced:

- `design/RTGDD.md` — game design source of truth
- `design/PrototypePrompts.md` — Prompts 1, 2, 3 (the exact scope of Phases 0 and 1)
- `revised/RT-Features.md` — feature list for scope tracking
- `revised/RT-BalanceSheet.md` — tunable parameters for all phases
- `revised/RT-OnboardingDesign.md` — tutorial script
- `analysis/GenreCrystallization.md` — genre positioning (2D top-down action roguelite)
- `analysis/AgencyModel.md` — combat timing rationale
- `analysis/ConflictModel.md` — combat FSM rationale
- `architecture/SystemArchitecture.md` (if produced by RT-A2)
- `assets/AssetPipeline.md` (if produced by RT-A3)
- `prototypes/ExtendedPrototypes.md` (if produced by RT-A4)

## Appendix B — Milestones Summary

| Milestone | Week | Date (from 2026-04-15 start) |
|---|---|---|
| Phase 0 start | 1 | 2026-04-15 |
| **Playable Prototype complete** | 4 | 2026-05-13 |
| Phase 1 start | 5 | 2026-05-14 |
| **Vertical Slice complete (Alpha Gate)** | 18 | 2026-08-19 |
| Phase 2 start | 19 | 2026-08-20 |
| **Content Complete (Beta Gate)** | 36 | 2026-12-16 |
| Phase 3 start | 37 | 2026-12-17 |
| **Launch** | 43 | 2027-02-03 |
| **Update 1.1 (Ghost Races)** | 46 | 2027-02-24 |
| **Update 1.2 (QoL)** | 49 | 2027-03-17 |
| **Update 1.3 (Cosmetic DLC)** | 52 | 2027-04-07 |
| Month 6 evaluation | ~62 | ~2027-06-16 |

---

## Closing note

This roadmap is conservative on purpose. Every week listed is a real week with real humans working on real tasks. If the team is small and the ambition is honest, 10 months from Phase 0 to launch is tight but achievable. If the team rushes or cuts buffer, the likely outcome is a 14-month slip that feels worse because it wasn't planned for. **Build slow, ship honest, cancel cleanly if you must.**
