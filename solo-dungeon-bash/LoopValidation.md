# Core Loop Validation — Solo Dungeon Bash

## Overview
This report validates the game loop extracted in Stage 5 for structural correctness: every state has a defined exit, the loop is reachable and cyclable, and terminal states are properly annotated.

## 1. Exit Condition Check (per state)
| State | Exit condition(s) | Defined? |
|---|---|---|
| Setup | → Turn Start (always) | ✅ |
| Turn Start | → Pick Square (always) | ✅ |
| Pick Square | → Enter Room (legal move) OR → Loss-Blocked (no move / unreachable) OR → Boss Fight (chose End) | ✅ |
| Enter Room | → Roll Content (always) | ✅ |
| Roll Content | → Content Switch (always) | ✅ |
| Content Switch | → Empty → Recover, Treasure → +1T → Recover, Potion → +1P → Recover, Monster → Combat | ✅ |
| Combat: Monster Attack | → Player Defend | ✅ |
| Combat: Player Defend | → Take Damage | ✅ |
| Combat: Take Damage | → Check Dead | ✅ |
| Combat: Check Dead | → Loss-Dead (HP ≤ 0) OR → Player Attack | ✅ |
| Combat: Player Attack | → Monster Defend | ✅ |
| Combat: Monster Defend | → Check Monster Dead | ✅ |
| Combat: Check Monster Dead | → Monster KO (any hit lands) OR → Monster Attack (loop) | ✅ |
| Monster KO | → Win (boss) OR → Recover (normal) | ✅ |
| Recover (step 6) | → Shop (always, even with 0 potions) | ✅ |
| Shop (step 7) | → Turn End (always, even with 0 treasure) | ✅ |
| Turn End | → Turn Start | ✅ |
| Loss-Blocked, Loss-Dead, Win | Terminal — no further transitions | ✅ |

**Result:** All 18 defined states have at least one defined exit. **No dead-ends detected.** ✅

## 2. Reachability Check
Can every state be reached from Setup?
- ✅ Setup → Turn Start → Pick Square (reachable on turn 1)
- ✅ Enter Room → Roll Content → Content Switch (reachable on any non-blocked turn)
- ✅ Combat states (reachable whenever a Monster outcome rolls; Level 1 has 50% chance on turn 1)
- ✅ Recover & Shop (reachable every turn post-content)
- ✅ Win (reachable via Enter End → Combat → Monster KO branch)
- ✅ Loss-Dead (reachable via Check Dead branch when monster overwhelms)
- ✅ Loss-Blocked (reachable in principle — requires the player to paint themselves into a corner)

**Result:** All 18 states are reachable from Setup. ✅

## 3. Cycle Check
Does the loop cycle from Turn Start back to Turn Start?
- Yes: Turn Start → Pick Square → Enter Room → Roll → Resolve → Recover → Shop → Turn End → Turn Start. Cycle confirmed. ✅

Can the cycle run indefinitely? No — bounded by the no-revisit grid (≤92 moves max). This is a desirable bound. ✅

## 4. Termination Guarantee
Does every run terminate? **Yes.** The grid has exactly 92 squares; each turn consumes exactly one new square; once all squares are consumed the player has either entered End or hit a block. Therefore no infinite loop is possible. ✅

## 5. Ambiguous Triggers
| Trigger | Ambiguity | Tracked in |
|---|---|---|
| "Blocked" loss trigger | When is reachability checked? | A-1 |
| Potion use timing (mid-combat?) | Step 6 only? | A-2 |
| Starting position movement | Which squares are adjacent to Start? | A-10 |

All three are noted in `AmbiguousRules.md` with digital defaults. None prevent the loop from running.

## 6. Structural Issues Found
### Issue L-1 (Minor): "Blocked" is an implicit state, not an explicit transition
The rulebook frames self-blocking as a warning, but the loop treats it as a loss. In a faithful extraction we must make this explicit: each "Pick Square" step should begin with a reachability check. Digital default: BFS every turn.

### Issue L-2 (Minor): No explicit "zero legal moves ≠ End" case
If the player has exhausted all neighbouring squares but is still adjacent to unvisited squares via a longer path, they're not *stuck*, but the rulebook doesn't describe this precisely. The natural extension is: a player is only "stuck" if none of their king-adjacent squares is legal (unvisited). The distinction between *being* stuck (no next move) and *heading to* stuck (End is still reachable but getting there will stuck you) is subtle. We treat "no next move" as Loss-Blocked immediately.

### Issue L-3 (Minor): Boss entry is one-way and forced
Once the player chooses to step into End, they *must* fight Dracular. There's no way to abort. This is a design feature, not a bug — but it must be surfaced in the digital UI (confirmation dialog).

### Issue L-4 (None): Combat loop termination
Combat is guaranteed to terminate because every round either the monster dies (1 HP, so first hit wins) or the player takes damage (17 HP, bounded downwards). Dracular's 9 DD theoretically extends combat, but with sufficient player AD the expected rounds-to-kill is bounded. No infinite loops.

## 7. Validation Verdict
| Criterion | Status |
|---|---|
| Every phase has a defined exit condition | ✅ |
| No dead-end states | ✅ |
| Loop is complete (Setup → Win/Loss) | ✅ |
| Ambiguous triggers all have digital defaults | ✅ |
| All states reachable from initial state | ✅ |
| Termination guaranteed | ✅ |

**The game loop is structurally valid.** Minor clarifications (L-1 through L-3) are already addressed in the digital default interpretations. The loop is ready for implementation.
