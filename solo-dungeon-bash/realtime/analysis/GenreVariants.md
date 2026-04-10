# Genre Variants — Solo Dungeon Bash → Real-Time

> RealTimeForge Stage RT-A1: Alternative genre interpretations of Solo Dungeon Bash.
> Primary genre is locked as **2D top-down Action Roguelite / Dungeon Crawler** (see `GenreCrystallization.md`).
> This document explores three alternative framings to stress-test the primary and to surface cross-pollination opportunities.

---

## Introduction — Why Explore Variants?

The primary genre of *Solo Dungeon Dash* — a 2D top-down Action Roguelite with Roll-and-Parry combat — is the result of a Wave 1 synthesis that balanced six design axes (temporal, spatial, agency, info, conflict, economy). It sits at ~93% faithfulness to the source Solo Dungeon Bash rulebook. It is the right answer for the market we want (Hades fans, Dead Cells fans, print-and-play nostalgics), and it correctly honors the three identity-triangle corners: Routing Puzzle, Parry Combat, and Dice Tray.

But "the right answer" is not the *only* answer, and the discipline of RealTimeForge demands that we seriously prototype three alternatives before locking the primary. Three reasons:

1. **Alternative genres expose hidden identity.** When we force the game into a tower-defense frame, we learn which features actually belong to "Solo Dungeon Bash" and which belong to "any dungeon game." If a feature survives three genre shifts, it's core. If it collapses in the first shift, it was never essential.
2. **Variants surface cross-pollination opportunities.** Features from an unrealized variant can ship as optional modes in the primary. A "Draw-Only" puzzle mode, a "Chaos" auto-battle mode — these are cheap additions that expand the primary's audience without diluting it.
3. **Variants hedge against market misreads.** If Action Roguelite saturation becomes a serious concern by Q3 of development, having a credible Tower Defense or Auto-Battler design in the drawer lets us pivot instead of restart. This document is insurance.

Three variants follow. Variant A is the "Unexpected Genre" flagged in RT-6 of the crystallization — the meditative puzzle-platformer hiding inside the source. Variant B is a "Flip The Lens" reinterpretation — you play as Dracular in a turn-based tower defense. Variant C is a "Mechanic Transplant" — the dice identity is preserved but the routing is sacrificed for auto-battler loadout strategy. Each is designed in enough detail to be buildable; each is then compared against the primary on faithfulness, market fit, risk, and scope.

A comparison table and a final recommendation close the document.

---

## Variant A — "Dungeon Solitaire"

### Working Title
**Ink & Dice** *(also considered: Parchment Pocket, Dice Pilgrim, The Slow Dungeon, Sixes & Ink)*

### Primary Genre
**Cozy puzzle / meditative puzzle-platformer** in the tradition of *Threes!*, *Mini Metro*, *A Monster's Expedition*, *Dorfromantik*, and *Into the Breach* (for commitment-weight, not pace). Secondary tag: "dice-cascade puzzler." Not action, not combat-centric, not twitchy. The core verb is **carve**, as in the player's statement from the primary — but here "carve" is taken literally, not metaphorically. The game is about drawing a line through paper.

### Core Loop Description

A run is ~10–20 minutes and consists of three nested rhythms:

**Atomic loop (5–15 seconds each):**
1. Player taps a dungeon cell adjacent to their current cell (king-move, 8 neighbors as in the source).
2. A path segment is drawn, in ink, from the current cell to the new one, with a satisfying pen-stroke sound and a small ink-splatter particle effect.
3. The contents of that cell reveal: treasure, potion, trap, monster, stairway, or shrine.
4. If no combat is triggered, the player exhales and can tap the next cell whenever they're ready. There is no timer. The coffee-morning feel is sacred.

**Primary loop (60–180 seconds each):**
1. A room with a monster triggers the **Dice Pouch Puzzle**. The monster's dice pattern is displayed above its sprite as a row of ghostly d6 silhouettes (e.g., "needs two 5s, one 6"). The player's dice pouch — their permanent dice inventory — is laid out below as drag-and-drop tokens.
2. The player drags dice from the pouch onto "attack slots" and "defense slots." There is no rolling. Every die in the pouch shows its current face. Rerolls cost a resource (ink?). Matching dice faces to monster requirements resolves the fight.
3. The fight ends. Dice are returned to the pouch, possibly in new orientations, possibly exhausted for the rest of the run. Health (if used at all — see below) adjusts.
4. The player resumes the atomic loop, now one monster lighter and one puzzle wiser.

**Secondary loop (10–20 minutes, whole run):**
1. Start in the North room with an empty parchment and a starter pouch of ~5 dice.
2. Carve a path through the 9×10 grid, revealing cells, collecting dice, solving dice puzzles.
3. Reach the South room (Dracular's chamber). Face Dracular as a three-phase dice puzzle — the boss has three dice patterns you must solve in sequence.
4. Win: the parchment is saved to a gallery. The player can scroll through past runs and admire the ink-work. Lose: the parchment is also saved, but with a grey "Fell Here" marker. No run is wasted; every run makes a picture.

**Meta loop (cross-run):**
- Gallery of past runs. Cosmetic ink colors unlock. Different parchment textures unlock. A daily seed lets friends compare parchments.
- No stat progression, no permanent buffs, no meta-unlocks that affect gameplay. The only progression is aesthetic.

### What Changes From the Primary (SDD Action Roguelite)

- **Combat verb: parry → match.** There is no real-time parry window, no dodge roll, no hit-stop. Combat becomes a dice-matching puzzle solved at the player's leisure.
- **Temporal model: beat-based real-time → fully turn-based, player-paced.** Nothing happens until the player taps. No monster AI tick, no cooldowns, no refresh timers. A player can put the phone down mid-run and come back three hours later with the game state intact.
- **Spatial model: room-at-a-time camera → whole-dungeon always-visible overhead.** The player sees the entire 9×10 parchment at once. The ink-on-paper feel is the primary visual pleasure.
- **HP model: 17 discrete hearts → simplified to a Luck pool of ~5, or removed entirely.** In a pure puzzle framing, "HP" becomes "how many more bad dice rolls can I tolerate." Some variants remove HP and make the game a pure path-puzzle where you must *solve* every monster or accept a movement penalty.
- **Dice tray: animated side-panel → the whole game.** The dice are the center of the UI, the center of the verb, and the center of the visual identity. They are drawn as hand-sketched d6 tokens with ink shading.
- **Input: M+KB + gamepad → single-finger touch, mobile-first.** Every interaction is a tap or a drag. No keyboard bindings, no modifier keys.
- **Art style: hand-drawn ink-on-parchment → hand-drawn ink-on-parchment but lighter and more colorful.** Instead of the slightly moody, Dark Souls-of-parchment aesthetic of the primary, this variant leans into Threes!-style clean readability: soft cream paper, gentle pastels for accents, cheerful sound design, lo-fi piano soundtrack.
- **Audio: combat hit-stops and riposte clinks → pen-stroke Foley, paper-rustle, dice-clack, ambient rain.** The whole audio design pivots toward ASMR.
- **Session length: 20 min median → 10–15 min median for a casual run, up to 30 min for a thoughtful solver.**
- **Failure handling: death-resets-the-run → "fell here" marker, run is still archived to the gallery.** Loss is gentler. The game never punishes you harshly because the pleasure is the drawing, not the winning.

### What Breaks (Faithfulness Sacrifices)

- **The Parry Combat corner of the identity triangle collapses.** There is no parry, no dodge, no timing. This is a 33% loss of the primary's identity right here.
- **The "roll" in "Roll-and-Parry" is softened.** Dice no longer tumble unpredictably; they are placed. The stochastic joy of a surprise d6 result is reduced to a reroll mechanic.
- **The "dungeon crawler" promise of the name is diluted.** This is a puzzle game with dungeon flavor, not a dungeon crawler. Marketing must reposition.
- **The lethality identity (1 HP monsters, bursty damage) is softened** — when there's no timing pressure, lethality stops feeling dangerous and starts feeling like a math problem.
- **Routing Puzzle corner is weakened but not broken.** The no-revisit rule and king-adjacency are preserved. The block-detection warning is still useful. But the *urgency* of routing under combat pressure is gone. You have all the time in the world to plan.
- **Shop Shrine intensity drops.** Without combat-adjacent resource pressure, buying a +1 AD die feels like a trivial decision rather than a heart-in-mouth choice.

### Faithfulness Estimate
**~85–88%.** Lower than the primary (93%) because the Parry Combat corner is sacrificed entirely. Higher than expected on the "drawing your path" axis — this variant is arguably *more* faithful to the source's ink-and-paper medium than the primary is, because the primary abstracts the parchment into a minimap, while this variant *is* the parchment. On the "feels like Solo Dungeon Bash on a tablet" metric, this is almost 100% true to the source as a print-and-play object. The sacrifice is purely in the "action roguelite feel" — which the source never had.

### Market Fit

**Audience:** Cozy game fans (*A Short Hike*, *Cozy Grove*, *Unpacking*, *Dorfromantik*), puzzle purists (*Threes!*, *Into the Breach*, *Mini Metro*, *Hexcells*), and the "board game solo adaptation" niche (people who played *Friday*, *Onirim*, *Maquis* on their phone). There is real overlap with players who subscribe to Apple Arcade specifically for quiet puzzle games.

**Shelf:** The Cozy Games shelf on Steam, the Puzzle top-charts on iOS, and the "lunchtime game" category in mobile stores. This variant lives next to *Mini Metro*, *Threes!*, *Dissembler*, and *Islanders* in the player's mind.

**Price point:** $3.99–$5.99 on mobile (App Store / Play Store). $7.99 on Steam. Possibly free with a small IAP to unlock daily seeds and extra parchment themes.

**Audience size:** The cozy puzzle segment is real and active but smaller and more price-sensitive than the roguelite market. Realistic ceiling: 20k–80k copies on mobile in year 1 if it finds its shelf, with a very long tail. Much smaller Steam numbers (~5k) because this player doesn't play on Steam.

**Discoverability pitch:** "*Threes!* meets a dungeon master's notebook. Draw your way through a tiny dungeon with a pouch of dice. No reflexes, no timers, just ink and luck."

### Risk Profile

- **Risk 1 — The combat minigame is too shallow.** If matching dice to slots resolves in 10 seconds per fight and never surprises, the game becomes tedious once the novelty of path-drawing wears off. Mitigation: carefully design monster dice patterns with real strategic variety; add "dice effects" (one die burns a slot, one die doubles the next). The combat must be a real puzzle.
- **Risk 2 — The pacing is too slow for modern audiences.** Even cozy game players have TikTok thumbs. If the first 3 minutes feel like "nothing is happening," the player uninstalls. Mitigation: front-load the first 2 minutes with a guided mini-run that ends in a satisfying boss-lite encounter. Onboard with juice.
- **Risk 3 — Loss of the "dungeon crawler" promise hurts discoverability.** People searching for dungeon games will not find this. People searching for cozy puzzles will find it. The name matters enormously. "Ink & Dice" or "Parchment Pocket" instead of any "Dungeon" title.
- **Risk 4 — Low revenue ceiling.** Cozy puzzles on mobile are notoriously hard to monetize. A $3.99 game needs to sell ~25k copies just to pay one dev for a year at indie rates. Realistic target is modest.
- **Risk 5 — Cannibalization of the primary.** If we ship both, each one dilutes the other's audience. Mitigation: ship this as a mode within the primary, not a separate title (see Cross-Pollination).

### Recommended Team Scope

- **1 designer/programmer.** Most of the work is systems design and UI; rendering is trivial (2D sprites on a paper texture). A single dev with Phaser or React + Canvas experience can build this.
- **1 illustrator** working part-time for 4–6 weeks to produce ~60 hand-drawn sprite assets (dice faces, monster silhouettes, parchment textures, UI icons) and the gallery frame art.
- **1 audio designer, part-time, 2 weeks** for ASMR Foley, lo-fi piano loops, dice-clack library, victory chime. Outsource a small piano loop package instead of a full commissioned soundtrack.
- **Time to playable alpha:** ~6 weeks.
- **Time to launch:** ~4 months from scratch, or ~2 months if shipped as a mode inside the primary SDD.
- **Engine:** **Phaser 3** (canvas renderer, mobile-first, small install, fast iteration) or **React + HTML5 Canvas** for maximum cross-platform reach including PWA install. Not Unity (overkill and mobile bloat). Not Godot (fine but less mobile-polished). Web-first, mobile-first, desktop is a nice-to-have.
- **Total scope:** roughly 1/4 the cost of the primary SDD.

---

## Variant B — "Dungeon Bash: Tower Defense Flip"

### Working Title
**Dracular's Keep** *(also considered: Lord of the Dungeon, Counter-Crawl, Keep the Keep, The Last Boss)*

### Primary Genre
**Turn-based tower defense / reverse dungeon crawler.** In the tradition of *Dungeon Keeper*, *Evil Genius*, *Keeper of the Kingdom*, and the "defender" subgenres of *Kingdom Rush* and *Bloons TD*. Plus a dash of *Plants vs. Zombies* for the "place units on a grid" feel. The identity trick is that the **player plays Dracular**, not the adventurer. This is a "Flip The Lens" variant where the protagonist/antagonist relationship of the source is reversed. The adventurer becomes an AI opponent who advances turn by turn through your dungeon. Your job is to stop them.

### Core Loop Description

A run is one full dungeon defense: ~25–40 minutes. Loops:

**Atomic loop (~5 seconds each):**
- On the player's turn, select a room on the 9×10 grid. Place a monster from your hand (drawn from the source's level tables — Gnoll, Goblin, Skeleton, Orc, Troll, Beholder, Dragon, etc.). Monsters cost gold from your treasury. Each has a level, attack dice, defense dice, and a special ability.

**Primary loop (one turn, ~60 seconds):**
1. **Planning phase:** the player sees the dungeon map, the current position of the advancing hero (starts in North, must reach South), and a hand of ~3 monster cards drawn from a level-appropriate deck. Place monsters, upgrade existing ones, or trap rooms.
2. **Advance phase:** the hero moves one step. If they enter a room with a monster, combat auto-resolves using the source's dice mechanics (AD vs DD, count 6s). Roll animation is fast (~2s) but visible. The dice tray is still the focal point of the combat panel.
3. **Resolve phase:** if the hero won, they take a wound, loot the room, and advance. If the hero lost, they retreat to their previous cell and try a different path next turn. If the hero took too many wounds, they die and the player wins the level.
4. **Reward phase:** player gains gold based on rooms the hero entered. Draw more monster cards. Turn ends.

**Secondary loop (one level, ~10 minutes):**
- The hero attempts to reach the End room (Dracular's chamber in the source — now *your* chamber). They pathfind using an A* variant biased toward "rooms they can probably beat." If they reach the End without dying, the level is lost.
- Each level introduces new monster types, new traps, and a tougher hero class (starting with a Level 1 Fighter, progressing to Level 5 heroes, eventually to multi-hero parties).
- A level ends either in a **player win** (hero dies in your dungeon) or a **player loss** (hero reaches the End room alive).

**Meta loop (~15 levels, ~6–8 hours total campaign):**
- 15 levels = 15 nights of defense, each a fresh 9×10 grid with new constraints (reduced budget, fewer monster types available, new hero archetype).
- Unlock new monster types and room types between levels. Unlock new traps and environmental hazards. Unlock cosmetic trophies of slain heroes that decorate your throne room between missions.
- End boss: a five-hero party, Raid-style, that the player must defeat on a maximum-difficulty grid.
- No permanent roguelite-style run reset — this is campaign-structured, like a mission-based strategy game.

### What Changes From the Primary (SDD Action Roguelite)

- **Perspective flip.** You play as Dracular (or a stand-in), not the adventurer. The core identity of the source is inverted. This is the largest single change and the most controversial.
- **Temporal model: real-time action → strict turn-based strategy.** No beat-based combat, no parry timing. Every action is a clicked command, resolved instantly on confirm.
- **Agency model: one avatar to control → a dungeon's worth of placed units.** The player is now a commander, not a fighter.
- **Combat identity: the player does combat → combat is automated.** The dice tray is still shown (for flavor and feedback), but the player doesn't control it. Combat resolves on AI tick.
- **Routing Puzzle corner inverts.** Instead of drawing the player's path, the player designs a path for the AI hero to fail at. The 9×10 grid becomes a maze-design problem: you want to funnel the hero past your strongest monsters.
- **Session length: 20 min → 25–40 min per level, campaign-length game.**
- **Lethality: player-side burst damage → AI-hero-side incremental attrition.** The hero is slowly whittled down across many rooms, not killed in one dramatic encounter.
- **Source monsters become your units.** The source's level tables (Level 1 through Level 5 monsters) become the player's recruitment pool. This is actually a *great* reuse of source content — every monster in the source has a stat line and a dice identity that maps perfectly to a TD unit.
- **Progression: single-run roguelite → campaign with permanent unlocks.** More like XCOM or Into the Breach's campaign mode than Hades's per-run structure.
- **Visual style: still hand-drawn ink-on-parchment** — this variant preserves the primary's aesthetic, which is cheap and sensible. Just adds new UI elements for monster cards, placement ghosts, and trap icons.

### What Breaks (Faithfulness Sacrifices)

- **The player is not the adventurer.** This is the biggest break. The source fantasy — "I am a lone hero exploring a dungeon with only dice and courage" — is replaced by "I am the dungeon itself, watching a hero try to defeat me." Many players will find this unfaithful to the point of being a different game.
- **The Parry Combat corner is gone entirely.** Combat is automated. The dice tray is a spectator UI, not an interactive one. 33% of the identity triangle is lost.
- **The Routing Puzzle corner inverts.** This is more of a transformation than a break — the routing is still there, just from the opposite side. If you squint, this is actually *more* of a routing puzzle than the primary, because you're designing the entire path rather than choosing one step at a time.
- **The single-run roguelite pacing is replaced with campaign progression.** This changes the "just one more run" hook fundamentally. Players come back for the next level, not for the next run.
- **The hand-drawn ink feel survives, but the "I am drawing this" sensation is gone.** You are no longer carving a line through paper. You are painting walls for someone else to bump into.

### Faithfulness Estimate
**~60–68%.** The lowest of the three variants. It preserves the grid, the monster identities, the dice math, the hand-drawn aesthetic, and the king-move topology — but sacrifices the player's role, the single-run structure, the parry combat, and the "I am the explorer" core fantasy. Ranking by identity-triangle preservation: Routing Puzzle ≈ 70% (preserved but inverted), Parry Combat ≈ 15% (essentially gone), Dice Tray ≈ 60% (present but spectator). Weighted average lands around 60–68%. Some SDB fans will love this; others will feel betrayed.

### Market Fit

**Audience:** Tower defense fans (*Kingdom Rush*, *Bloons TD 6*, *Plants vs. Zombies*, *Bad North*), dungeon-management fans (*Dungeon Keeper*, *War for the Overworld*, *Legend of Keepers*), and strategy campaign players (*Into the Breach*, *XCOM*). The overlap with the primary SDD audience is modest — maybe 30% of Hades fans would try this.

**Shelf:** Steam Strategy tag, Tower Defense subcategory. On mobile, the Strategy top charts. Dungeon Keeper-adjacent Steam curator lists.

**Price point:** $9.99–$14.99 on Steam (full strategy game, campaign content expected). $4.99 on mobile with IAP for extra campaigns.

**Audience size:** Tower defense on Steam is a crowded but reliable segment. A well-received entry can reach 30k–150k copies in year 1. Dungeon Keeper-style games are underserved and can punch above their weight — *Legend of Keepers* (2021) sold well in this niche.

**Discoverability pitch:** "A turn-based Dungeon Keeper with dice combat. Play the dungeon, not the adventurer. 9×10 grid, hand-drawn parchment, ~15 nights of siege."

### Risk Profile

- **Risk 1 — Perspective flip alienates source fans.** People who enjoyed the source played it to be the hero. This variant asks them to be the villain. The emotional core is different. Mitigation: lean into the villain fantasy (flavor text, throne-room trophies, dramatic names like "Dracular's Keep"), not into "reverse Solo Dungeon Bash."
- **Risk 2 — Automated combat is less engaging than participatory combat.** Watching dice roll without deciding anything can become dull after 50 fights. Mitigation: give the player interventions — "spend 1 dark mana to reroll a defense die," "trigger a trap mid-combat for +1 AD" — so they are still active within the combat, just less mechanically.
- **Risk 3 — Scope explodes.** A full campaign tower defense is 3–5x the content requirement of the primary. 15 levels, 30+ monster types, 20 rooms, 10 traps, a campaign narrative, enemy AI that pathfinds intelligently, a metagame shell. This is not a 3-month project.
- **Risk 4 — AI pathfinding is the whole game's difficulty curve.** If the AI hero is dumb, every level is trivial. If the AI is smart, every level is punishing. Tuning the hero AI is the single hardest engineering task in the project.
- **Risk 5 — The "hand-drawn on parchment" aesthetic, which is so appropriate for a solo crawler, feels slightly off when you're managing a keep.** Tower defense players expect crisper, more readable UI than the primary's wobbly ink style provides. Some art retooling required.

### Recommended Team Scope

- **2 programmers.** One systems/gameplay, one AI/pathfinding. The AI dev is mandatory and non-trivial.
- **1–2 artists.** One for new UI (placement ghosts, monster cards, trap icons, stat readouts), one for ~30 additional monster variants. The primary's artist can probably cover the base pipeline but campaigns need more content.
- **1 designer.** Dedicated campaign-and-progression designer to tune the 15 missions, the budget curves, the unlock tree, and the difficulty ramp.
- **1 audio designer**, part-time.
- **Time to playable alpha:** ~5 months.
- **Time to launch:** ~10–14 months. This is a bigger game than the primary.
- **Engine:** **Godot 4** (good turn-based UI, cheap, handles 2D sprite grids well) or **Unity** (better asset store and tower-defense templates, larger install). PC primary, mobile as a port after launch.
- **Total scope:** roughly 2–2.5x the cost of the primary SDD. This is a major investment, not a side project.

---

## Variant C — "Dungeon Bash: Auto-Battler"

### Working Title
**Pocket Dice Dungeon** *(also considered: Dice Ascent, Roll to the Bottom, Six & Sinew, Tray Tactics)*

### Primary Genre
**Dice-builder auto-battler / roguelite deckbuilder hybrid.** In the tradition of *Slay the Spire* (route-choice structure), *Teamfight Tactics* and *Super Auto Pets* (auto-battle resolution), *Dicey Dungeons* (dice-as-cards), *Inscryption* (hand-drawn occult aesthetic), and *Monster Train* (floor ascent as structure). This variant preserves the dice identity of the source — dice are the central mechanic — but sacrifices the routing puzzle in favor of loadout strategy, shop optimization, and combat-as-automated-math.

### Core Loop Description

A run is ~30 minutes and consists of 10 "floors" (one per row of the source's 9×10 grid). Loops:

**Atomic loop (~10 seconds):**
- Drag a die from your dice bag to a slot in your dice tray. Consider swapping a trinket. Click "Ready" to lock in.

**Primary loop (one room, ~90 seconds):**
1. **Room selection:** at the start of each floor, the player sees 9 rooms (one for each column of the source's grid) presented as cards in a row — Combat, Elite, Shop, Campfire, Mystery, Treasure, Shrine, Trap, Rest. Click one to enter it. Unentered rooms are discarded when you advance. This is a route-choice mechanic akin to *Slay the Spire*'s map.
2. **Encounter:** depending on room type, different things happen. Most commonly, combat.
3. **Combat auto-resolve:** the player sees their dice tray, the enemy's dice tray, and hits "Fight." Dice roll in turn order. Hits are counted. Damage is applied. The whole exchange takes ~10 seconds to animate, accompanied by a satisfying crescendo of clacks and ink-splashes. The player can tune animation speed in settings.
4. **Reward:** win → gold, possibly a new die, possibly a new trinket. Lose → lose a heart from your run pool. Run ends when the run pool reaches 0.

**Secondary loop (one floor, ~2–3 minutes):**
- Enter a room, resolve it, see a new row of 9 rooms for the next floor. Advance. Each floor's rooms are drawn from a level-appropriate pool, so early floors are softball and late floors are brutal.
- Floors 1 through 9 are standard rooms. Floor 10 is Dracular.

**Tertiary loop (one run, ~30 minutes):**
- 10 floors, climaxing in Dracular on floor 10. Dracular is a multi-phase dice boss (he has three dice trays, each with different elemental affinities, revealed in sequence). Defeating him is the run win condition.
- Between major floors, "Campfire" rooms let you heal, reforge a die (change its faces), or delete a die from your pool (thin your pool, higher average quality).
- Shop rooms sell new dice with special faces (burning dice, poison dice, chain dice, stacking dice), trinkets that modify your tray globally ("all 6s deal +2"), and curses that add negative effects for gold.

**Meta loop (cross-run):**
- Unlock new starting loadouts (6 different "classes" each with a different starting dice bag — Barbarian has big attack dice, Rogue has chain dice, Cleric has heal dice, etc.)
- Unlock new dice types, new trinkets, and new floor-specific events.
- Ascension system (à la Slay the Spire) that adds modifiers for replayability.

### How the 9×10 Grid Maps to Floors
This is the key structural translation. The source's 9×10 dungeon grid becomes: **10 floors (rows), each floor presenting a choice of up to 9 rooms (columns)**. On each floor, the player chooses one of the 9 rooms to enter. The remaining 8 are discarded. Over 10 floors, the player visits exactly 10 rooms out of a possible 90 — preserving the source's "you don't visit every cell" feel without requiring king-move topology or no-revisit logic. The choice is horizontal (which column), not topological (which neighbor).

This mapping elegantly preserves the source's grid dimension, its cell-count, and its "choose your path" feeling, while trading spatial adjacency for something more like *Slay the Spire*'s branching map. It's a legitimate translation but it **does sacrifice king-move routing entirely**.

### What Changes From the Primary (SDD Action Roguelite)

- **Combat verb: parry-and-punish → dice-loadout-and-autoresolve.** The player builds the dice bag; combat plays itself. The player is a deckbuilder, not a fighter.
- **Temporal model: beat-based real-time → fully turn-based with snappy animations.** Each combat round is a 10s auto-resolution. The player has infinite time between rounds to plan.
- **Spatial model: 9×10 king-move grid → 10 floors × 9 choices-per-floor linear ascent.** The grid is preserved numerically but its topology is gutted. There is no adjacency, no blocked paths, no "rooms you can't reach from here" — every room on the next floor is always reachable.
- **Agency model: mouse+keyboard avatar control → point-and-click deckbuilder menus.** No locomotion. No room-entry animation. Just cards, trays, and buttons.
- **HP model: 17 hearts → a small run-life pool of ~5, or a per-combat HP that resets between fights.** Either works; Slay the Spire style (per-run HP that decays) is more tense.
- **Info model: content-fog room reveal → full transparency of next floor's 9 options.** Fog only applies to what's *two* floors away. You always see your immediate choices clearly.
- **Meta structure: pure run reset → meta-unlocks of classes, dice types, and ascension levels.** More long-tail meta than the primary's "cosmetic and bestiary only" rule.
- **Art: hand-drawn ink → hand-drawn ink, but stylized more toward *Inscryption* or *Dicey Dungeons*.** Dice as the central visual object. Every die is a character. Rare dice have unique art.
- **Session length: 20 min → 25–35 min per run. Slightly longer but very similar.**

### What Breaks (Faithfulness Sacrifices)

- **The Routing Puzzle corner collapses to a linear ascent.** King-adjacency is gone. No-revisit is trivially preserved (each floor is visited exactly once). The block-detection warning becomes meaningless. Roughly 50% loss of this corner.
- **The Parry Combat corner collapses entirely.** Combat is automated. The dice tray is the player's only interaction. 100% loss of this corner.
- **The Dice Tray corner is amplified — preserved at ~130% of the primary.** The dice are now the *entire* game. Every combat verb, every upgrade, every shop decision is about the dice bag. This variant is arguably the most faithful to the "count 6s on d6" identity of the source.
- **The king-move / 8-neighbour feeling of the source's printed grid is sacrificed entirely.** This is the biggest geometric loss.
- **The "one-shot lethal" combat identity softens into attritional HP chipping** — classic auto-battler pacing. Fights are won by a cumulative advantage, not by a single well-timed strike.
- **The 2D top-down avatar is removed.** There is no "walking around a dungeon." The game is pure menus + combat animations.

### Faithfulness Estimate
**~75–82%.** Higher than Variant B (~60–68%) because the dice identity is preserved and amplified. Lower than Variant A (~85–88%) because the parchment ritual of path-drawing is essentially gone — replaced by card-picking. Specifically: Routing Puzzle ≈ 40% (linear ascent is a shadow of king-move routing), Parry Combat ≈ 10% (automated), Dice Tray ≈ 130% (the whole game). Weighted average lands around 75–82% depending on how much weight you give the Dice Tray corner versus the others. If you weight Dice Tray highly (since it's the unique differentiator), the faithfulness is nearly as high as the primary. If you weight Routing Puzzle highly (since it's the "hidden subgenre" of the source), the faithfulness drops.

### Market Fit

**Audience:** Slay the Spire fans, *Dicey Dungeons* fans, *Inscryption* fans, *Monster Train* fans, *Super Auto Pets* fans, *Balatro* fans. This is an enormous and active audience in 2024–2026. Deckbuilder roguelites are one of the healthiest genres on Steam right now, and the dice-as-cards subgenre is a proven niche within that.

**Shelf:** Steam "Deckbuilder" and "Roguelite" tags. On mobile, the Strategy and Card/Roguelite top charts. Directly adjacent to *Slay the Spire*, *Balatro*, *Dicey Dungeons* in the player's mental shelf.

**Price point:** $14.99–$19.99 on Steam. $4.99–$9.99 on mobile with optional DLC.

**Audience size:** This is the largest potential audience of the three variants. A good deckbuilder roguelite can sell 100k–1M copies in year 1 if it catches. *Dicey Dungeons* sold over 400k; *Balatro* sold over 5 million. The ceiling is high and the floor is respectable.

**Discoverability pitch:** "Slay the Spire meets a dungeon master's notebook. Every die in your pouch is a character. Every floor is a choice. 30-minute runs, 10 classes, deep loadout strategy."

### Risk Profile

- **Risk 1 — Saturated market.** Deckbuilder roguelites are having a moment. Competition is *fierce*. A game has to be exceptional to stand out. Mitigation: lean hard into the dice-specific identity and the hand-drawn parchment aesthetic. Don't be "another deckbuilder"; be "the ink-on-parchment dice game."
- **Risk 2 — Over-competition with *Dicey Dungeons* and *Balatro*.** Those games already do "dice + roguelite." Any new entry will be compared to them. Mitigation: the 10-floor × 9-choice structure is distinct from either, and the aesthetic is strongly differentiated.
- **Risk 3 — Auto-resolve combat is less satisfying than participatory combat.** The primary's parry thrill is gone. Players need a different thrill. Mitigation: make the dice animations incredibly juicy, and make loadout decisions deeply strategic. The thrill should be in the *anticipation* of the fight, not its resolution.
- **Risk 4 — Content scope is large.** 10 classes × 40 dice types × 30 trinkets × 20 floor events × 10 bosses = hundreds of content cards that need art, stats, and balance. This is bigger than the primary.
- **Risk 5 — Balance is a years-long job.** Slay the Spire and Balatro are still being balanced years after launch. A deckbuilder is a live-balance game. Plan for it.

### Recommended Team Scope

- **2 programmers.** One for systems and combat resolution, one for meta-progression and UI.
- **2 artists.** One for dice and trinket card art (the bulk of the content), one for character portraits, boss art, and UI frames.
- **1 designer.** Dedicated balance and content designer. This is a full-time job for the whole project and into post-launch.
- **1 audio designer.** Dice clacks, shop stingers, victory fanfares, boss music. Part-time but ongoing.
- **Time to playable alpha:** ~5–6 months.
- **Time to launch:** ~12–18 months. Deckbuilders need polish time.
- **Engine:** **Godot 4** or **Unity**. Unity has the asset-store edge for deckbuilder UI kits. Godot is lighter and cheaper to maintain. Either works. PC/Steam primary, mobile as a strong secondary (deckbuilders play beautifully on mobile).
- **Total scope:** ~2x the cost of the primary SDD, with significant post-launch content ambition.

---

## Comparison Table

| Dimension | **Primary (SDD Action Roguelite)** | **Variant A (Ink & Dice — Cozy Puzzle)** | **Variant B (Dracular's Keep — Tower Defense)** | **Variant C (Pocket Dice Dungeon — Auto-Battler)** |
|---|---|---|---|---|
| **Core verb** | Parry + route | Carve + match | Place + tune | Build + advance |
| **Temporal model** | Beat-based real-time | Fully turn-based, player-paced | Strict turn-based | Turn-based w/ auto-resolve |
| **Player role** | Adventurer | Ritualist/puzzler | Dungeon master (villain) | Deckbuilder |
| **Spatial model** | 9×10 king-move grid, room-at-a-time | 9×10 full grid, always visible | 9×10 grid, enemy advances through it | 10 floors × 9 rooms (linear ascent) |
| **Combat** | Real-time parry + dodge | Dice-matching puzzle | Auto-resolve w/ interventions | Auto-resolve w/ loadout strategy |
| **Dice tray role** | Side panel, passive visualizer | The entire UI, interactive | Spectator UI | The entire game |
| **Session length** | 20 min median | 10–20 min | 25–40 min/level | 25–35 min |
| **Meta progression** | Cosmetic only | Cosmetic only | Full campaign unlocks | Classes + ascension |
| **Failure handling** | Run reset | Gallery archive, soft loss | Level restart | Run reset |
| **Lethality feel** | 1-hit death, bursty | Softened or removed | Hero attrition | HP chipping |
| **Art style** | Hand-drawn ink, moody | Hand-drawn ink, cheerful | Hand-drawn ink + villain flourishes | Hand-drawn ink + card art |
| **Audio feel** | Combat hits, parry clinks | ASMR, lo-fi piano | Siege tension + dungeon ambience | Clacks + shop stingers |
| **Primary platform** | PC/Steam | Mobile-first, web | PC/Steam | PC/Steam + mobile strong second |
| **Engine** | Godot or Unity | Phaser 3 / React Canvas | Godot 4 or Unity | Godot 4 or Unity |
| **Price point** | $7–$12 | $3.99–$7.99 | $9.99–$14.99 | $14.99–$19.99 |
| **Target audience size** | Medium (roguelite fans) | Small (cozy puzzle niche) | Medium (TD + dungeon keeper) | Large (deckbuilder boom) |
| **Identity corners preserved** | Routing ✅, Parry ✅, Dice ✅ | Routing ⚠, Parry ❌, Dice ✅ | Routing ⚠ (inverted), Parry ❌, Dice ⚠ | Routing ❌, Parry ❌, Dice ✅✅ |
| **Faithfulness estimate** | **93%** | **85–88%** | **60–68%** | **75–82%** |
| **Team size** | 1 dev + 1 artist + part-time audio | 1 dev + part-time artist + part-time audio | 2 devs + 1–2 artists + 1 designer + audio | 2 devs + 2 artists + 1 designer + audio |
| **Time to launch** | 6–9 months | 2–4 months | 10–14 months | 12–18 months |
| **Relative cost vs primary** | 1.0x | ~0.25x | ~2–2.5x | ~2x |
| **Revenue ceiling (rough)** | 50k–200k copies | 20k–80k copies | 30k–150k copies | 100k–1M copies |
| **Risk profile** | Moderate (crowded genre) | Low-medium (small audience) | High (scope + perspective flip) | High (saturated + balance-intensive) |

---

## Cross-Pollination Opportunities

Even if the primary Action Roguelite is the final shipping genre, each variant contains features that can — and should — be ported back as optional modes, challenge runs, or post-launch content. These are cheap wins that expand the primary's audience without diluting its identity.

### From Variant A — "Ink & Dice"

- **Draw-Only Mode.** A post-launch toggle (Settings → "Cozy Mode") that removes real-time combat. In this mode, combat pauses and becomes a dice-matching puzzle exactly as in Variant A. The player can take their time, drag dice to slots, and resume the run at their own pace. This mode is the **single highest-value** cross-pollination in this document. It's a 2-week implementation in the primary that opens up the entire cozy-game audience. Ship it as a free post-launch update 3 months after launch to extend the tail.
- **Daily Seed Gallery.** Store completed run parchments (or death markers) in a local gallery, and allow a daily seed challenge where players can compare their routes to a friend's. This adds social retention without multiplayer. 1–2 week implementation.
- **ASMR Audio Option.** A toggle to replace combat hit Foley with pen-stroke Foley. Niche but beloved by some players. 1-week implementation with purchased Foley assets.
- **No-Time-Pressure Difficulty.** A "Zen" difficulty where no combat happens in real time, all encounters are turn-based. Overlaps with Draw-Only Mode.

### From Variant B — "Dracular's Keep"

- **Dungeon Master Sandbox.** A post-launch DLC or free mode where the player can design a dungeon on the 9×10 grid and challenge friends (or their own alternate runs) to beat it. The level editor uses the existing art and monster assets, so the cost is ~1 month of UI work. Low faithfulness impact, high community value.
- **"Boss Rush Defense."** A single-room defensive mode where the player, as Dracular himself, is swarmed by waves of heroes and must last as long as possible. This is a Variant-B-lite implementation — just the combat, not the full TD campaign. Fits the primary as a challenge mode. ~3 weeks of work.
- **Monster Data Visibility.** Variant B requires full monster stat transparency (because the player places them as units). Porting that transparency to the primary as an unlockable Bestiary tool is a lightweight feature that enriches the meta-progression without changing core combat. Already planned in the primary's post-run dashboard; just expand it.

### From Variant C — "Pocket Dice Dungeon"

- **Chaos Auto-Battle Mode.** A weekly challenge mode where the primary's combat is replaced with Variant-C-style auto-resolve. The player still explores the 9×10 grid with real-time locomotion, but combat resolves automatically based on loadout. This is a "speedrunner's mode" — you optimize the dice bag, then let the runs play themselves. 3–4 week implementation.
- **Dice Trinket System.** Variant C's dice trinkets (items that modify the whole dice tray: "all 6s deal +2," "first attack each room rolls an extra die") are directly portable to the primary as a new shop inventory category. They deepen the primary's loadout decisions. ~2 weeks of work. **Highly recommended.**
- **Shop Shrine Dice-Reforge.** Let the player reforge a single die's face at a Shop Shrine (turn a 1 into a 4, or swap a mundane d6 for a burning d6). This comes straight from Variant C's campfire rooms and deepens the primary's meta without adding a new currency. ~1 week.
- **Class Loadouts / Alt Starts.** Variant C's 10 starting classes can be imported as unlockable "Kit" presets in the primary — a Barbarian kit, a Rogue kit, a Cleric kit. This is already a common roguelite feature and would be a natural post-launch DLC. ~1–2 months of content work, perfect for a Year 1 DLC.

### Meta-cross-pollination — things all three variants teach the primary

- **The Dice Tray must be genuinely interactive, not merely visualized.** All three variants promote the dice tray to a higher level of importance than the primary's current design. The primary's current spec has the dice tray as a side-panel visualization. This document suggests making it at least partly interactive (e.g., click a die to manually assign its effect) to serve both action fans and dice-curious players.
- **The 9×10 grid is more flexible than we thought.** It can be king-move (primary), full-grid (A), siege-map (B), or ascent (C). Preserving the number 9×10 is a strong identity anchor even when the topology changes.
- **The hand-drawn parchment aesthetic is universal.** Every variant uses it. This confirms the primary's decision to make the art style non-negotiable — it's the single most portable identity element.

---

## Recommendation

**Ship the primary (Action Roguelite).** It is the correct answer for the target market, the correct answer for the source's three identity corners, and the correct answer for the team's realistic scope. No variant should displace it.

### Variant A — Ink & Dice: **Port as a Mode, Do Not Ship Standalone**
Ship Variant A's core loop as a **"Draw-Only Mode"** inside the primary. This is the highest-value cross-pollination in this analysis. The cost is ~2–4 weeks of additional development (reusing all art, all monster stats, all dungeon generation) and the gain is access to the entire cozy-puzzle audience. Do NOT ship Variant A as a separate title — it would cannibalize the primary's marketing budget and compete against a player base that barely overlaps with our target. Integrate it, don't spin it off. Ship the Draw-Only Mode as a free post-launch update approximately 3 months after launch to re-engage press and extend the review cycle.

### Variant B — Dracular's Keep: **Do Not Build**
Variant B is the most interesting variant intellectually but the least valuable commercially within our scope. It is 2–2.5x the cost of the primary, requires a dedicated AI engineer, sacrifices the player-role fantasy that drives source-fan purchases, and competes in a crowded Dungeon-Keeper-alike niche against entrenched titles. The perspective flip is creatively admirable but strategically risky. **Shelve it.** Preserve the idea in this document and consider it only if the primary ships successfully and the team earns the budget for a sequel/spinoff. A lightweight "Boss Rush Defense" mode (where the player briefly flips to play Dracular) is the only Variant B feature worth porting to the primary, and even that is a stretch goal.

### Variant C — Pocket Dice Dungeon: **Do Not Build as Standalone, but Port Features Aggressively**
Variant C is the most commercially promising of the three because the deckbuilder market is booming. But it is also the riskiest because the deckbuilder market is booming — competition is fierce, content demands are enormous, and balance work is years-long. Our team is not currently scoped for an 18-month deckbuilder with live-balance commitments. **Do not build it.** However, aggressively port Variant C's dice-trinket system, dice-reforge shop, and class-loadout unlocks into the primary as post-launch content. Those three features alone will meaningfully deepen the primary's meta and attract deckbuilder-curious players without requiring a full genre pivot. If the primary sells well (>100k copies in year 1), reopen the Variant C discussion as a spinoff project for year 2–3.

### Priority order for post-launch content, based on this analysis
1. **Draw-Only Mode (Variant A port)** — ~3 weeks, highest audience expansion per dev-week.
2. **Dice Trinket system (Variant C port)** — ~2 weeks, deepens primary's loadout.
3. **Dice-Reforge shop (Variant C port)** — ~1 week, adds shop decision space.
4. **Daily Seed Gallery (Variant A port)** — ~1 week, adds social retention.
5. **Class Loadouts DLC (Variant C port)** — ~1–2 months, paid DLC in year 1.
6. **Boss Rush Defense (Variant B lite)** — ~3 weeks, niche challenge mode.

### Final Decision Lock — Stage RT-A1 Output
- **Primary genre stays locked** to 2D top-down Action Roguelite / Dungeon Crawler.
- **Variants A, B, and C are documented but not pursued as primary directions.**
- **Cross-pollination priority list (above) becomes the post-launch content roadmap seed.**
- **Draw-Only Mode is explicitly pre-committed** as a 3-month post-launch free update, because it unlocks a whole audience segment at minimal cost.
- **Variant B is explicitly deprioritized** and will not appear in any further RealTimeForge stages.
- **Variant C features feed into the year-1 DLC plan** without triggering a genre pivot.

The primary survives its genre stress test. The variants have paid for themselves in cross-pollination ideas. Stage RT-A1 closes with one new commitment (Draw-Only Mode as a post-launch mode) and zero changes to the primary's core scope.
