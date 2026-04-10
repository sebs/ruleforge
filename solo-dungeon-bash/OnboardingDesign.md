# Onboarding & Tutorial Design — Solo Dungeon Bash

## Design Philosophy
Solo Dungeon Bash has a shallow rule surface but several interlocking subsystems. The tutorial must teach the *order* of the turn sequence as much as each individual rule. We favour **"learn by doing" on a scripted micro-run** over a screen-wall of tooltips.

## Complexity Rank (priority of teaching)
| Priority | Concept | Why it matters | Where to teach |
|---|---|---|---|
| 1 | Moving one square at a time | First action the player takes. Blocks nothing else. | Turn 1 of scripted micro-run |
| 2 | Revealing room contents with a die | Second action, same turn. | Turn 1, after move |
| 3 | Treasure & Potion counters | Passive resource gain, trivial. | Turn 2 (seeded Treasure) |
| 4 | Combat structure (attack-defence-attack-defence) | The most complex rule. Break into 4 micro-steps. | Turn 3 (seeded weak Orc) |
| 5 | HP cap and healing | Reassurance — "you can heal". | Turn 4 (seeded Potion) |
| 6 | Shop and permanent upgrades | Long-term progression. | Turn 5 (enough Treasure to buy) |
| 7 | Item slot exclusivity | Prevents early confusion. | First time player tries to buy a second weapon |
| 8 | No-revisit and king-adjacency | Taught implicitly — invalid squares just don't highlight. | Throughout |
| 9 | Path-blocking loss | Only when dangerous. | Contextual warning when BFS detects imminent block |
| 10 | End square & Dracular | Climactic reveal. | Turn N when reaching End |

## Progressive Disclosure Plan
1. **First launch → "Learn" or "Play"** — dialog with two big buttons. Default to "Learn" for new players.
2. **Scripted micro-run on a reduced grid (e.g. 5×5)** — seeded dice, guaranteed encounters, no random grief.
3. **Full game unlocks on completing micro-run OR on player skipping tutorial.**
4. **Contextual first-time tooltips** reappear in the real game for rare events (first time facing a boss, first time blocked warning).

## Step-by-Step Tutorial Script

### Screen 0 — Welcome
**Narrator banner** (ink-sketch parchment style):
> Welcome, adventurer. You stand at the edge of the Dungeon. Ten levels of monsters and treasure lie above. At the top: Dracular, the Big Bad Boss. Kill him, and you win. Let's learn how.

**Button:** "Let's go."

### Screen 1 — The Grid
Highlight the 5×5 tutorial grid. Animate a pen drawing the grid on-screen.
> The dungeon is a grid. Each cell is a room. You start here (pulse Start square). Your goal is to reach the End (pulse End square at the top).

**Tooltip anchor:** on Start square.

### Screen 2 — First Move
Highlight three legal adjacent squares above Start with a soft glow.
> Every turn, you move one square. You can step up, diagonally, or sideways — any of the squares next to you. Try clicking the square directly above Start.

**Player action:** clicks highlighted square.
**System action:** marks cell as visited, draws ink line from Start.

### Screen 3 — Room Contents
Scripted: this room is Empty.
> Every time you enter a new room, you roll a die to see what's inside. Let's roll.

**Animation:** d6 rolls. Lands on 5. "Empty."
> An empty room. Nothing here — just dust. On to the next turn.

### Screen 4 — Treasure
Player clicks the next highlighted square. Scripted Treasure result.
**Animation:** d6 rolls. Lands on 6.
> **Treasure!** That's +1 Treasure to your pile. You'll spend Treasure later to buy weapons and armour.

Counter animates from 0 → 1.

### Screen 5 — First Combat
Player clicks next square. Scripted monster: Orc (1 AD).
> **An Orc!** Monsters block your path — you have to fight. Don't worry, this is a weak one.

Overlay: combat UI appears. Slow-motion mode engaged.

> **Step 1:** The monster attacks first. The Orc rolls 1 die. Each 6 is a hit.

d6 rolls. Scripted: rolls 2 (no hit).
> No hit. You're safe.

> **Step 2:** Now you defend. Each 6 on your defence dice cancels a hit. Since there's nothing to block, we skip this. (In a real fight, you'd roll now.)

> **Step 3:** Your turn. You roll your attack dice. Each 6 hits the monster.

d6 rolls. Scripted: rolls 6.
> **Hit!** The Orc had 1 HP — any unblocked hit kills it. You won the fight.

Animate Orc fading away with an ink-smudge effect.

### Screen 6 — After Combat
> After combat, you can drink potions to heal, and then buy upgrades. You don't have any potions or treasure yet, so let's keep moving.

### Screen 7 — Potions
Player clicks next square. Scripted: Potion.
> **A Potion!** +1 Potion. You can drink potions between turns — each one heals 1 HP, up to your starting maximum of 17.

### Screen 8 — A Harder Fight
Scripted: next square is a Wolf (2 AD). Player takes 1 HP of damage.
> A Wolf. Slightly tougher — 2 attack dice. Watch what happens.

d6 rolls for monster: 6, 3. → 1 hit.
> 1 hit incoming.

d6 rolls for player defence: 1. → 0 blocks.
> You couldn't block. **−1 HP.** You're now at 16.

Player attacks, kills wolf.
> Your turn. *Rolls 6.* Wolf dead. Good work.

### Screen 9 — Using a Potion
> You have 1 Potion and you lost 1 HP. Want to heal? Click the Potion.

**Player action:** clicks Potion button. HP 16 → 17, Potion 1 → 0.
> Full HP again. Nice.

### Screen 10 — The Shop
Player has 1 Treasure from Screen 4.
> **Time to shop.** Tap a shop item to buy it. With 1 Treasure you can buy a Buckler (+1 Defence Die) or a single Potion.

Modal displays all shop items with locked/greyed state for unaffordable items.

**Player action:** buys Buckler.
> Nice! You now have **2 Defence Dice** permanently.

### Screen 11 — Item Slots
Now try to buy the Shield (not enough Treasure, greyed out).
> The Shield would give you 2 Defence Dice, but it can't be used with the Buckler you just bought. When you see "—" on an item, it means it conflicts with one you already own. Weapons, shields, and armour each have **one slot**. Pick wisely!

### Screen 12 — Path-Blocking Warning
Script a scenario where the player is about to move into a dead-end-forming square.
> ⚠ **Careful!** If you move here, you won't be able to reach the End square. You'd lose. Pick a different square.

Highlight safer options.

### Screen 13 — The End Square
Fast-forward to End square approach.
> The **End square** is above Level 10. Dracular lives there. Dracular has 9 attack dice **and** 9 defence dice — he's much tougher than any monster you've fought. Go when you're ready.

**Confirmation dialog:** "Enter End square? You cannot turn back. [Cancel] [Enter]"

### Screen 14 — Tutorial End
Victory or defeat in a scripted boss fight. Either way:
> You've learned the basics! Real runs are harder — the grid is bigger, the dice are real, and Dracular won't be as easy. Good luck.

**Buttons:** "Start a real run" / "Replay tutorial"

## Tooltip & Hint Library
These show once per new player on first encounter.

| ID | Trigger | UI Copy |
|---|---|---|
| T-01 | Hovering a Level-5+ monster for the first time | "Deeper levels roll tougher monsters. Check your dice before committing." |
| T-02 | First time reaching ≤ 5 HP | "You're in trouble. Consider detouring to L1/L3/L6/L9 for potions." |
| T-03 | First time spending Treasure | "Spending now means permanent power for the rest of the run." |
| T-04 | First time you could have bought a potion and didn't | "You can also spend Treasure on Potions — 3 for 2T is the best value." |
| T-05 | First time entering a row you've never been to | "New level — new table. Tap the row to see what might be here." |
| T-06 | First time the path reaches End | "You can enter the End square now. Or keep farming for more loot." |
| T-07 | First mid-combat hit landed | "A 6 is always a hit. Defence 6s cancel incoming hits one-for-one." |
| T-08 | First time at full Potion inventory (5+) | "Potions don't expire. Save them for the boss fight." |

## Practice Scenario (Skippable)
A free-play sandbox mode where the player can:
- Replay the tutorial micro-run.
- Choose specific rooms manually (for practicing combat).
- Preview any monster's combat against their current gear.

## Content-Free Onboarding Layer
For experienced players who skip the tutorial, provide a **single help screen** showing:
1. The 7-step turn order.
2. The 11 monsters and their AD.
3. The item shop table.
4. The win/lose conditions.

This is a single scrollable page, no interaction required.

## Tutorial Success Metrics
- % of tutorial completers who then start a real run (target: ≥ 75%)
- % of first-run attempts that last ≥ 10 turns (target: ≥ 60%)
- Time to first successful combat in real run (target: < 3 turns)
- Skip-rate for returning players (target: 100% — skip should be obvious)
