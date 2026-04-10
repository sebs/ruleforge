# Ambiguous Rule Flagging — Solo Dungeon Bash

This file collects every passage of the rulebook where the text admits more than one reasonable interpretation, references something that isn't defined, or leaves a mechanical gap. For each flag we quote the source, explain the ambiguity, propose a default digital ruling, and note a "house rule" alternative where relevant.

---

## A-1 — What actually happens when you block yourself
**Type:** Incomplete rule (loss-condition procedure)

> *"Be careful not to block yourself from getting into End square, because if you do you've lost!"*

### Ambiguity
The rulebook treats this as a warning, not a procedure. It does not tell you *when* to check whether you are blocked: is it every turn (BFS from current square to End across unvisited squares), only when you have zero legal moves at all, or only when the player notices? A literal strict reading means the moment the End square becomes unreachable you lose — but nothing instructs the player to check.

### Interpretations
- **(a) Strict / Reachability check each turn:** After every move, run a reachability test; if End is no longer reachable from current square via only unvisited squares (king-adjacency), the run ends immediately.
- **(b) Lenient / Only when stuck:** You only lose when you actually have zero legal next-square options AND End hasn't been reached.
- **(c) Awareness-based:** You lose the moment *you* declare you're blocked (table-honour rule).

### Proposed digital default
Use **(a)**. Digital can trivially compute reachability on every move with a flood-fill/BFS. Showing an "unreachable" warning preserves the *spirit* of the rule (punish bad pathing) while being actionable. Optionally surface a toggle for (b) as an easier mode.

---

## A-2 — Can you drink Potions mid-combat?
**Type:** Incomplete rule (timing)

> *"Take any or all Potions ... At this stage you can take a breather, a chance to recover. You may use 1 or more Potions that you have collected."*

### Ambiguity
Potion use is listed as step 6 of the turn sequence, *after* combat resolves in step 5. But combat is an inner loop that may span multiple sub-rounds — nothing in the text says you cannot sip a Potion between combat rounds to survive.

### Interpretations
- **(a) Strict / "Only at step 6":** You may only heal after combat is fully resolved. If you die mid-combat, you die even if you have 50 Potions.
- **(b) Lenient / "Between combat rounds":** Because the turn sequence is literal and step 6 only happens post-combat, a strict reading clearly supports (a). But many players intuitively assume (b).

### Proposed digital default
Use **(a) strict**. The turn-sequence numbering is unambiguous about *where* Potions are used. Offering mid-combat potions would flatten difficulty significantly (large stockpiles would make the player effectively immortal). Consider (b) as an optional "Forgiving Mode."

---

## A-3 — Spiky Armour and the "armour slot"
**Type:** Ambiguous rule (slot semantics)

> *"5 — Spiky Armour: Give you an additional 2 Attack Dice an 1 additional Defence Die."*  
> *"6 — Magical Armour: Gives you an additional 5 Defence Dice. **Can not be used in conjunction with Spiky Armour**"*

### Ambiguity
Magical Armour explicitly excludes Spiky Armour. Spiky Armour has no exclusion listed, but the name implies it's armour — and the item right below it (Magical Armour) declares an exclusion. Does the player have an "armour slot" that allows exactly one of Spiky/Magical, or is Spiky genuinely stackable with any other item including Shields/Bucklers?

### Interpretations
- **(a) Literal / Text-only:** Exclusions only apply where written. Spiky Armour stacks with Shield, Buckler, Sword, Axe, and itself-isn't-excluded-with-anything-else. Only Magical Armour excludes Spiky.
- **(b) Slot-based:** There is an implicit armour slot. Spiky and Magical are mutually exclusive (already stated), and there's *also* likely a shield slot (Buckler XOR Shield, already stated) and a weapon slot (Sword XOR Axe, already stated).

### Proposed digital default
Use **(a) literal**. The rulebook is careful about listing exclusions on weapons and shields; it's reasonable to assume the author listed every exclusion they intended. Spiky Armour stacks with Shield/Buckler and with Sword/Axe, subject to those items' own exclusions.

Document this clearly in UI so players aren't surprised.

---

## A-4 — Can you own multiple copies of the same item?
**Type:** Incomplete rule

### Ambiguity
Nothing prevents the player from buying two Bucklers (2 Treasure → +2 Defence Dice), or two Big Swords (6 Treasure → +2 AD), which would be strictly better per-Treasure than stacking up to a Shield or Big Axe. The exclusivity clauses talk about *different* items ("can not be used in conjunction with X") but not about duplicates.

### Interpretations
- **(a) Literal / Duplicates allowed:** The player may buy duplicates. This trivially breaks the economy, because Big Sword is 3T for +1 AD, but Big Axe is 4T for +2 AD — and buying 2× Big Sword (6T for +2 AD, no axe needed) collapses the design.
- **(b) Spirit / Single copy:** Each item is owned at most once. Exclusivity is between *categories* but quantity is 1 per item.

### Proposed digital default
Use **(b)**. It matches the rulebook's clear intent to balance the item cost ladder. Each item is a permanent, one-copy unlock. In the UI, an item becomes greyed-out after purchase.

---

## A-5 — Does a Treasure or Potion room trigger a combat?
**Type:** Ambiguous rule (step ordering)

> *"3. If room contains Treasure add it to your Treasure score. 4. If room contains Potion add it to your Potion count. 5. If room contains a Monster, fight to the Death."*

### Ambiguity
Each level table's d6 result is a *single* type of room (Treasure **or** Potion **or** Monster **or** Empty). The turn sequence lists 3/4/5 as independent checks, which reads as if multiple could apply — but the tables only ever produce one outcome. This is not a contradiction, just structural ambiguity for someone reading the turn sequence in isolation.

### Proposed digital default
Interpret step 3/4/5 as a mutually-exclusive `switch` on room content. No ambiguity in digital — just model rooms as an enum.

---

## A-6 — The ambiguous spelling "Dracular!"
**Type:** Implicit knowledge / typographical

> *"End — 1 - 6 Dracular! 9 Attack Dice and 9 Defence Dice"*

### Ambiguity
"Dracular!" is clearly a typo of "Dracula!" (the famous vampire). This does not affect mechanics but does affect presentation. Should the digital version keep "Dracular!" as period-authentic charm, or correct it?

### Proposed digital default
**Keep "Dracular!"** as-is. It's a direct, recognizable quote from the source and the `!` implies intentional flair. Surface the original spelling in a footnote or tooltip.

---

## A-7 — Monster Defence Dice general case
**Type:** Implicit knowledge

> *"4. Monster rolls its Defence Dice, if it has any."*

### Ambiguity
The level tables list Attack Dice values for every monster except the boss. No monster in the normal rows has a listed Defence value. The phrase "if it has any" is future-proofing for Dracular (9 DD), not a signal that other monsters have undocumented DD.

### Proposed digital default
All normal monsters have **0 Defence Dice**. Dracular has **9 Defence Dice**. No ambiguity in implementation.

---

## A-8 — "Fight to the Death" and interruption
**Type:** Incomplete rule

> *"If room contains a Monster, fight to the Death."*

### Ambiguity
There is no "flee" or "retreat" option listed. The phrase "to the death" is unambiguous in intent, but players sometimes expect retreat from traditional dungeon crawlers.

### Proposed digital default
**No retreat.** You must fight every monster in its entirety to completion. (This is actually a notable design feature — it means high-level rooms carry real commitment.)

---

## A-9 — What if I have no Treasure to spend?
**Type:** Trivial / procedural

Not really ambiguous, but worth noting: step 7 is always in the turn sequence even when the player has zero Treasure. The digital implementation should auto-skip the shop (or show it greyed) when affording nothing.

---

## A-10 — Starting position movement
**Type:** Incomplete rule

### Ambiguity
The very first move: the player starts on the Start square, which is below row 1 in the middle column. Is the Start square itself rolled for contents? (Probably not — it's a special square.) Is the first move from Start to any of the 3 squares in row 1 column 4/5/6? (Yes by king-adjacency.) Does "adjacent" at Start include the column-adjacent squares or just the one directly above?

### Proposed digital default
Start and End rooms have **no content rolls**. From Start, the player's first legal moves are to the three squares in row 1 that are king-adjacent: (row 1, col 4), (row 1, col 5), (row 1, col 6). From the top of row 10, entering End requires being on (row 10, col 4/5/6). On reaching End, the boss fight is forced.

---

## Summary by Severity
| Flag | Severity | Digital Resolution |
|---|---|---|
| A-1 (blocking check) | Medium — affects loss-condition correctness | BFS reachability each turn |
| A-2 (potion timing) | Medium — affects difficulty curve | Strict: only at step 6 |
| A-3 (Spiky armour slot) | Medium — affects optimal loadout | Literal: stacks with shields/weapons |
| A-4 (duplicate items) | **High — breaks economy** | Single copy per item |
| A-5 (room content exclusivity) | Low — clear from tables | Enum |
| A-6 (Dracular spelling) | Cosmetic | Keep original |
| A-7 (monster DD) | Low | All normal monsters = 0 DD |
| A-8 (no flee) | Low — intentional | No flee |
| A-9 (empty shop) | Trivial | Auto-skip |
| A-10 (start/end square) | Low | Special squares, no roll |
