# Solo Dungeon Dash — Asset Pipeline Specification

**Document:** RT-A4 Asset Pipeline
**Project:** Solo Dungeon Dash (Godot 4 real-time adaptation)
**Team scope:** 1 programmer + 1 artist + part-time audio
**Target platforms:** PC / Steam Deck / Mobile (touch)
**Aesthetic:** Hand-drawn ink on parchment — "a 1970s dungeon master's notebook"
**Status:** Draft v1, feeds into RT-D1 (Prototype Prompts) and RT-P1 (Production Schedule)

---

## 1. Visual Style Guide

### 1.1 Philosophy

> **Draw everything as if Felbrigg Herriot himself sketched it in a 1970s dungeon master's notebook.**

Every visual asset in Solo Dungeon Dash must read as if it were **hand-drawn in wobbly ink over an afternoon of tea and dungeon mapping.** The fantasy is not "retro pixel art" and not "clean vector illustration." It is a **handcrafted artifact** — the player is peering into the private notebook of a game master who sketched this entire dungeon run from memory after the session ended.

Core tenets:

1. **Wobble is a feature, not a bug.** Lines should not be geometrically perfect. Circles are slightly oblong. Rectangles lean by 1-2 degrees. This is deliberate.
2. **Every asset is ink on paper.** No digital gradients, no anti-aliased vector curves, no glow effects. Light comes from crosshatch density, shadow comes from parallel line accumulation.
3. **Color is the exception, not the rule.** Most of the world is black-ink-on-parchment. Color is reserved for specific semantic meaning (gold = treasure, red = danger, etc.) so that when color appears, it signals.
4. **Imperfection is canonical.** A slightly off-center sword tip, a wobbly heart outline, a hatch pattern that wanders slightly — all of this reinforces the handmade feel. DO NOT normalize away these variations.
5. **The parchment is alive.** The background is never pure white. It has yellowing, faint stains, a subtle fiber texture. It feels old.

### 1.2 Color Palette

The palette is **intentionally small** — seven colors total. If an asset needs a color not on this list, reject the asset.

| Name              | Hex       | Usage                                                  |
| ----------------- | --------- | ------------------------------------------------------ |
| Parchment Beige   | `#F4E9D0` | Primary background; the "paper" itself                 |
| Aged Parchment    | `#E8D9B0` | Corner stains, worn edges, old map feel                |
| Ink Black         | `#1A1A1A` | All line work, all text, all shading crosshatch        |
| Ink Shadow        | `#2E2A25` | Optional deep shadow under objects (use sparingly)     |
| Treasure Gold     | `#D4AF37` | Coins, loot, dice-of-fortune faces, altar glow         |
| Potion Blue       | `#2E64A0` | Potions, defense dice, safe UI accents, heal VFX       |
| Danger Red        | `#CC4444` | Enemies, attack dice, damage flash, critical warnings  |
| Safe Green        | `#4A7C3A` | Current-cell indicator, healing confirmation, "go" UI  |
| Visited Grey      | `#8A8378` | Explored-but-empty cells, fog-of-war, disabled UI      |

**Rule:** No asset may contain more than **3 non-ink colors** simultaneously. Ink black always dominates.

### 1.3 Line Weight

- **Primary outlines:** 2.5px variable weight, with natural taper (thicker at stroke start, thinner at stroke end).
- **Interior detail lines:** 1.5px.
- **Crosshatch shading:** 1px.
- **Wobble:** all lines should have **~5% pixel-level jitter** along their length. Perfectly straight lines are forbidden except when intentionally architectural (pillars, walls) — and even then, a 1-degree lean is preferred.
- **Ink bleed:** line ends may bloom slightly (a small round dot of ink) to simulate a pen stopping and lifting.

### 1.4 Shading

- **Crosshatch only.** No gradients, no soft shadows, no painted fills.
- **Shading density** is the only tonal control:
  - 0 hatches = highlight / parchment showing through
  - Single hatch = light shade
  - Crosshatch (two directions) = mid shade
  - Triple hatch (three directions) = deep shadow
  - Solid ink fill = only for tiny eye sockets, pupils, or ink splats
- Hatching should **follow form** — curve hatches around curved surfaces, not grid-locked.

### 1.5 Textures

- **Background parchment** is a loaded canvas:
  - Base: `#F4E9D0`
  - Subtle fiber texture (tileable noise, 5% opacity)
  - Aged yellowing in corners using `#E8D9B0` radial vignette
  - Occasional faint coffee-ring or ink-splatter decoration (1-2 per scene)
- **Paper grain** should be subtly visible underneath all assets — achieved via a single global parchment texture layer that shows through partially transparent sprites.

### 1.6 Typography

- **Headers / titles / labels:** Caveat or Homemade Apple (Google Fonts, free, permissive license). Handwritten, wobbly.
- **Numbers / HUD readouts:** Inter, regular weight. Clean but not clinical. Numbers must be legible at a glance — readability > theme here.
- **Body text (tutorials, dialog):** Caveat Regular at 18-24pt.
- **Never use:** Comic Sans, Papyrus, or any "fantasy" display font. The aesthetic is a **real** sketchbook, not a themed Renaissance faire.

### 1.7 Reference Touchstones (non-linked, for style conversation)

When the artist needs a reference point, reach for these:

1. **Edward Gorey** — crosshatch density, wobble, macabre whimsy, ink-on-paper feel.
2. **Night in the Woods** — linework weight, selective color, character readability in top-down.
3. **Slay the Princess** — charcoal/ink character study style, emotional line weight variation.
4. **Dungeon Meshi (marginalia illustrations)** — bestiary sketch feel for monster art.
5. **1970s TSR module illustrations (Erol Otus, David Trampier)** — spiritual ancestors; wobbly monsters in wobbly dungeons.

### 1.8 Inconsistency Budget

**Hand-drawn means some variance is explicitly allowed.** Do not over-polish.

- Frame-to-frame animation may have **~3% wobble drift** — this is fine, it adds life.
- Two different rooms may have slightly different hatching density — fine.
- A monster idle may have 4 subtly different poses — fine.
- **What is NOT fine:** style drift between the same asset (e.g., the Orc looking "clean" in frame 1 and "sketchy" in frame 4), or obvious digital tool marks (smooth Bezier curves, pressure-sensitive gradient strokes that look digital).

**Rule of thumb:** if two random pixels in the asset look 100% identical in color and tone, you're probably over-polished. Add a little noise.

---

## 2. Character / Entity Asset List

### 2.1 Player Character

Top-down 2D perspective. Sprite resolution: **64x64 source, rendered at 128x128** (2x scale). The player is a hooded adventurer with a small sword silhouette readable at a glance.

| Animation | Frames | Notes                                                                    |
| --------- | ------ | ------------------------------------------------------------------------ |
| Idle      | 4      | Breathing loop, cloak sway, 8fps                                         |
| Walk      | 8      | Top-down cycle, 4 directions handled by sprite rotation + mirroring      |
| Attack    | 6      | Sword swing arc; impact frame at frame 4; 12fps                          |
| Parry     | 4      | Shield raise + spark frame; impact frame at frame 3                      |
| Dodge     | 6      | Roll animation, invincibility frames 2-4; ends with recovery pose        |
| Hurt      | 2      | Red flash + recoil; 2 frames is enough for a readable flinch             |
| Death     | 4      | Collapse into ink puddle; final frame is a small ink splat decal         |

**Player frame subtotal: 34 frames.**

Visual notes:
- Hooded figure, ink silhouette, no facial detail (keeps budget low, supports player projection).
- Cloak trails slightly behind movement — one hatch pattern shows motion blur via line direction.
- Sword is always visible in hand even in idle; it is part of the silhouette.

### 2.2 Monsters (11 total)

All monsters use a **shared animation template** to keep production feasible:

**Standard monster animation set (~18 frames each):**
- Idle: 4 frames (breathing / hover / sway)
- Telegraph: 2 frames (the "tell" before an attack — this is CRITICAL for readability)
- Attack: 4 frames
- Guard: 2 frames (the blocking stance)
- Opening: 2 frames (stunned / open to counterattack)
- Death: 4 frames (ink-splat finish)

Each monster's personality lives in the **telegraph** and **attack** animations — those are the feel-differentiators.

#### 2.2.1 Per-Monster Specifications

| # | Monster        | Signature Feel                                  | Animation Notes                                                                 |
| - | -------------- | ----------------------------------------------- | ------------------------------------------------------------------------------- |
| 1 | Orc            | Heavy, slow, telegraphed club slam              | Telegraph = club raised overhead; attack = downward slam; opening = club stuck  |
| 2 | Wolf           | Fast, lunging, multi-strike                     | Telegraph = crouch; attack = lunge-style forward dash; opening = overshoot      |
| 3 | Skeleton       | Clattering rusty sword, loose frames            | Telegraph = sword raise with bone rattle; attack = horizontal slash             |
| 4 | Evil Warrior   | Shield-bash, high guard, hard to crack          | Telegraph = shield forward; attack = bash + follow-up sword; high guard uptime  |
| 5 | Devil Bat      | Aerial, darting, hit-and-run                    | Idle = hover loop; telegraph = altitude drop; attack = dive peck                |
| 6 | Cyclops        | Ground-slam with radial VFX shockwave           | Telegraph = fist raise; attack = two-handed slam; radial VFX spreads outward    |
| 7 | Dark Elf       | Dagger flurry, multi-hit, fast windows          | Telegraph = dagger flourish; attack = 3-hit dagger combo; very short opening    |
| 8 | Skeleton Lord  | Summons minor ranged projectile (bone shard)    | Telegraph = raises hand; attack = spawns projectile; fallback to sword at close |
| 9 | Wizard         | Projectile spell, keeps distance                | Telegraph = hand glow (white hatch highlight); attack = projectile fire         |
| 10| Demon          | Arena-wide scythe sweep, huge opening           | Telegraph = scythe wind-up (3 frames extra budget); attack = full-arena sweep   |
| 11| (extra slot)   | Reserved for late-beta addition                 | Budget carried — keep 18 frames open                                            |

Actually reviewing the rules extraction: only **10 named monsters** + Dracular. Slot 11 is a beta reserve for a potential Skeleton Captain variant if time allows. Treat as optional.

**Monster frame subtotal: 10 × 18 = 180 frames** (+ 18 reserve).

Visual notes:
- Monsters are drawn as **tiny bestiary sketches** — the style is "marginal illustration in a medieval manuscript," not "game sprite."
- Each monster has a **signature ink-splat shape** for death (wolf = pawprint splat, wizard = swirling glyph splat, etc.) — this is a cheap way to add character.
- Sizes vary: Orc ~64x64, Cyclops ~96x96, Devil Bat ~48x48, Demon ~128x96 (arena boss-like).

### 2.3 Dracular (Boss, 3 Phases)

Dracular is **the** boss. Phase transitions must feel like dramatic ink-eruptions. Budget a lot of love here.

#### Phase 1 — "The Gambler"

Dracular sits on his throne, never rises. Rolls dice at the player from afar.

- Throne sprite (static, 128x128): 1 asset
- Seated idle: 4 frames (dice-shaking hand gesture loop)
- Telegraph: 2 frames (raises hand with dice)
- Dice-throw attack: 4 frames (arc motion)
- Phase transition out: 4 frames (stands up, throne cracks)

Subtotal Phase 1: ~15 frames + 1 throne.

#### Phase 2 — "The Duelist"

Dracular walks the arena and duels the player in direct combat.

- Idle standing: 4 frames
- Walk cycle: 8 frames
- Telegraph: 2 frames (blade rising)
- Duel attack A (lunge): 4 frames
- Duel attack B (sweep): 4 frames
- Duel attack C (counter-riposte): 4 frames
- Phase transition out: 4 frames (roars, body ink-cracks, ink begins pouring from silhouette)

Subtotal Phase 2: ~30 frames.

#### Phase 3 — "The Ink Storm"

Dracular dissolves into an ink-wrath form — no longer humanoid, a swirling mass of ink with a single eye.

- Idle hover: 4 frames (ink-swirl loop)
- Telegraph: 2 frames (ink condenses to a point)
- Ink-tendril attack: 4 frames (tendrils lash from center)
- Arena ink-flood attack: 4 frames (ink spreads across floor)
- Death: 6 frames (ink evaporates upward, leaves a single dropped signet ring)

Subtotal Phase 3: ~20 frames.

**Dracular frame subtotal: ~65 frames** (plus throne).

Visual notes:
- Phase 1 Dracular is **precise and composed** — clean hatching, steady lines.
- Phase 2 Dracular has **more energy** — hatching gets slightly more frantic.
- Phase 3 Dracular is **wild** — lines drip, hatching is chaotic, silhouette is undefined. This is where the "ink storm" name earns itself.
- The signet ring is a deliberate narrative hook for post-launch content.

### 2.4 VFX (Per-Ability)

Visual effects are drawn as short frame-based sprite animations (4-8 frames each), NOT as particle systems. The hand-drawn aesthetic requires that VFX look inked too.

| VFX                         | Frames | Notes                                                  |
| --------------------------- | ------ | ------------------------------------------------------ |
| Parry spark                 | 4      | Radial ink-star burst at sword meeting                 |
| Dodge dust                  | 4      | Small hatch-cloud trail behind the roll                |
| Player hit spark            | 3      | Red ink droplets                                       |
| Ink death splash (player)   | 5      | Expanding ink blot; final frame is a permanent decal   |
| Coin shower                 | 6      | Gold coins arcing from killed enemy                    |
| Potion drink                | 4      | Blue sparkle above player head                         |
| Dice roll particles         | 5      | Small dice tumbling at cursor / dice tray              |
| Monster hit sparks (1/type) | 3 each | 10 variants, ~30 frames total                          |
| Cyclops ground-slam radial  | 6      | Expanding shockwave ring                               |
| Wizard projectile trail     | 4      | Looping trail of small ink sparks                      |
| Demon scythe arc trail      | 6      | Red crescent arc following the sweep                   |
| Dracular dice swirl         | 8      | Signature — dice orbit around throne during Phase 1    |
| Dracular ink tornado        | 8      | Signature — Phase 3 idle background effect             |
| Door seal animation         | 4      | Chains wrap around doorway                             |
| Healing glow                | 4      | Green hatching pulse over player                       |

**VFX frame subtotal: ~90 frames** (across ~15 distinct effects).

---

## 3. Environment Asset List

### 3.1 Room Tileset

The dungeon is made of **octagonal chambers** (8 sides, 8 possible doorways). Each room is assembled from modular tile pieces.

| Asset                               | Count | Notes                                                      |
| ----------------------------------- | ----- | ---------------------------------------------------------- |
| Octagonal floor base                | 1     | Single base sprite, ~256x256                               |
| Wall segment (straight)             | 4     | 4 rotational variants (N-S, E-W, diagonals)                |
| Wall segment (corner)               | 4     | 4 corner pieces joining the octagon vertices               |
| Doorway opening variants            | 8     | One per compass direction (N/NE/E/SE/S/SW/W/NW)            |
| Doorway — open (drawn state)        | 8     | Arched, dark interior showing                              |
| Doorway — sealed (drawn state)      | 8     | Chains / bars drawn across                                 |
| Doorway — sealing animation         | 4     | 4 frames of the chains wrapping (shared across all 8 dirs) |
| Wall decoration — pillar            | 2     | Stone pillar variants                                      |
| Wall decoration — torch             | 2     | Lit / unlit variants; lit has VFX flicker                  |
| Wall decoration — cracks            | 3     | 3 crack patterns for visual variety                        |
| Wall decoration — chains            | 2     | Hanging chain variants                                     |
| Wall decoration — bones pile        | 3     | Small, medium, and large bone piles                        |

**Room tileset subtotal: ~47 sprites.**

### 3.2 Floor Tiles

| Asset                               | Count | Notes                                          |
| ----------------------------------- | ----- | ---------------------------------------------- |
| Cobblestone floor                   | 1     | Primary                                        |
| Worn flagstone floor                | 1     | Slightly damaged variant                       |
| Inked marker trail                  | 1     | Arrow / path markers (for tutorial rooms)      |
| Cracked floor                       | 1     | For high-danger rooms                          |
| Bloodstained floor                  | 1     | For combat memory / boss arena approach        |

**Floor tiles subtotal: 5 sprites.**

### 3.3 Special Rooms

**Shop Shrine Room:**
- Altar sprite (center): 1 asset
- Glowing book on altar (loop, 4 frames): 4 frames
- Crystal backdrop: 1 asset
- Candle decorations: 2 variants
- Subtotal: ~8 sprites.

**Start Room:**
- Rope ladder descending: 1 asset
- Entrance arch: 1 asset
- Subtotal: 2 sprites.

**Boss Arena (Dracular's Room):**
- Dracular's throne (Phase 1 and Phase 2 cracked variants): 2 sprites
- Scattered dice (decorative, 6-8 individual sprites): 8 sprites
- Ink pools (4 variants, mood-setting): 4 sprites
- Candelabras: 2 sprites (lit variants with VFX)
- Blood-stain decals: 3 variants
- Shattered throne (Phase 3): 1 sprite
- Subtotal: ~20 sprites.

### 3.4 Ambient Lighting

- Torch-glow overlay (radial, 128x128): 1 asset (used globally with tinting)
- Map overlay fade texture: 1 asset (full-screen parchment transition)
- Room entry vignette: 1 asset

**Lighting subtotal: 3 sprites.**

**Environment grand total: ~85 sprites** (exceeds the ~40 estimate in the budget — see Section 7 for revised numbers).

---

## 4. UI Asset List

### 4.1 HUD Elements

The HUD is always visible during gameplay. It must look like **notebook margins** — sketched indicators scattered at the edge of the parchment.

| Asset                               | Count | Notes                                                         |
| ----------------------------------- | ----- | ------------------------------------------------------------- |
| Heart icon — empty                  | 1     | Hollow ink heart, wobbly outline                              |
| Heart icon — filled                 | 1     | Cross-hatched red fill                                        |
| Heart icon — half                   | 1     | Left-half filled for 0.5 HP granularity if needed             |
| 17 hearts rendered inline           | —     | These are just the 3 base sprites repeated in the HUD layout  |
| Dice Tray background (parchment)    | 1     | Small parchment strip, torn edges                             |
| Attack die face (red) × 6           | 6     | 1-6 pip faces                                                 |
| Defense die face (blue) × 6         | 6     | 1-6 pip faces                                                 |
| Treasure icon (gold coin)           | 1     | Small coin, inked                                             |
| Potion icon (blue flask)            | 1     | Blue flask sketch                                             |
| Item icon — Buckler                 | 1     |                                                               |
| Item icon — Shield                  | 1     |                                                               |
| Item icon — Big Sword               | 1     |                                                               |
| Item icon — Big Axe                 | 1     |                                                               |
| Item icon — Spiky Armour            | 1     |                                                               |
| Item icon — Magical Armour          | 1     |                                                               |
| Reachability warning (pulsing)      | 4     | 4-frame pulse loop                                            |
| Map toggle hint                     | 1     | "M" keycap sketch                                             |
| Row indicator label                 | 1     | Handwritten label sprite                                      |
| Current-cell indicator (green)      | 2     | Normal + pulsing variant                                      |
| Visited cell overlay (grey)         | 1     | Hatching overlay                                              |

**HUD subtotal: ~33 UI sprites.**

### 4.2 Menu Screens

| Asset                               | Count | Notes                                                        |
| ----------------------------------- | ----- | ------------------------------------------------------------ |
| Main menu background (full scene)   | 1     | Parchment scene with ink illustrations of dungeon + dice     |
| Title logo "Solo Dungeon Dash"      | 1     | Hand-lettered ink brush, centerpiece                         |
| Button — normal state               | 1     | Sketched rectangle with handwritten label                    |
| Button — hover state                | 1     | Darker ink + small doodle accent                             |
| Button — pressed state              | 1     | Filled ink version                                           |
| Button — disabled state             | 1     | Faded, grey outline                                          |
| Settings icon — volume              | 1     |                                                               |
| Settings icon — text size           | 1     |                                                               |
| Settings icon — accessibility       | 1     |                                                               |
| Credits scroll background           | 1     | Long parchment scroll                                        |
| Pause menu overlay                  | 1     | Semi-transparent parchment veil                              |
| Game Over screen                    | 1     | Ink-splat dramatic artwork                                   |
| Victory screen                      | 1     | Dracular's ring + celebratory ink burst                      |

**Menu screen subtotal: ~13 sprites.**

### 4.3 In-Game Prompts

| Asset                               | Count | Notes                                                        |
| ----------------------------------- | ----- | ------------------------------------------------------------ |
| Tooltip bubble (small)              | 1     | Sketched speech bubble                                       |
| Tooltip bubble (large)              | 1     | For rule reminders                                           |
| Confirmation dialog (boss entry)    | 1     | Dramatic parchment frame                                     |
| Tutorial callout — floor marker     | 3     | Painted-on-floor arrows, 3 variants                          |
| Tutorial callout — speech balloon   | 1     | Tutor character speech                                       |
| Shop radial menu background         | 1     | Circular parchment wheel                                     |
| Shop radial menu slot (x6)          | 1     | Reusable slot sprite                                         |
| Map overlay frame                   | 1     | Parchment frame for the full dungeon map                     |
| Map legend                          | 1     | Hand-drawn legend with sample icons                          |
| Damage number pop-up                | 3     | 3 size variants for small/medium/crit                        |

**In-game prompt subtotal: ~14 sprites.**

**UI grand total: ~60 sprites.**

---

## 5. Audio Asset List

No voice acting. Music is atmospheric and minimal. SFX must feel **analog and warm**, not synthetic.

### 5.1 Music (3 Tracks)

| Track               | Duration  | Description                                                                 |
| ------------------- | --------- | --------------------------------------------------------------------------- |
| "Parchment Calm"    | 60s loop  | Main menu. Ambient, 2-3 instruments (lute, hurdy-gurdy, soft hand drums). Distant dice clatter as incidental texture. Feels like sitting at a dungeon master's table before the session starts. |
| "Dust of the Ink"   | 90s loop  | In-dungeon. Ambient base with occasional percussion swells. Crossfades between rooms. Medium intensity — should not fatigue over 30+ minute runs. |
| "Nine Dice Duel"    | 90s loop  | Boss (Dracular). Intense orchestral-ish arrangement. Dice-rolling percussion as signature motif. Builds in intensity across Dracular's 3 phases — ideally a layered track where stems can be added per phase. |

**Music format:** Ogg Vorbis, 192kbps, stereo. Loop points embedded in file metadata.

### 5.2 Sound Effects

#### Movement (~6 sounds)
- Footstep variants × 4 (stone, wood, metal grate, puddle)
- Dodge-roll swish
- Door-open creak
- Door-seal clank

#### Combat (~8 sounds)
- Parry-clang (metallic ring)
- Parry-miss (wooden thud)
- Light-attack whoosh
- Hit-thud (flesh impact)
- Crit-impact (meatier variant)
- Death-ink-splash (wet splatter)
- Dodge-woosh
- Heal-glug (potion consumption)

#### Dice (~4 sounds)
- Dice-roll clatter (3-5 dice tumbling on wood)
- Dice-land (final stop)
- Dice-six pulse (signature "good roll" reward chime)
- Dice-tray upgrade chunk (when adding a new die to tray)

#### Monster Signatures (~11 sounds)
One signature sound per monster + Dracular:
- Orc: guttural grunt
- Wolf: bark + snarl
- Skeleton: bone clatter
- Evil Warrior: shield clang
- Devil Bat: screech
- Cyclops: deep roar + ground impact
- Dark Elf: whispered taunt
- Skeleton Lord: echoing laugh
- Wizard: incantation hum
- Demon: roar-scream
- Dracular: signature laugh (3 variants, one per phase)

#### Environment (~4 sounds)
- Dungeon ambient hum (loopable)
- Torch crackle (loopable)
- Water drip (periodic)
- Echo tail (reverb accent)

#### UI (~6 sounds)
- Button click (soft ink-stamp)
- Menu open (page turn)
- Shop open (heavier page turn + small chime)
- Purchase confirm (coin drop)
- Toggle (pen click)
- Error (dull scrape)

#### Narrative (~4 sounds)
- Victory fanfare (short, triumphant)
- Defeat dirge (short, mournful lute)
- Tutorial chime (gentle bell)
- Achievement ding (warm confirmation)

**SFX total: ~43 one-shots.**

### 5.3 Audio Format Spec

| Asset type  | Format                | Channels  | Notes                            |
| ----------- | --------------------- | --------- | -------------------------------- |
| Music       | Ogg Vorbis, 192kbps   | Stereo    | With loop metadata               |
| SFX         | WAV 16-bit 44.1kHz    | Mono      | Normalized to -12 LUFS           |
| Ambient loops | Ogg Vorbis, 128kbps | Stereo    | For long-running loops (torch)   |

**Voice acting: NONE.** Zero budget for voice. Tutorial text is all on-screen parchment.

---

## 6. Procedural Asset Requirements

Solo Dungeon Dash is **lightly procedural** — the procedural layer is the **room arrangement**, not the art.

### 6.1 What is procedural

- **Dungeon layout:** The generator produces a 9×10 grid of cells with pre-seeded contents (monsters, loot, shop, boss). Each cell selects from the room tileset variants at random.
- **Monster positions within rooms:** Random offset within safe bounds so the player doesn't see the exact same monster layout twice.
- **Room tileset variation:** Each room instance randomly picks floor tile variant, wall decoration variants, and ambient lighting intensity.
- **Doorway seal animations:** Trigger based on combat state, not baked into room sprite.

### 6.2 What is NOT procedural

- **Dice faces** — hand-drawn, one per pip value per color (12 total).
- **Monster sprites** — all hand-drawn, no procedural variation.
- **Dracular's boss arena** — fully hand-designed, no randomization.
- **UI** — fully hand-drawn.
- **VFX** — frame-animated sprites, not particle systems.
- **Traps** — **none in v1.** No trap art budgeted.
- **Loot placement** — deterministic per seed (part of the run seed system).

### 6.3 Procedural implications for the artist

The artist needs to provide **variants and modular pieces**, not full scenes. Specifically:
- Give 3+ variants of every floor tile.
- Give 2-3 variants of every wall decoration.
- Design the octagonal room so any combination of doorway states reads visually clean.
- Provide a "room decoration rarity table" — some decorations (big bone piles, rare torches) should show up 1-in-5 rooms, not every room.

---

## 7. Asset Budget Summary

Revised totals after the detailed breakdown above.

| Category       | Subtotal                                                                            |
| -------------- | ----------------------------------------------------------------------------------- |
| Characters     | 34 player + 180 monster (+ 18 reserve) + 65 Dracular = **~297 sprite frames**       |
| VFX            | ~90 frames across ~15 distinct effects                                              |
| Environments   | ~47 room tileset + 5 floor + 8 shop + 2 start + 20 boss + 3 lighting = **~85**      |
| UI             | ~33 HUD + 13 menu + 14 prompt = **~60 assets**                                      |
| Audio          | 43 SFX + 3 music = **46 audio assets**                                              |

**Grand total: ~532 visual assets + 46 audio assets.**

> **Note:** This is slightly higher than the initial ~450 visual asset estimate because environments and VFX were under-scoped on first pass. The priority tier system (Section 8) keeps the actual P1 delivery under 200 assets.

---

## 8. Priority Tiers

Assets are gated by **when they are needed** in the development roadmap. The P0 → P3 split maps directly to the Prototype Prompts document (RT-D1).

### 8.1 P0 — Prototype (Prompt 1)

**Goal:** prove the core loop runs. Zero art required.

- **All assets:** placeholder rectangles, circles, labeled text sprites.
- **Color coding:** just use the palette colors to distinguish entity types (red rectangle = enemy, green circle = player, gold dot = treasure).
- **Audio:** bfxr-generated placeholder beeps, no music.

**Artist hours:** 0.
**Asset count:** ~0 final assets.

### 8.2 P1 — Alpha (Prompt 2 + Prompt 3 Vertical Slice)

**Goal:** a playable vertical slice that **looks like the final game** for the core loop, but only covers a subset of monsters and one room.

| Asset group                                      | Count      |
| ------------------------------------------------ | ---------- |
| Player sprites (full 7 animation sets)           | 34         |
| 3 monster sets (Orc, Wolf, Cyclops) at 18 each   | 54         |
| Dracular **placeholder only** (Phase 1 only)     | ~15        |
| 1 full octagonal room tileset                    | ~47        |
| Floor tiles (3 of 5 variants)                    | 3          |
| HUD essentials (hearts, dice tray, treasure, potion, 6 item icons) | ~25 |
| Dice faces (all 12)                              | 12         |
| VFX essentials (parry, dodge, hit, heal, death)  | ~20        |
| Tutorial floor markers                           | 3          |
| Main menu placeholder                            | ~4         |
| Audio: 10 most important SFX + 1 music track     | 11         |

**P1 total: ~228 assets.**
**Artist time estimate:** ~2-3 weeks part-time.

### 8.3 P2 — Beta

**Goal:** content completeness. All monsters, all rooms, full audio.

| Asset group                                      | Count      |
| ------------------------------------------------ | ---------- |
| Remaining 7 monster sets (Skeleton, Evil Warrior, Devil Bat, Dark Elf, Skeleton Lord, Wizard, Demon) | ~126 |
| Remaining floor variants                         | 2          |
| Wall decoration full set                         | ~12        |
| Shop Shrine room assets                          | ~8         |
| Start room assets                                | 2          |
| All 10 item icons + shop radial menu art         | ~10        |
| Full HUD polish (reachability pulse, map overlay, row indicators) | ~10 |
| Complete VFX library (monster-specific hit sparks, Cyclops radial, Demon scythe, Wizard projectile, door seal) | ~70 |
| Full audio pass: remaining 33 SFX + 2 music tracks | 35       |
| In-game prompt assets (tooltips, callouts, dialogs) | ~14     |
| Menu screens (full set: settings, credits, pause, game over, victory) | ~10 |

**P2 total: ~299 assets.**
**Artist time estimate:** ~4-5 weeks part-time.
**Audio designer time:** ~1 week (contracted).

### 8.4 P3 — Launch Polish

**Goal:** final pass on Dracular, weakest assets, and launch delighters.

| Asset group                                      | Count      |
| ------------------------------------------------ | ---------- |
| Dracular full 3-phase art (Phases 2 and 3)       | ~50        |
| Dracular signature VFX (dice swirl, ink tornado) | ~16        |
| Boss arena complete environment                  | ~20        |
| Animated main menu background                    | ~10        |
| Achievement unlock icons                          | ~10        |
| Polish on weakest P1/P2 assets                   | ~10        |
| Victory screen final art                          | ~2         |

**P3 total: ~118 assets.**
**Artist time estimate:** ~2 weeks.

### 8.5 Rolling Totals

| Tier  | Cumulative asset count | Cumulative artist weeks |
| ----- | ----------------------- | ----------------------- |
| P0    | 0                       | 0                       |
| P1    | ~228                    | ~2-3                    |
| P2    | ~527                    | ~6-8                    |
| P3    | ~645                    | ~8-10                   |

> **Note:** P3 cumulative slightly exceeds the ~532 grand total in Section 7 because P3 includes polish passes on existing assets. The count of **distinct** assets is still ~532; the ~645 reflects ~113 assets that get a second polish pass.

---

## 9. Asset Production Pipeline

### 9.1 Tools

| Stage          | Tool                                                     | Notes                                               |
| -------------- | -------------------------------------------------------- | --------------------------------------------------- |
| Sprite art     | **Aseprite** (primary) OR Procreate + export             | Aseprite for anyone working on desktop; Procreate for iPad workflow if artist prefers |
| Animation      | Aseprite timeline / Procreate animation assist           | Export as individual frames + JSON metadata         |
| UI layout      | Aseprite for sprites; Godot scene for layout            | Artist delivers sprites; programmer composes in Godot |
| Audio SFX      | **Bfxr** for prototype SFX, **Audacity** for cleanup     | Both free                                           |
| Music          | **LMMS** (free) or **Reaper** (paid, cheap) — or commission externally | See Section 11                     |
| Import         | Godot 4 direct import via Aseprite importer plugin       | Plugin: https://github.com/viniciusgerevini/godot-aseprite-wizard |
| Version control| **Git LFS** for binary assets                            | Required — do not commit raw PNGs to regular git    |

### 9.2 File Format Standards

| Asset type              | Format              | Notes                                                          |
| ----------------------- | ------------------- | -------------------------------------------------------------- |
| Sprite sheets           | PNG, transparent    | 2x scaled source; individual frames exported per animation     |
| Sprite metadata         | JSON (Aseprite)     | Frame tags, timings, loop points                               |
| Textures (backgrounds)  | PNG, opaque         | Max 2048×2048                                                  |
| Music                   | Ogg Vorbis 192kbps  | Stereo, with loop metadata                                     |
| SFX                     | WAV 16-bit 44.1kHz  | Mono, -12 LUFS normalized                                      |
| Ambient loops           | Ogg Vorbis 128kbps  | Stereo                                                         |
| Fonts                   | TTF / OTF           | From Google Fonts only                                         |

### 9.3 Naming Convention

All assets use **snake_case** and a strict prefix system:

```
player_idle_01.png
player_idle_02.png
...
monster_orc_attack_03.png
vfx_parry_spark_02.png
ui_heart_filled.png
env_room_octagon_base.png
env_door_n_open.png
audio_sfx_combat_parry_clang.wav
audio_music_parchment_calm.ogg
```

Prefixes: `player_`, `monster_<name>_`, `vfx_`, `ui_`, `env_`, `audio_sfx_`, `audio_music_`.

### 9.4 Folder Structure

```
project/
├── assets/
│   ├── sprites/
│   │   ├── player/
│   │   ├── monsters/
│   │   │   ├── orc/
│   │   │   ├── wolf/
│   │   │   └── ...
│   │   ├── dracular/
│   │   ├── environment/
│   │   │   ├── rooms/
│   │   │   ├── doors/
│   │   │   └── decorations/
│   │   └── vfx/
│   ├── ui/
│   │   ├── hud/
│   │   ├── menus/
│   │   └── prompts/
│   ├── audio/
│   │   ├── music/
│   │   └── sfx/
│   │       ├── movement/
│   │       ├── combat/
│   │       ├── dice/
│   │       ├── monsters/
│   │       ├── environment/
│   │       ├── ui/
│   │       └── narrative/
│   └── fonts/
```

### 9.5 Import Workflow

1. Artist creates asset in Aseprite / Procreate.
2. Artist exports to `assets/<category>/<subcategory>/` using the naming convention.
3. Artist commits via Git LFS.
4. Programmer runs Godot Aseprite importer (or manually imports PNG + JSON).
5. Godot auto-generates AnimatedSprite2D scenes for multi-frame animations.
6. Programmer places asset in the game scene.
7. Playtest in context.
8. If style issue: flag via PR comment; artist revises.

### 9.6 Placeholder Discipline

**Every asset must have a working placeholder from day one.** The programmer is responsible for committing placeholder sprites (colored rectangles, text labels) for every asset in the budget before the artist starts. This ensures:
- The game is always playable even if an asset is missing.
- Replacing a placeholder with a final asset is a one-file swap, not a code change.
- P0 (prototype) is shippable for internal testing without any art.

---

## 10. Quality Gates

Every asset must pass these gates before being merged:

### 10.1 Visual Gates

- [ ] Matches the style guide (ink lines, crosshatch, parchment palette).
- [ ] Uses only colors from the approved palette.
- [ ] Has visible line wobble (not perfectly straight or Bezier-smooth).
- [ ] Has appropriate hatching density for its lighting role.
- [ ] Is exported at 2x source resolution.
- [ ] Has a transparent background (where applicable).
- [ ] Animates at 60fps-ready — not baked to 30fps. (Frame timings in Aseprite metadata should be expressible at 16.67ms intervals.)
- [ ] Is named per convention and placed in the correct folder.

### 10.2 Audio Gates

- [ ] SFX is under 1 second (except intentional ambient loops).
- [ ] Music loops seamlessly (no audible click at loop point).
- [ ] All audio normalized to -12 LUFS.
- [ ] Mono for SFX, stereo for music.
- [ ] No clipping, no plosives, no background hiss.

### 10.3 Review Cadence

- **Daily style review** during P1 and P2 production: programmer and artist do a 10-minute style check together. Any asset drifting from the style guide is flagged and revised same-day.
- **Weekly in-context playtest:** programmer builds the game with all current-week assets and the artist playtests to see their work in motion.
- **Pre-merge peer review:** artist reviews their own asset once at end of day; programmer reviews at merge time.

### 10.4 Reject Criteria

An asset is rejected if:
- It has colors outside the palette.
- It has smooth vector curves (no wobble).
- It has digital gradients or soft shadows.
- It has audio clipping or loop clicks.
- It does not match the filename convention.
- It is baked to 30fps and cannot be retimed.
- It is missing its animation metadata JSON.

---

## 11. Outsourcing Guidance

The team is small. Outsourcing is allowed and expected for some categories.

### 11.1 Art — In-house preferred

- **In-house:** all player, all monsters, all Dracular phases, all UI. These are the style-critical assets and need tight feedback loops.
- **Outsourceable:** environment decorations (pillars, torches, cracks, bones), floor tiles, tooltips, button states. These are lower-risk repetitive assets.
- **Outsourcing venues:** Fiverr ($50-200/asset set), Upwork ($500-2000/asset set), or a trusted freelance illustrator from the artist's own network.
- **Rule:** any outsourced asset must be approved by the in-house artist before acceptance to prevent style drift.

### 11.2 Music — Commission externally

- 3 tracks is a small-enough commission that hiring an indie composer makes sense.
- Budget target: **$300-800 per track** (total $900-2400).
- Recruit from: GameJolt composer directories, Reddit r/gameaudio, or Twitter/Bluesky game audio community.
- Deliverables: final master in Ogg Vorbis with loop metadata + unmixed stems (for Dracular fight layering).
- **Written commission agreement required** — see Legal section.

### 11.3 SFX — DIY or bundle

- **DIY option:** Bfxr for prototypes, Audacity for cleanup. Most of the 43 SFX can be created in-house in ~1 week.
- **Bundle option:** purchase a "dungeon crawler SFX bundle" from sites like Sonniss, Soundly, or itch.io for **$30-80 one-time**. This covers ~70% of the list; remainder is DIY.
- **Commission option:** a sound designer for ~$500-1500 for a full custom pass. Only if budget allows.

### 11.4 Voice — SKIP

No voice acting. Zero budget, zero complexity, zero localization overhead. The game is narratively light and parchment-text is sufficient.

### 11.5 Backup plans

- **Artist unavailability:** have a backup artist identified before P1 starts. Ideally someone who has seen the style guide and can match it within 1 week of ramp-up.
- **Audio designer unavailability:** DIY fallback with Bfxr is always viable — the game ships even if the contracted designer vanishes.
- **Composer unavailability:** CC-licensed ambient tracks from Free Music Archive or Pixabay Music as absolute-last-resort fallback.

---

## 12. Legal

### 12.1 Originality

- **All assets must be original or licensed for commercial use.**
- No "borrowed" references in final assets — mood boards are fine, tracing is not.
- All contracted artists must sign a work-for-hire / IP assignment agreement.

### 12.2 Fonts

- **Google Fonts only.** Caveat, Homemade Apple, Inter — all are SIL Open Font License or similar permissive licenses.
- No fonts from Dafont, free download sites with unclear licenses, or AI-generated fonts.
- Font licenses stored in `/docs/licenses/` in the repo.

### 12.3 SFX Bundles

- **Read the license carefully before purchase.** Some "royalty-free" bundles forbid use in commercial games.
- Prefer licenses that allow: commercial use, unlimited seats, redistribution as part of compiled game.
- Store license PDF in `/docs/licenses/`.
- Common acceptable sources: Sonniss, Pro Sound Effects, Soundly, itch.io bundles with explicit commercial terms.

### 12.4 Music Commissions

- **Written commission agreement required**, covering:
  - Work-for-hire / exclusive license
  - Platform usage rights (PC, Steam Deck, mobile)
  - Promotional / trailer use
  - Credit terms (composer listed in credits scroll)
  - Payment schedule and deliverable list
  - Revision terms (typically 2 rounds included)
- Template agreement stored in `/docs/contracts/`.

### 12.5 AI-Generated Content

- **NO AI-generated final art.** The hand-drawn aesthetic depends on real human irregularity; AI output lacks the right kind of wobble and the marketing story would be damaged.
- **Allowed:** AI tools for brainstorming mood boards, concept sketching, bfxr for procedural SFX (which is not AI).
- **Not allowed:** Stable Diffusion, Midjourney, Dall-E, or any generative model for shipped assets.

### 12.6 Attribution

- Credits scroll must list every contracted artist, composer, and sound designer by name.
- Open-source tools (Godot, Aseprite, Blender if used) should be acknowledged.
- Font licenses linked in About menu.

---

## 13. Risks

### 13.1 Production Risks

| Risk                                              | Likelihood | Impact | Mitigation                                                        |
| ------------------------------------------------- | ---------- | ------ | ----------------------------------------------------------------- |
| Artist unavailability mid-project                 | Medium     | High   | Identify backup artist before P1 starts; document style guide thoroughly |
| Style drift across animations                     | High       | Medium | Daily style reviews during hot production; shared reference sheet |
| Audio designer missing deadline                   | Medium     | Medium | DIY Bfxr fallback is always viable                                |
| Scope creep (more monsters, more rooms)           | High       | High   | Priority tiers are hard gates — do not start P2 art before P1 ships |
| Asset pipeline bottleneck (programmer waits on art) | Medium   | High   | Placeholder discipline — every asset has a working placeholder from day one |

### 13.2 Technical Risks

| Risk                                              | Likelihood | Impact | Mitigation                                                        |
| ------------------------------------------------- | ---------- | ------ | ----------------------------------------------------------------- |
| Audio latency on mobile                           | Medium     | Medium | Use Godot's low-latency audio bus; test on actual device early    |
| Sprite sheet import failures                      | Low        | Medium | Aseprite importer plugin is battle-tested; keep fallback manual import workflow |
| Animation frame timing inconsistencies            | Medium     | Low    | Enforce 60fps-ready gate; all timings in Aseprite metadata        |
| Git LFS quota / bandwidth limits                  | Low        | Medium | GitHub LFS has a quota; monitor; move to self-hosted if needed    |
| Godot 4 project size blow-up from sprite frames   | Medium     | Medium | Use texture atlases where possible; target <500MB final build     |

### 13.3 Style Risks

| Risk                                              | Likelihood | Impact | Mitigation                                                        |
| ------------------------------------------------- | ---------- | ------ | ----------------------------------------------------------------- |
| Artist over-polishes and loses wobble             | High       | High   | Inconsistency budget in Section 1.8 is explicit; review daily     |
| UI readability suffers from hand-drawn aesthetic  | Medium     | High   | Use Inter for numbers; test with reduced vision settings in audit |
| Monsters aren't visually distinct at small size   | Medium     | High   | Silhouette test — each monster must be readable in black only     |
| Color palette feels too restrictive               | Low        | Medium | Palette is deliberately small; pushback means "use hatching, not color" |

### 13.4 Schedule Risks

| Risk                                              | Likelihood | Impact | Mitigation                                                        |
| ------------------------------------------------- | ---------- | ------ | ----------------------------------------------------------------- |
| P1 art slips past 3 weeks                         | High       | High   | Cut scope — drop reserve monsters, reduce decoration variants     |
| P2 audio slips                                    | Medium     | Medium | Use DIY Bfxr + purchased bundle as fallback                       |
| P3 Dracular art runs out of time                  | Medium     | High   | Phase 1 must be complete before P3 begins; Phases 2-3 are the polish buffer |

---

## 14. Open Questions

1. **Aseprite vs Procreate** — which is the artist's preferred tool? This decides the export workflow.
2. **Mobile texture memory** — can we fit ~500 sprite assets in mobile GPU memory comfortably, or do we need to aggressively atlas? (Feeds into RT-T1 technical architecture.)
3. **Localization** — will hand-lettered text need to be re-drawn per language, or do we use the Caveat font for all localizable strings? Recommendation: font-based localization for all strings, hand-lettered only for the title logo.
4. **Accessibility high-contrast mode** — do we need an alternate palette for accessibility? (Feeds into RT-A5 accessibility pass.)
5. **Modding / workshop support** — if we want Steam Workshop modding post-launch, what asset hook points do we expose? (Out of scope for v1.)

---

## 15. Next Steps (post-RT-A4)

1. Share this document with the artist for style-guide signoff.
2. Artist creates a "style calibration sheet" — one player sprite + one monster (Orc) + one UI element (heart) — for approval before any bulk production.
3. Programmer creates placeholder assets for entire P1 list and commits to the repo.
4. Lock P1 scope with the team.
5. Begin P1 production; daily style reviews start immediately.
6. Feed asset list into RT-D1 (Prototype Prompts) so prompts reference correct asset names and counts.

---

**End of document.**
