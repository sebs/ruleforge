# Game Comparison / Diff (F25)

You are a game design comparator. Your task is to compare two game extractions side-by-side, highlighting similarities, differences, and relative strengths across all dimensions.

## Input

`$ARGUMENTS` should contain references to two games to compare. This can be:
- Two game directory slugs: `catan terraforming-mars`
- Two paths to GDD or rules files
- One game directory + one game name for manual comparison
- A base game + expansion: `catan catan-cities-and-knights`

If not provided, list available game directories in `output/` and ask the user which two to compare.

## Comparison Process

### Step 1: Load Both Games
For each game, read available extraction files from their `output/{game-slug}/` directory:
- `.context.json` for metadata
- `RulesExtraction.md` for rules
- `Mechanics.md` for mechanics
- `BalanceSheet.md` for parameters
- `GameLoop.md` for loop structure
- `GameFitness.md` for fitness scores (if available)

If a file is missing for one game, note it and compare only what's available.

### Step 2: Metadata Comparison
| Dimension | Game A | Game B |
|---|---|---|
| **Players** | | |
| **Play Time** | | |
| **Complexity** | | |
| **Primary Mechanics** | | |
| **Theme** | | |

### Step 3: Mechanics Comparison
Create a comparison matrix:

| Mechanic | Game A | Game B | Notes |
|---|---|---|---|
| Resource Management | Primary | Secondary | A uses fixed income, B uses variable |
| Deck Building | Not present | Primary | — |
| Area Control | Secondary | Primary | Different scoring approaches |
| ... | ... | ... | ... |

Identify:
- **Shared mechanics:** Present in both games (note implementation differences)
- **Unique to Game A:** Mechanics only in Game A
- **Unique to Game B:** Mechanics only in Game B
- **Similar but different:** Same category, different implementation

### Step 4: Game Loop Comparison
Compare the loop structures:
- Turn complexity (phases per turn)
- Game length (turns/rounds)
- Loop nesting depth
- Player interaction timing (during turn vs. between turns)
- End-game trigger type

### Step 5: Balance Parameter Comparison
For parameters that exist in both games, compare:
| Parameter Type | Game A | Game B | Insight |
|---|---|---|---|
| Starting resources | 3 gold | 5 credits | B starts players richer relative to costs |
| Actions per turn | 2 | 1 (with bonus actions) | B has more variable action economy |
| Game length | ~12 rounds | ~25 turns | B runs longer but with simpler turns |

### Step 6: Adaptation Comparison (if both have gap reports)
- Which game is easier to adapt digitally?
- What "Redesign Required" items does each have?
- Which shared mechanics are handled differently?

### Step 7: Fitness Comparison (if both have fitness reports)
| Dimension | Game A | Game B | Advantage |
|---|---|---|---|
| Decisional | X/100 | X/100 | A/B/Tie |
| Balance | X/100 | X/100 | A/B/Tie |
| Engagement | X/100 | X/100 | A/B/Tie |
| Elegance | X/100 | X/100 | A/B/Tie |
| Replayability | X/100 | X/100 | A/B/Tie |
| Implementation | X/100 | X/100 | A/B/Tie |
| **Overall** | **X/100** | **X/100** | **A/B/Tie** |

## Use Cases

### Base Game vs. Expansion
If the two games are a base game and its expansion:
- What mechanics does the expansion add?
- What parameters does it change?
- Does it increase or decrease complexity?
- Does it address any base game weaknesses?

### Similar Games Analysis
If comparing two competing games:
- What does each game do better?
- What could each learn from the other?
- Where would a hybrid approach work?
- Which is better suited for digital adaptation?

## Output Format

```markdown
# Game Comparison: [Game A] vs [Game B]

## At a Glance
[Metadata comparison table]

## Mechanics Overlap
[Venn-style analysis: shared, unique to A, unique to B]

## Key Differences
[Top 5 most significant differences with analysis]

## Key Similarities
[Shared design patterns and approaches]

## Fitness Comparison
[Side-by-side scores if available]

## Verdict
[Summary: what each game does best, which is more complex, which adapts better digitally]
```

## Output

Write to `output/Comparison-{game-a}-vs-{game-b}.md` (at the top level of output/, not inside a game directory).

Display to the user: the metadata comparison table, mechanics overlap summary, and the top 3 most significant differences.
