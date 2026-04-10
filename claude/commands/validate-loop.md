# Core Loop Validation (F05)

You are a game loop quality analyst. Your task is to validate an extracted game loop for structural correctness and completeness.

## Input

`$ARGUMENTS` may reference a game loop file. If not provided, look for a game context file (`output/*/.context.json`) to identify the active game directory, then check for `output/{game-slug}/GameLoop.md` or `output/{game-slug}/GameLoop.mmd`. If multiple game directories exist, ask the user which game to use.

## Validation Checks

Perform each of these checks and report pass/fail with details:

### 1. Completeness Check
- [ ] Every phase has a defined entry condition (how do you get here?)
- [ ] Every phase has a defined exit condition (how do you leave?)
- [ ] The loop can cycle from game start to game end without gaps
- [ ] All player actions mentioned in the rules appear somewhere in the loop
- [ ] Setup phase is defined
- [ ] End-game trigger is defined
- [ ] Final scoring/resolution is defined

### 2. Dead-End Detection
- [ ] No phase exists from which there is no valid transition out
- [ ] No circular dependency that prevents progress (infinite sub-loop with no exit)
- [ ] All conditional branches have at least one reachable path

### 2b. State Reachability Check
- [ ] All game states referenced in the rules can be reached from the initial setup state
- [ ] No "orphan" phases exist that are defined but unreachable via any transition
- [ ] The end-game state is reachable from every possible mid-game state

### 3. Ambiguity Check
- [ ] All transition conditions are clearly defined (no "then proceed" without specifying where)
- [ ] Turn order is unambiguous (who goes next?)
- [ ] Phase order within a turn is unambiguous
- [ ] Simultaneous action resolution is defined (if applicable)

### 4. Player Agency Check
- [ ] At least one decision point exists per primary loop iteration
- [ ] Players are not forced into a single path with no meaningful choice
- [ ] Information available at decision points is defined

### 5. Pacing Check
- [ ] The loop structure suggests escalation or progression over time
- [ ] There is variation in the loop (not every turn is identical)
- [ ] The end-game trigger prevents indefinite play

## Output Format

For each check, report:
- **Status:** PASS / FAIL / WARNING
- **Detail:** What was found
- **Recommendation:** How to fix (if FAIL or WARNING)

Provide a summary:
- Total checks: X
- Passed: X
- Warnings: X
- Failed: X
- **Loop Validity: VALID / VALID WITH WARNINGS / INVALID**

Write to `output/{game-slug}/LoopValidation.md`. Display the summary and any FAIL/WARNING items to the user.
