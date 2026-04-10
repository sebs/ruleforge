# Prototype Prompts — Solo Dungeon Bash

These three tool-agnostic prompts are designed to drop into any AI prototyping tool (Rosebud.ai, v0.dev, Bolt, Lovable, Cursor) to generate a working interactive prototype of one screen at a time. Each prompt is self-contained: paste it in and receive a runnable artefact.

**Visual style note (applies to all three prompts):** hand-drawn ink-on-parchment aesthetic, as if the player had sketched the game in a journal. Wobbly lines, cross-hatch shading, handwritten fonts for player-facing labels, clean sans-serif only for numbers. Monochrome base with selective colour accents: gold for treasure, blue for potions, red for monsters, green for the current square.

---

## Prompt 1 — Core Gameplay Loop Prototype

> Build an interactive single-screen prototype of a solo dungeon crawler called **Solo Dungeon Bash**.
>
> **Visual style:** hand-drawn ink-on-parchment. Think medieval adventurer's journal — wobbly grid lines, cross-hatched shading, handwritten-style labels. Beige/cream background. Monochrome with selective colour: gold for treasure, blue for potions, red for monster encounters, green highlight for the current square, grey for visited squares.
>
> **Layout:** two columns. Left (70% width): the dungeon grid. Right (30% width): a player HUD.
>
> **The dungeon grid:**
> - 9 columns by 10 rows of square cells, laid out as a rectangular grid.
> - Below the middle column of row 1, draw a single "Start" square labelled **Start**. Above the middle column of row 10, draw a single "End" square labelled **End**. Both connect visually to the grid.
> - Label the rows 1–10 on the left side (*"Dungeon Level"*).
> - On game start, place a marker on the Start square.
> - The player clicks a highlighted cell to move there. Legal cells are the king-adjacent (8-neighbour) unvisited cells from the current position. Highlight legal cells with a soft pulsing glow.
> - When the player clicks a legal cell: (1) mark the old cell as visited with a small circled "X" in ink, (2) animate an ink line from the old cell to the new cell, (3) move the marker.
> - Revisiting any visited cell is forbidden.
>
> **Room content roll:**
> - Immediately after moving into a new cell, animate a d6 die rolling in the middle of the screen for about 800ms.
> - The die lands on a random number 1–6. Look up the result in the level table for the row number the player is in (the row = dungeon level). For this prototype, use these simplified level tables:
>   - **Level 1:** 1=Potion, 2–4=Orc (1 AD), 5=Empty, 6=Treasure
>   - **Level 2:** 1=Orc (1 AD), 2=Wolf (2 AD), 3–5=Empty, 6=Treasure
>   - **Level 3:** 1=Orc, 2=Wolf, 3=Skeleton (3 AD), 4=Treasure, 5=Potion, 6=Empty
>   - For rows 4–10, increase the top monster Attack Dice by 1 per row (Level 10 tops out at "Demon 10 AD").
> - Display the room result as an ink-sketched icon in the cell: gold coin for Treasure, blue flask for Potion, monster silhouette for Monster, dot for Empty.
>
> **The HUD (right column):**
> - Five large counters, each styled like a handwritten labelled box:
>   - **Health:** 17 / 17
>   - **Attack Dice:** 1
>   - **Defence Dice:** 1
>   - **Treasure:** 0
>   - **Potions:** 0
> - Below the counters, a "Next Turn" button (only clickable when the current turn is complete).
>
> **Combat (simplified for this prototype):**
> - When a Monster result is rolled, open a combat modal.
> - Combat runs in alternating rounds until one side is dead:
>   1. Monster rolls its Attack Dice (show the dice animated). Count how many land on 6 — those are Hits.
>   2. Player rolls their Defence Dice. Count 6s — those cancel Hits. Remaining Hits subtract from Player Health.
>   3. If Health ≤ 0, show "YOU DIED" and end the run.
>   4. Player rolls Attack Dice. Count 6s as Hits.
>   5. If any hits land (normal monsters have 0 defence and 1 HP), the monster dies; close the modal.
> - Show every die visually, highlight 6s in red, show the tally clearly.
>
> **Treasure and Potion resolution:** instant, just +1 to the relevant counter with a brief number-pop animation.
>
> **Recovery phase (after combat or peaceful room):** if the player has any potions, show a small "Use Potion" button. Each tap: Potions -1, Health +1 (cap at 17). Then the "Next Turn" button appears.
>
> **Win/Lose:**
> - If the player moves into the End square, show a scripted boss fight against **Dracular** (9 Attack Dice, 9 Defence Dice) using the same combat logic but with the monster rolling defence dice in step 4. If the player wins, show "VICTORY." If the player dies, show "YOU DIED."
> - If the player has zero king-adjacent unvisited cells, show "BLOCKED — you are trapped. Run ended."
>
> **Tech constraints:** plain HTML + CSS + JavaScript (or whatever the prototyping tool prefers). No backend. No build step. One HTML file if possible. Mobile-friendly. Dice rolls use `Math.random()`. Keep the code readable.
>
> **Out of scope for this prototype:** shop, item slots, bought-potion-next-turn timing, achievements, audio, save/load. Keep it to the core loop only.

---

## Prompt 2 — Game Setup & Configuration Screen

> Build an interactive game-setup screen for a solo dungeon crawler called **Solo Dungeon Bash**. This is the screen the player sees *before* starting a run.
>
> **Visual style:** hand-drawn ink-on-parchment aesthetic. Beige/cream parchment background with subtle texture. Monochrome with selective gold highlights for buttons. Wobbly lines, cross-hatched borders, handwritten-style typography for labels, clean sans-serif for numbers. Think "Choose Your Own Adventure book cover" crossed with "dungeon master's notebook".
>
> **Layout:** a single centred page, roughly 500 × 800 pixels, presented like an adventurer's journal.
>
> **Content — top to bottom:**
>
> 1. **Title:** "Solo Dungeon Bash" in large ink-brush lettering. Below it, a smaller subtitle: "Enter the dungeon. Survive Dracular. Walk out alive."
>
> 2. **Character sheet block** (stylised as a handwritten sheet with ruled lines):
>    - **Adventurer name:** (text input, placeholder "Your name here")
>    - **Starting Health:** 17 (not editable in v1; shown as a fixed stat)
>    - **Starting Attack Dice:** 1
>    - **Starting Defence Dice:** 1
>
> 3. **Difficulty selection** — four radio buttons drawn as ink-filled circles:
>    - **Easy** (subtitle: "20 HP, +1 DD") — "For first-time adventurers."
>    - **Normal** (subtitle: "17 HP, 1 AD, 1 DD") — **default** — "The true dungeon."
>    - **Hard** (subtitle: "14 HP, 1 DD") — "For the bold."
>    - **Nightmare** (subtitle: "14 HP, boss +1 AD/+1 DD") — "Do not attempt alone."
>    - Only one selectable at a time; show the selected one with an inked-in circle.
>
> 4. **Dungeon options:**
>    - **Seed:** a text input with a small "🎲 Randomize" button that generates a new 8-character alphanumeric seed. Below the input: a helper line "Same seed = same dungeon. Share it with a friend."
>    - **Daily Seed button:** a single button with today's date, e.g. "Today's Seed: 2026-04-10". Click to lock the seed to the daily challenge seed (e.g. hash of today's date).
>
> 5. **Preview panel:**
>    - As the player changes difficulty or seed, update a small preview panel showing derived stats: starting HP, the boss's Attack Dice and Defence Dice, and a pre-computed "threat score" (just the sum of max monsters per level, for flavour).
>
> 6. **Actions — bottom of page:**
>    - A big inked "**Descend into the Dungeon**" button — only enabled when the adventurer name is non-empty.
>    - A smaller secondary link: "Tutorial" — tooltip: "Learn the basics first."
>    - A smaller secondary link: "Back to Menu".
>
> **Interactions to wire up:**
> - Randomize seed button generates a new 8-char alphanumeric seed and writes it to the input.
> - Daily seed button overwrites the seed with a deterministic value derived from today's date.
> - Difficulty radio updates the preview panel in real time.
> - "Descend into the Dungeon" button logs the selected config to the browser console (in a real build this would start the run) and shows a toast "Run started with seed: XYZ123".
>
> **Accessibility:**
> - Labels are associated with inputs.
> - Radio group uses `role="radiogroup"`.
> - Keyboard nav works with arrows and tab.
> - Alt text on decorative ink flourishes.
>
> **Tech constraints:** plain HTML + CSS + JavaScript. No backend. One HTML file. Mobile-friendly. Use CSS to fake the parchment texture (repeating-radial-gradient is fine). Use a handwritten Google Font like "Caveat" or "Homemade Apple" for labels and "Inter" for numbers.

---

## Prompt 3 — End-of-Run Results Screen

> Build an interactive end-of-run results screen for a solo dungeon crawler called **Solo Dungeon Bash**. This is the screen the player sees when their run ends — whether in victory or defeat.
>
> **Visual style:** hand-drawn ink-on-parchment. Beige/cream parchment with subtle age spots. Monochrome with selective colour: gold for victory accents, red for defeat accents. Wobbly ink lines, cross-hatch shading, handwritten-style labels. Think "page torn from a dungeon master's journal".
>
> **Layout:** a single centred page, about 500 × 900 pixels, presented like a mini hero's obituary or a victory chronicle.
>
> **The screen has two modes — Victory and Defeat — and should toggle-able via a switch at the top for development purposes.**
>
> **Victory mode:**
> 1. **Headline in large ink-brush letters:** "VICTORY" with a crossed-sword flourish. Gold accent.
> 2. **Subtitle** (handwritten italic): "Dracular lies dead. You emerge from the dungeon, bloodied but alive."
>
> **Defeat mode:**
> 1. **Headline:** "YOU DIED" or "BLOCKED — NO PATH TO END" depending on cause. Red accent.
> 2. **Subtitle** (handwritten italic): one of two, randomly:
>    - "Your bones will rest in Level {N} forever." (for combat death, where N is the level the player died in)
>    - "Trapped in your own footsteps. The End remains unreached." (for the blocked case)
>
> **Always-present content — below the headline:**
>
> 3. **Run summary block** (styled as a crumpled piece of notebook paper with dotted lines):
>    - **Adventurer:** (the name from setup)
>    - **Dungeon seed:** e.g. `XYZ12345`
>    - **Difficulty:** Normal
>    - **Turns taken:** 47
>    - **Rooms explored:** 47 of 92
>    - **Treasure earned:** 14
>    - **Potions used:** 8
>    - **Monsters killed:** 23
>    - **Deepest level reached:** 10
>    - **Final HP:** 3 / 17 (victory) or 0 / 17 (defeat)
>    - **Run time:** 27m 14s
>
> 4. **"Fallen foes" list** — a compact inked table of monster types killed with tallies:
>    ```
>    Orc ||||
>    Wolf ||||
>    Skeleton |||
>    ...
>    Dracular (if killed) ✓
>    ```
>    Tick marks rendered in hand-drawn ink strokes.
>
> 5. **"Best moments" highlights** — a short bullet list of 2–4 dramatic events, e.g.:
>    - "Survived a 6-hit round at 4 HP (Cyclops, Level 7)"
>    - "Bought Magical Armour with 6 Treasure on turn 29"
>    - "First-try killed Dracular in 4 combat rounds"
>
>    (For this prototype, hardcode the highlights — in a real build they'd be generated from the run log.)
>
> 6. **Actions — bottom of page:**
>    - A big inked "**Play Again**" button — starts a new run with a fresh random seed.
>    - **"Same Seed Again"** — rerun the exact same seed.
>    - **"Copy Result"** — puts a share-friendly string on the clipboard: `"Solo Dungeon Bash — Normal — Seed XYZ12345 — Victory in 47 turns 🏆"` (or with 💀 for defeat).
>    - **"View Run Log"** — opens a modal with the turn-by-turn log (for this prototype, just hardcode 5–10 log lines).
>    - **"Back to Menu"**.
>
> **Interactions to wire up:**
> - Toggle at the top switches between Victory and Defeat mode, updating headline, subtitle, and final HP.
> - "Copy Result" writes the share string to the clipboard and shows a toast "Copied!".
> - "View Run Log" opens a modal with hardcoded sample log lines (e.g. "Turn 1: moved (5,1), rolled 4, fought Orc, won", etc.).
> - "Play Again" logs an event to console (in a real build would start a new run).
>
> **Accessibility:**
> - The headline uses heading semantics (`<h1>`).
> - All buttons have accessible names.
> - Colour is not the only indicator of Victory/Defeat — the text is.
> - The run summary uses a `<dl>` (description list) for name/value pairs.
>
> **Tech constraints:** plain HTML + CSS + JavaScript. No backend. One HTML file. Mobile-friendly. Use a handwritten Google Font like "Caveat" or "Homemade Apple" for labels and "Inter" for numbers. Use CSS to fake the parchment texture.

---

## Notes on Using These Prompts

- Each prompt is a self-contained deliverable. You can produce prototypes in any order.
- Prompt 1 is the most ambitious; if the tool struggles, break it into "grid navigation only" first, then layer on combat.
- Prompts 2 and 3 are pure UI screens with hardcoded data — they should succeed on the first try.
- Collectively they cover the "front door" (setup), "kitchen" (core loop), and "back door" (end screen) of the game — enough to user-test the aesthetic and flow before any real engineering.
- When ready for integration, combine the three into a single SPA using the architecture in `Architecture.md`.
