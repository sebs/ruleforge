# SpatialModel.md — Solo Dungeon Bash → Real-Time Translation

> Stage RT-1: Spatial Translation
> Source: Felbrigg Herriot (2007), *Solo Dungeon Bash*. 3-page solo roll-and-write.
> Target: Real-time interactive digital game (format decision below).

---

## 1. Source Spatial Model

Solo Dungeon Bash has one of the thinnest spatial models you can find in a printed game. It is strictly:

### 1.1 Grid topology

- A rectangular orthogonal grid of **9 columns × 10 rows = 90 "normal" cells**.
- **Two special protrusion cells** attached to the grid:
  - **Start**: a single square attached *below* row 1 at the middle column (column 5 of 9).
  - **End**: a single square attached *above* row 10 at the middle column (column 5 of 9).
- Total addressable squares: **92**.
- Each row is a named "**Dungeon Level**" (1 through 10). Row 1 is shallowest (closest to the Start protrusion), row 10 is deepest (closest to the End protrusion and to Dracular).

The player's position is a single cell. The grid lives only as pen marks on graph paper — there is **no board**, no tile art, no spatial depiction of what a room looks like.

### 1.2 Adjacency rule

- Movement is **king-move / 8-neighbourhood**. From any cell, the set of legal next cells is the 8 cells that share a side OR a corner.
- The Start cell is king-adjacent to the **3** bottom cells of row 1 that touch it (column 4, 5, 6 of row 1) — i.e., same as a king sitting on an edge square.
- Symmetrically, the End cell is king-adjacent to the 3 top cells of row 10 (column 4, 5, 6).
- There is no variable edge cost. Every legal move has cost "one turn."

### 1.3 Movement constraints

- **No revisit.** A cell, once entered and marked with a circle, is forever forbidden. The traced path is a simple self-avoiding walk on the grid.
- **You may backtrack levels.** The direction of travel is unrestricted as long as each step is king-adjacent and unvisited. A player may dip into row 6, drop back to row 3 via a diagonal column corridor, then climb again.
- **Must keep End reachable.** At any commit, if the graph of "still-unvisited king-adjacent cells" no longer has a path from the player's current cell to the End cell, the player has lost. This is checked implicitly by the player in the print version; digitally, the default rule (A-1) is a BFS at every move.
- **There is no hostile blocking.** No monster ever claims a cell, no tile ever becomes impassable, and no "zone of control" extends from anything to anything. The only obstacles are cells the player themselves has already burned.

### 1.4 Scale

- 1 cell = 1 "room" in-fiction.
- Room size is **unspecified** by the rules. A room is narratively atomic: you enter, roll, fight, leave. No intra-room position exists. A "room" in Solo Dungeon Bash is closer to a **node in a graph** than to a physical space.
- The grid itself has no declared scale in metres. The game does not even commit to whether rooms are connected by doors, corridors, open archways, or just holes in walls.

### 1.5 What the spatial model is NOT

- It is **not** a map. There are no hallways, no T-intersections, no dead ends (until you create them), no doors, no keys, no walls between cells. Every pair of king-adjacent cells is connected by default.
- It is **not** line-of-sight restricted. Rolls happen only on entry; no peeking.
- It is **not** 3D or multi-level (the 10 "Dungeon Levels" are **rows**, not Z-layers).
- It is **not** continuous. All movement is discrete cell-to-cell.

---

## 2. Abstract vs Concrete Spatiality

This is the crux of the translation. Solo Dungeon Bash is **maximally spatially abstract** — its "space" is basically an adjacency graph dressed up as a grid. A cell isn't a *place* in any meaningful sense; it's a **slot into which random content is injected the moment you step on it**. You don't navigate a dungeon; you paint one.

### 2.1 The three legitimate readings

When translating to a real-time game, the cell-as-what question has three defensible answers:

1. **Cell = physical room.** Each cell becomes a walled chamber with doors/openings to its 8 neighbours. Entering the next cell = walking through a doorway and triggering a transition. Closest to Binding of Isaac, Enter the Gungeon, or old-school roguelike room rendering.

2. **Cell = tile in a continuous world.** Each cell becomes a region of terrain in a seamless dungeon. The player walks freely across tile boundaries without transitions. Closest to Hades' room-to-room flow, or a Zelda overworld.

3. **Cell = abstract node with presentation layer.** Each cell becomes a UI card or 3D diorama that the player "visits" without any physical traversal. Closest to Dungeon Encounters, Slay the Spire, or FTL.

### 2.2 Why this matters

The source game has **no spatial content to preserve inside a room**. If we pick option 1 (physical room), we invent geometry the source never had. If we pick option 2 (continuous world), we lose the discreteness that made the roll-and-write work. If we pick option 3 (abstract), we keep the source model pure but risk making the game feel like another deckbuilder and losing the "real-time" mandate.

The correct move, given RealTimeForge's brief, is **option 1 with deliberate constraints**: every cell becomes a literal room, but rooms are kept **small, tight, and rapidly traversable**, so the grid still feels like a grid, not a sprawling dungeon. The *content* of a room (a monster, a chest, a potion, nothing) becomes a real thing you fight or pick up in real time, but the *structure* (king-adjacency, no-revisit, rows-as-levels) survives verbatim.

The abstraction we inherit is a gift: because the source is silent on what a room looks like, we get full creative freedom to define it, and we should use that freedom to make each room read in under a second.

### 2.3 What we will NOT do

- We will not convert the grid into a continuous open dungeon. The adjacency/no-revisit rules are the *whole puzzle*; dissolving them dissolves the game.
- We will not turn it into a Slay-the-Spire-style pure node map. The "real-time" directive means combat and movement must be physically embodied. A pure card/node interface violates the brief.
- We will not add levels, Z-depth, stairs, or any third spatial axis the source doesn't have.

---

## 3. Primary Spatial Format Decision

### 3.1 The decision

**Top-down 2D, tile-based, with room-tight cameras and snap-to-grid movement.**

Reference points: *The Binding of Isaac*, *Enter the Gungeon*, the original *Legend of Zelda* (dungeon mode), *Children of Morta* (dungeon sections). The player is rendered as a small sprite (~32-48 px) in a flat overhead view of a square room (~192×192 px or ~384×384 px). The camera shows one room at a time; when the player steps through a doorway, the view slides to the next room.

### 3.2 Why top-down 2D

**Because Solo Dungeon Bash is a 3-page rulebook with no fiction and no spatial detail**, the digital adaptation should honour that minimalism. Top-down 2D is the spatial format that:

1. **Reads instantly.** A top-down room shows all its contents at a glance — no camera manipulation, no occlusion, no perspective to parse. This matches a game where each room is a 1d6 table roll.
2. **Preserves the grid literally.** A grid of square rooms in 2D is a grid of square rooms. You can still draw the player's traced path over a full-map overview.
3. **Cheap to build.** The source is a 3-page print-and-play. The digital adaptation should not explode in production cost. Top-down 2D is the cheapest viable real-time format.
4. **Supports the hand-drawn aesthetic** the GDD (Appendix A) already recommends. Pencil-on-graph-paper art lives natively in 2D.
5. **Makes "no revisit" physically enforceable.** A collapsing doorway or sealed archway behind the player is a trivial 2D effect.
6. **Matches the input expectation for PWA/mobile.** The GDD targets web and mobile; top-down tap-to-move or virtual-stick WASD is the idiomatic mobile format.

### 3.3 Why not the alternatives

- **Isometric 2.5D (Hades, Diablo 1).** Too much production cost and visual noise for a 3-page source. Isometric also obscures grid alignment — players have to translate between the fictional grid and a rotated view, which dilutes the "you are on square (x,y)" clarity the source depends on.
- **Side-scrolling 2D (Dead Cells, Rogue Legacy).** Fundamentally wrong topology. Side-scrollers have gravity and a single horizontal movement axis. Solo Dungeon Bash's 8-way king-adjacency simply cannot live in a side-scroller without becoming a ladder-and-platform puzzle that has nothing to do with the source.
- **3D third-person (Darkest Dungeon 2, Returnal).** Even more production cost than isometric, and worst of all, *breaks the roll-and-write charm*. The source game's appeal is that it's small and intimate. A 3D camera makes it feel bloated.
- **3D first-person (Legend of Grimrock, Dungeon Encounters).** Tempting, because Grimrock is itself a king-move dungeon crawler, but first-person actively harms our puzzle layer: the player can't see the whole grid, can't plan a path, and can't visually confirm they're not blocking themselves. The self-blocking puzzle dies in first-person.
- **Abstract/HUD-based (FTL, Slay the Spire).** Violates the real-time brief. We need embodiment; a 2D tile view is the minimum viable embodiment.

### 3.4 One-line justification

**Solo Dungeon Bash is a grid you draw with a pen; top-down 2D is a grid you draw with pixels. Everything else is overkill.**

---

## 4. Spatial Translation Table

| Board Game Concept | RT Equivalent | Notes |
|---|---|---|
| **Grid cell (row r, col c)** | A square **Room** in the 9×10 grid, rendered as a self-contained top-down chamber roughly 6×6 to 8×8 tiles in size. Has fixed world-space coordinates derived from (r, c). | Room layout is consistent at a given (r, c) — entry side faces the previous room, exits face the 8 king-neighbours. |
| **Start square** | **Entrance Hall** — a slightly larger chamber below the grid, with a single exit north into (row 1, col 5). Renders as a lit torch-lit foyer. No content roll; the player just walks in, sees the first three glowing doorways (to columns 4, 5, 6 of row 1), and picks one. | This is also where the UI intro / pre-run setup lives: shop tutorial appears here. |
| **End square** | **Boss Chamber** — a larger chamber above the grid with Dracular visible (or in silhouette) the moment the player enters. Triggers a forced boss fight on the same frame the player crosses the threshold. | Treated as an arena, not a corridor. Camera pulls back slightly for boss readability. |
| **Row (Dungeon Level 1–10)** | **Depth Band** — a horizontal stripe of 9 rooms with a shared aesthetic (lighting, floor tile, ambient sound) that signals "how deep you are." Level 1 is dimly-lit sandstone; level 10 is volcanic obsidian with red glow. The Level Table used on entry is derived purely from the room's row. | This is where the *only* environmental storytelling lives. The rows ARE the level design. |
| **King-adjacency (8-way)** | **8 physical doorways per room** — 4 cardinal + 4 diagonal. Diagonal doorways are rendered as archways cutting the corner of the room. Each doorway is a trigger volume; crossing it moves the player into the next room. Doorways whose target is out-of-grid or already-visited are sealed (shown as collapsed stonework). | This is the spatial translation most at risk. Diagonal doorways feel unusual in top-down games — see section 5.2 for how we sell them. |
| **No-revisit rule** | **Doorways collapse behind you.** The moment the player enters a new room, the door they came through visually slams shut (portcullis falls, rubble piles, stone slab). A collapsed door is unclickable and un-walk-through for the rest of the run. In the map overlay, visited rooms are drawn in ink (as they were on graph paper). | This is the *primary* translation of the draw-your-own-line tactility. Instead of drawing a line, you leave a trail of sealed doors. |
| **Path line (drawn by pen)** | **Map overlay** (toggled with a key, e.g. M or Tab). Shows the 9×10 grid drawn on virtual graph paper, with the player's actual traced path inked in as a polyline through cell centres, exactly as the source did. Current room glows; Start and End protrude. | This overlay IS the roll-and-write heritage, made explicit. It's the tactility transplant. |
| **Blocking self (BFS fail)** | **Path-blocked warning** — a BFS runs every time the player approaches a doorway. If committing to that door would make the End unreachable, the door glows red and a "Dead end ahead?" indicator appears. Committing anyway is legal (strict A-1 rule) but irrevocable and a likely loss. On confirmed block, the game shows a "You are lost in the dungeon" screen. | Digital-only affordance, explicitly allowed by the GDD. |

### 4.1 Translations of things that are not in the source but emerge from the real-time format

| New Concept (digital) | Purpose |
|---|---|
| **Doorway trigger volume** | The physical act of "moving to adjacent cell" becomes walking into a trigger. |
| **Room arena bounds** | The physical fighting space. Keeps combat readable. |
| **Collapsed-door VFX** | Sells the no-revisit rule physically. |
| **Map overlay UI** | Restores the roll-and-write tactility the real-time format cost us. |
| **Highlight on available doors** | The 8 king-adjacent doors are lit; collapsed / out-of-grid ones are dark. |

---

## 5. Movement Model

### 5.1 Continuous vs discrete vs commit-to-room

**Continuous inside a room, commit-to-room across doorways.**

- **Inside a room**: the player has full real-time WASD / virtual-stick / tap-to-move control. They dodge around monsters, walk over treasure, and navigate to whichever doorway they want. This is normal top-down action movement.
- **Across rooms**: the moment the player steps into a doorway trigger, the game commits — the camera slides to the new room, the level table rolls, and the old room's doorway visually seals.

This matches the turn structure of the source:
- "Inside a room" = steps 2–7 of the turn sequence (roll, resolve content, fight, drink potions, shop).
- "Step through doorway" = step 1 of the *next* turn (move to adjacent cell).

Critically, the game is **real-time inside a room** and **discrete at the room boundary**. This is how we get to call it a real-time game without dissolving the grid.

### 5.2 How 8-way adjacency survives

Diagonal doorways are the single most novel spatial element. In typical top-down games rooms connect orthogonally (N/S/E/W). We need 8 connections.

**Solution: corner archways.** Each room has up to 8 doorways, rendered as:
- 4 cardinal doorways in the middle of each wall (north, south, east, west).
- 4 diagonal archways cutting the four corners of the room at a 45° angle. The corner of the room is chamfered; the archway leads "through the wall at the corner" into the diagonally-adjacent room.

Visually this reads as a **chamfered square** — an octagon, really — with 8 exits. Not realistic architecture, but no one cares; the source didn't care, and the player sees the logic immediately on their first room.

Each doorway's trigger volume is independent. Walking into the NE corner archway commits to cell (r-1, c+1). Walking into the N door commits to (r-1, c).

If that feels too geometrical for production art, an equally-legal alternative is:
- Square room with 4 cardinal doorways, and diagonal movement is handled by walking through a corner of the current room into a *liminal "junction cell"* that auto-forwards to the diagonal neighbour. This is uglier but less bespoke art. **Recommend sticking with octagonal chamfered rooms.**

### 5.3 What happens to the "no revisit" constraint?

**The doorway physically seals the moment the player commits to leaving.** Implementation:

1. Player walks into doorway trigger volume.
2. Camera cross-fades to the next room; level table rolls and room is populated.
3. As the cross-fade completes, the door on the *old* room (facing the new room) plays a collapse animation — portcullis drop, stone slab, rubble, whichever fits the art style.
4. The *new* room's door-from-previous is likewise sealed; when the player turns around and walks toward it, they bump into rubble and can't pass.
5. The map overlay inks in the traversed edge as a solid line.

Visited rooms exist in the fiction but cannot be physically revisited. They *could* be shown on the map overlay as ghosted rooms with their contents remembered (treasure collected, monster killed) — a nice digital touch that doesn't break any rules.

**No, we do NOT "destroy" visited rooms.** They persist on the map overlay. The sealing is door-level, not room-level.

**No, we do NOT leave them open but empty.** That would let the player physically re-enter a room and then be told "oh, you can't actually do anything here." Confusing, and breaks the fiction. Seal the doors.

### 5.4 Special cases

- **Start → first room.** The Start cell has a single exit, up into row 1. That exit has 3 sub-doors (to columns 4, 5, 6) that fan out in the Entrance Hall's north wall. The player picks one by walking into it.
- **Row 10 → End.** From any column of row 10, the "up" direction normally leads out of the grid. Only from (row 10, col 4/5/6) is there a doorway leading into the End/Boss chamber. From other row-10 columns, the "up" doorway is sealed shut (never exists).
- **Grid edges.** On any row-1 cell, the "south" doorway exists only for column 5 (to Start). Elsewhere it's sealed. Similar for column edges.
- **The player can still choose to NOT move.** In a real-time format this means "hang around in the current room drinking potions and shopping." This maps to steps 6–7 of the source turn sequence. The player commits to movement only by stepping through a doorway.

### 5.5 Dodging vs positional combat

A subtle question: since the source combat is purely dice-based with fixed initiative, does the player actually **dodge** in the real-time combat?

This crosses into Stage RT-2's territory, but spatially we must answer: **yes, combat is dodgeable, but the dice math still decides hits, not spatial collision**. Monster attacks telegraph and project hit volumes; the player can dodge out of the volume to reduce incoming hit count without rolling defence dice. This preserves the real-time feel while still respecting the d6 combat skeleton. The room needs enough space for a few dodge-rolls — hence the 6×6 to 8×8 tile room size baseline.

---

## 6. Camera Design

### 6.1 Default camera: one-room-at-a-time

The camera is **room-framed, static within a room**. When the player is in room (r, c), the camera is centred on that room's world-space origin, showing the full room and its 8 doorways. The player moves inside the frame; the camera does not track the player.

This matches The Binding of Isaac exactly, and the reason is identical: rooms are small, the player needs to see all enemies, projectiles, and exits at once, and room-framed cameras make spatial decisions fair.

### 6.2 Transitions

When the player commits to a doorway, the camera **slides** to the next room's frame over ~200–300 ms. The slide direction matches the movement direction (so a NE commit slides the camera up-right). During the slide, input is briefly locked, the level-table roll animation plays in a corner, and the new room's contents fade in as the slide completes.

### 6.3 Map overlay (secondary camera / UI mode)

Pressing M / Tab toggles the **map overlay** — a semi-transparent layer showing the full 9×10 grid as handwritten graph paper. Visited rooms are inked, current room is green-highlighted, Start and End are labelled. The player's path is drawn as a pen line between cell centres. This is NOT a real camera; it's a UI panel over a blurred game view. Gameplay pauses during map view (the game is turn-based between rooms anyway, so this is free).

The map overlay also lets the player see at a glance whether they're at risk of blocking themselves.

### 6.4 Presentation shots

- **Boss room entry:** short camera pull-back and slow zoom-in on Dracular, roughly 1.5 seconds, diegetic music sting. This is the only cutscene.
- **Death:** camera zooms on the player sprite, fade to red, fade to "You died" screen with run summary.
- **Victory:** slow pan across the full map overlay with Dracular's slain body and the traced path lit up.

### 6.5 Zoom

No user zoom control. The camera is locked at a fixed zoom per room. Deviating from this would let the player see upcoming rooms, which violates the source's "content is revealed on entry" rule.

---

## 7. Map Design Philosophy

### 7.1 Generation model

**Each run: fresh procedurally-generated content in a fixed topological skeleton.**

The **grid itself is static** — always 9×10, always Start below column 5, always End above column 5, always king-adjacency. This is the source game's skeleton and must not change.

The **contents** of each cell are decided per-run:
- Room geometry (wall variants, decoration, door positions) is drawn from a small pool (3–6 variants per depth band).
- Room content (monster / treasure / potion / empty) is decided by the level table roll on first entry, NOT by pre-baked seeds, exactly as the source does it. Content is *not* pre-generated at run start; it's generated on first entry. This matters — it preserves the source's "you don't know what's in a room until you enter" property and makes save-scumming harder.
- Monster placement within a room is randomized from a small pool of spawn patterns.

This is **lazy procedural generation**: the grid is fixed, the topology is fixed, and only the contents are rolled, and only on first entry.

### 7.2 Optional runs

- **Daily seeds** (mentioned in GDD §5 opportunities) fix the random seed so all players face the same rolls in the same cells. The grid is still 9×10 and identical; only the level-table rolls are seeded.
- **Endless / no-boss** mode (post-MVP) could relax the 10-row cap. Out of scope for RT-1.

### 7.3 What stays hand-crafted

- The grid dimensions (9×10).
- The depth-band art themes (10 themes, one per row).
- The Start and Boss rooms — fully hand-crafted arenas.
- The Level Tables — fixed data per the source.
- Room geometry pool — hand-authored variants.

### 7.4 What is procedural per run

- Which room geometry variant each cell uses.
- Which monster spawn pattern plays when a monster is rolled.
- The d6 level-table roll result.
- The shop's available stock (always all items, but maybe randomized display order — cosmetic).

### 7.5 Not procedural

- Grid topology (always 9×10 king-grid with fixed Start/End).
- King-adjacency rule.
- No-revisit rule.

---

## 8. Scale & Proportion

Concrete numbers, to feed downstream stages (architecture, balance, assets).

### 8.1 Room size

- **Tile-grid unit:** 32 pixels per tile (standard indie 2D tilemap size).
- **Room footprint:** 8 × 8 tiles = 256 × 256 px = roughly 8 m × 8 m in fictional space (1 tile ≈ 1 metre).
- **Playable interior:** 6 × 6 tiles (1-tile border for walls/doors).
- **Player sprite:** 32 × 32 px (1 tile), normal walk speed ~4 tiles/sec.
- **Time to cross a room:** ~1.5 seconds corner to corner, ~1 second centre to doorway.

### 8.2 Full dungeon footprint

- 9 columns × 256 px wide = 2304 px. Plus a 32 px gap between rooms for walls = ~2592 px wide.
- 10 rows × 256 px tall = 2560 px. Plus gap = ~2880 px tall. Plus Start (below) and End (above), say 512 px each = **~3900 px tall**.
- **Total world space:** ~2600 px × ~3900 px. This is comfortable for the map overlay — fits at ~30% scale on a 1080p screen.

### 8.3 Turn time

- **Expected time in a room:** ~5 seconds empty, ~10–20 seconds with a monster, ~30–60 seconds with a hard monster + shop visit.
- **Full run (10-row minimum path):** ~11 rooms minimum, ~180 seconds best case. Typical run with farming: 30–60 rooms, ~20–40 minutes. **This matches the GDD's stated session length.**
- **Boss fight:** ~30–90 seconds.

### 8.4 Minimap scale

The toggled map overlay shows the full 9×10 grid at a scale where each cell is ~48 px on screen (1/5 to 1/6 of its in-world size). Total overlay: ~500 × ~700 px centered on screen. Readable and glanceable.

### 8.5 UI chrome budget

Outside the room view, HUD takes ~150 px on the right side (player tracker — HP, AD, DD, Treasure, Potions) and ~80 px on the top (current depth band label, minimap toggle hint). Room view is ~1200 × ~900 px on a 1920×1080 window; single-room view dominates the screen.

---

## 9. Procedural Generation Implications

### 9.1 What the 9×10 grid + level tables tell the generator

- **Exactly 10 depth bands.** The generator must produce 10 themed tilesets (sandstone, mossy stone, crypt, library, magma, etc.). This is a design/art deliverable, not a runtime algorithm.
- **90 cell geometry instances.** Each cell is populated at run start with a geometry variant chosen from its depth band's pool. 3–6 variants per band × 10 bands = 30–60 hand-authored room variants. Modest authoring cost.
- **One level table per row.** The d6-to-content mapping is hardcoded in data (already extracted in RulesExtraction.md). The generator reads it, rolls, and populates the room on first entry.
- **Monster spawn patterns.** For each of the 10 monster types + Dracular, 2–4 spawn patterns (e.g., "single in centre," "two flanking doorways," "patrolling in circle"). Roughly 25–40 patterns total.

### 9.2 Generator inputs

- Random seed (optional, for daily/replay).
- Grid dimensions (fixed at 9×10).
- Level table data (from `RulesExtraction.md`).
- Depth-band asset pool.

### 9.3 Generator outputs

- 92-cell `Room[,]` array where each cell has: `rowIndex, colIndex, geometryVariantId, visited: false, content: null`.
- `content` is filled lazily on first entry: `{ rollResult: 1..6, contentType: Monster|Treasure|Potion|Empty|Boss, monsterData?, spawnPatternId? }`.
- Start and End rooms are singletons, always present, never generated.

### 9.4 What MUST NOT be procedural

- **The grid topology itself.** Changing it changes the game. The player's self-blocking puzzle depends on 9×10 being 9×10.
- **The level tables.** They are the source's balance. Regenerating them would decouple the digital game from the source.
- **King-adjacency.** See above — this is the game.

### 9.5 What COULD be procedural but shouldn't be in MVP

- Room geometry variants are the only real procedural hook and can even be dropped in MVP (1 variant per depth band).
- Shop item randomization — keep the full catalogue always available; don't randomize.
- Music / ambience — use fixed tracks per depth band.

**Summary: procedural generation in this game is very thin.** The source is so abstract that procedural content is mostly cosmetic. This is a feature, not a bug — it makes the project small.

---

## 10. Spatial Feel

### 10.1 The target feel

**Tight, claustrophobic, cramped — and deliberately unsprawling.** Each room is small. Doorways are visible. The map is walkable end-to-end in a few minutes if the player is just rushing. The full 9×10 is **knowable** — a player who has played 10 runs should be able to close their eyes and picture the whole grid.

This is the opposite of most modern roguelikes, which spray content across a huge procedural space. Solo Dungeon Bash's charm is that the whole world is 90 rooms. The digital version should feel correspondingly bounded.

### 10.2 Labyrinthine vs simple

**Simple, not labyrinthine.** The grid has no dead ends by default — only those the player creates. The spatial challenge comes entirely from *self-inflicted* blocking, which is cognitive (planning routes), not navigational (finding the path). The dungeon should not feel like a maze. It should feel like an honest grid where the player is drawing their own maze through it.

### 10.3 Vertical vs horizontal

**Vertical, conceptually.** Row 1 is "up near the surface," row 10 is "deep down." The Start cell is below row 1 in the source because you *enter from outside*; the End is above row 10 because the boss lives at the top of the dungeon — actually in the source, that's ambiguous, but for our digital version we commit to the reading: **the dungeon is a tower-climb with Start at the bottom and End/boss at the top**. Row numbers flipping orientation ("Level 1" is the shallowest/lowest row visually, "Level 10" is the deepest/highest row — deepest into the tower) is fine because that's exactly what the source does.

Wait — the source has Start *below* row 1 and End *above* row 10, with Level 10 being "deepest." That means in the source, "up the grid" means "deeper into the dungeon." The digital adaptation should preserve this: the player walks upward on the screen as they descend into the dungeon. This is slightly counter-intuitive but it's the source's layout and we respect it.

Alternative: render the grid upside-down so the player walks visually downward into depth. This is cleaner narratively but *inverts the source's diagram*, which affects any player who has seen the original. **Recommendation: match the source — Start at bottom, End at top, row 1 visually at the bottom, row 10 visually at the top. Fiction: "the boss sits atop the accursed tower."** This converts a dungeon into a tower climb, which is a small but defensible reframing.

### 10.4 Open vs closed

**Closed and funnelled.** The player is always in a bounded room. There is no open space, no outdoor vista, no skybox. The only visual relief is the depth-band themes shifting as the player ascends.

### 10.5 Pacing

- **Shallow rooms (rows 1–3)** feel like warm-up: quick combats, loose rooms, lots of empty cells.
- **Mid-dungeon (rows 4–6)** intensifies: combat density climbs, the shop starts mattering, the player starts checking the map overlay.
- **Deep rooms (rows 7–10)** are survival mode: every room can roll a high-AD monster, potions get scarce, the player plans every doorway.
- **Boss room** is the crescendo: 9 AD / 9 DD, 1 HP each side, single decisive fight.

---

## 11. Flagged Issues

Things that resist spatial translation and need product-level decisions later.

### 11.1 The tactility of drawing a line

**The single biggest translation loss.** In the print version, the player *draws* with a pen. The feel of the pen scratching cell to cell *is* the game for many players. We cannot fully replicate this in a real-time 2D game.

**Mitigation:** The map overlay shows the traced path as an inked polyline, growing in real-time as the player enters rooms. We add a **"pen scratch"** SFX on every doorway commit. We render visited doorways and the map overlay with hand-drawn/sketchy art. This recovers ~70% of the feel but not 100%.

**Alternative mitigation (stretch):** Tablet/touchscreen stylus mode where the player literally drags a finger/stylus from room to room on the minimap, and that draws both the line and moves the character. This leans into the tactility but complicates input on keyboard/mouse. **Defer to RT-4 (input model).**

### 11.2 Diagonal doorways in top-down art

Octagonal chamfered rooms are unusual in top-down games. Players may take one room to understand. **Mitigation:** the onboarding tutorial (first run / first 3 rooms) highlights all 8 doorways explicitly with directional arrows. After ~5 rooms the player has internalized king-movement. Not a fatal issue, but an art direction constraint: every room tile set must support 8 exits cleanly.

### 11.3 Row backtracking feels weird in a "tower climb"

The source allows the player to move back down levels (row 6 → row 5 → row 4 via diagonal moves). In a tower fiction, going "down" looks like retreat. **Mitigation:** This is fine. Retreat is legal in the source and backtracking is a real strategic option (e.g., dipping into a lower row to farm safer rooms). The fiction can absorb it: "you retrace your steps into already-cleared levels." The only oddness is the doors sealing behind you, which limits how much retreat is actually possible. This is exactly correct per the source rule.

### 11.4 The BFS "blocked" warning

Committing to a door that blocks your path to End is legal in the source but is a game-ending mistake. In the source, the player must self-police. In a real-time game, allowing the player to accidentally doom themselves without warning feels cheap; over-warning feels patronizing. **Mitigation:** Path-blocked warning per §4 translation table. The door glows red and a subtle "Dead end ahead?" label appears. The player can still commit. This is a **quality-of-life digital affordance explicitly endorsed by GDD §5**. We preserve strict A-1 semantics but surface them through UI.

### 11.5 "Content generated on first entry" vs save-scumming

In a real-time digital game, save-scumming is a real concern. If content is generated on first entry with a deterministic seed, the player can reload a save and the rolls stay the same. If it's non-deterministic, reloading gives new rolls. **Recommendation:** MVP uses non-deterministic rolls (fresh seed on every entry), preserving the source's "every roll is a new risk" feel. Daily seeds and optional "ironman" mode ban reloading. **Defer to RT-5 (persistence model).**

### 11.6 Real-time dodge vs dice-roll hits

Alluded to in §5.5. The spatial format (top-down with room arenas) invites traditional action-combat where the player dodges projectiles. But the source uses count-6s dice math. **Mitigation:** dodges reduce the number of *monster attack dice that roll against you this round*, rather than dodging discrete projectiles. The dice math stays canonical; the dodge is a pre-roll modifier. This is a hybrid that preserves the source math while giving the player real-time agency. **Deferred to RT-2 (combat translation).**

### 11.7 Room layout consistency across runs

If the same cell has different geometry variants on different runs, it undermines the "knowable 9×10 grid" feel from §10.1. **Recommendation:** Daily/seeded runs use a fixed geometry per cell; free runs use variants. OR use fixed geometry always, and let only content vary. **Leaning toward: fixed geometry per cell, variants are deprecated — every cell has ONE look, and runs differ only in contents.** This makes the dungeon feel like a *place* the player learns, not a new maze each time. Confirms the "knowable" feel.

### 11.8 Lack of in-fiction explanation for "doors seal behind you"

The source doesn't care — it's a pen-on-paper abstraction. In a real-time game we need a diegetic reason. **Options:**
- The dungeon is cursed: Dracular's magic seals each doorway to prevent escape.
- The tower collapses behind you: unstable stonework crumbles as you pass.
- You are trailing a magical rope/thread that blocks your path back (metaphorical reverse Ariadne).
- No fiction — just a game rule, sold through UI.
**Recommendation:** "Dracular's curse seals each doorway" — one sentence in the intro, never spoken again. Cheap, clean, fits the minimal lore budget.

### 11.9 The grid extensions (Start, End) are special but small

Rendering Start and End as single cells the same size as regular rooms makes them visually underweight relative to their narrative importance. **Recommendation:** Both Start (Entrance Hall) and End (Boss Chamber) are rendered as **1.5× or 2× size rooms**, breaking the grid visually to signal "this is special." The adjacency rule is unchanged — they still connect only to their 3 middle-column neighbours. This is pure art scaling, no rules impact.

### 11.10 Camera slide direction on diagonal commits

When the player commits to a NE corner doorway, does the camera slide NE (diagonally)? That looks weird and is uncommon in top-down games. **Recommendation:** the camera slides *diagonally* for diagonal commits, matching the player's movement direction. It's unusual but it's *correct* — it reinforces king-adjacency visually. First-run players might be briefly confused; by room 5 it's internalized. Alternatively, slide cardinally (say, east first, then north) to show the diagonal as a compound move, but this takes twice as long and feels wrong for a single turn.

---

## 12. Summary

**Primary spatial format: Top-down 2D, room-at-a-time camera, snap-to-grid room transitions, continuous control inside each room, sealed doorways behind you, and a toggleable hand-drawn map overlay that restores the roll-and-write tactility.**

This preserves every spatial rule from the source (9×10 king-grid, no revisit, self-blocking) while giving the player real-time embodiment (dodge, walk, fight in a physical room). It honours the source's abstraction (rooms have no intrinsic geometry, only what we give them) and its minimalism (production cost is a fraction of 3D). And it matches the GDD's stated targets (web/mobile, hand-drawn aesthetic, 20-40 minute sessions).

The only meaningful translation losses are the literal pen-on-paper tactility (mitigated by the map overlay + pen SFX + sketchy art) and the awkwardness of diagonal doorways in top-down art (mitigated by octagonal chamfered rooms and tutorial highlighting). Everything else transfers cleanly.

Downstream stages (RT-2 combat translation, RT-3 temporal model, RT-4 input model, RT-5 persistence) should treat this spatial decision as load-bearing: they can adjust how things *feel* inside a room, but the 9×10 grid + king-adjacency + no-revisit + room-at-a-time camera is fixed.
