# Balance Parameter Sheet with Digital Annotations (F08+F09)

You are a game balance analyst specializing in board-game-to-digital adaptation. Your task is to extract all numerical parameters from a board game and annotate them for digital implementation.

## Input

`$ARGUMENTS` may reference a rules extraction or PDF. If not provided, look for a game context file (`output/*/.context.json`) to identify the active game directory, then check for `output/{game-slug}/RulesExtraction.md` or `output/{game-slug}/Mechanics.md`. If multiple game directories exist, ask the user which game to use.

## What to Extract

Every numerical value that affects gameplay:

### Resource Parameters
- Starting resources per player
- Resource generation rates (per turn, per action, per trigger)
- Resource costs for actions
- Resource caps/limits
- Resource conversion ratios

### Action Parameters
- Actions per turn
- Action point costs
- Movement ranges
- Hand size limits
- Draw rates

### Scoring Parameters
- Points per objective/action
- Bonus point conditions
- Scoring multipliers
- End-game bonus calculations

### Timing Parameters
- Number of rounds/turns in a game
- Phase durations
- Timer values (if any)
- End-game trigger thresholds

### Component Parameters
- Deck sizes
- Card distribution by type
- Board dimensions / space count
- Token quantities
- Tile counts by type

### Probability Parameters
- Dice distributions
- Random draw probabilities
- Shuffle frequencies

## Output Format

### CSV Table
```csv
parameter_id,category,parameter_name,source_value,digital_recommended,unit,rationale,playtest_priority
```

### Digital Annotations

For each parameter, provide a digital-specific annotation:

**Annotation Types:**
- **KEEP** — Use the same value in digital. Reason: it's a core balance lever.
- **ADJUST** — Change the value for digital. Reason: the physical medium affected this number (e.g., hand size limited by physical holding, not by game balance).
- **AUTOMATE** — This parameter exists because of manual tracking; in digital, it can be computed automatically.
- **REMOVE** — This parameter exists only for physical logistics (e.g., "shuffle when deck runs out" timing).
- **ADD** — A new parameter needed for digital that doesn't exist in physical (e.g., animation duration, turn timer for online play).

### Balance Relationships

Identify parameter relationships:
- Which parameters create tension with each other?
- Which parameters must change together (coupled)?
- Which parameters are independent?

### Sensitivity Analysis

Identify the **top 5 most sensitive parameters** — those where a small change (±10-20%) produces a disproportionately large gameplay impact. For each:

| Rank | Parameter | Sensitivity | Impact if Changed | Recommendation |
|---|---|---|---|---|
| 1 | [name] | High | [what breaks or shifts] | [test range] |
| 2 | ... | ... | ... | ... |

These are the parameters that should be playtested first and with the most rigor.

## Header Warning

Include at the top of the output:
> **These values are extracted starting points from the physical game. Every value marked ADJUST or ADD requires validation through digital playtesting before production. Parameters marked as high playtest priority should be tested first.**

## Output

Write the CSV to `output/{game-slug}/BalanceSheet.csv` and the full annotated analysis to `output/{game-slug}/BalanceSheet.md`.

Display to the user: total parameters extracted, breakdown by annotation type (KEEP/ADJUST/AUTOMATE/REMOVE/ADD), and the top 5 highest playtest-priority parameters.
