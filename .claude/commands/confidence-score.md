# Extraction Confidence Score (F17)

You are a quality assurance analyst for game concept extraction. Your task is to assess and score the confidence level of an extraction.

## Input

`$ARGUMENTS` may reference extraction output files. If not provided, look for a game context file (`output/*/.context.json`) to identify the active game directory, then check that directory for all existing extraction files and assess them collectively. If multiple game directories exist, ask the user which game to use.

## Confidence Dimensions

Score each dimension from 0-100%:

### 1. Rules Extraction Confidence
- Were all rulebook sections readable and parseable?
- Were there sections that couldn't be extracted (images, tables, sidebars)?
- How many terminology inconsistencies were found and resolved?
- Are there gaps in the extracted rules (missing setup, missing end-game, etc.)?

### 2. Mechanics Identification Confidence
- Were all mechanics clearly identifiable from the rules?
- Are the mechanic categorizations unambiguous?
- Were there mechanics that could belong to multiple categories?
- Are the numerical parameters for each mechanic complete?

### 3. Game Loop Confidence
- Is the turn structure complete and unambiguous?
- Are all transitions between phases defined?
- Are there any dead-end states or missing exit conditions?
- Does the loop validation pass?

### 4. Balance Parameters Confidence
- Are all numerical values explicitly stated in the rulebook (not inferred)?
- Are the digital annotations well-grounded?
- Are parameter relationships identified?
- Are there missing parameters that should exist but weren't found?

### 5. Digital Adaptation Confidence
- Are adaptation assessments (No Change / Simple / Redesign) well-justified?
- Are alternative approaches for Redesign items concrete?
- Were all mechanics assessed?

## Scoring Formula

Each dimension is scored 0-100% based on:
- **Completeness** (40% weight) — How much of the expected information was extracted?
- **Clarity** (30% weight) — How unambiguous is the extracted information?
- **Verifiability** (30% weight) — Can each extracted item be traced to a specific rulebook source?

**Overall Confidence** = Weighted average of all dimensions, with the lowest-scoring dimension contributing an extra penalty:
`Overall = (avg of all dimensions) - (penalty if any dimension < 50%)`

## Confidence Bands

| Score | Band | Meaning | Recommended Action |
|---|---|---|---|
| 80-100% | **High** | Output is reliable and development-ready | Proceed to development |
| 60-79% | **Medium** | Output is a solid draft, review flagged sections | Review ambiguities, then proceed |
| 40-59% | **Low** | Output requires significant manual review | Treat as draft; manual review required |
| 0-39% | **Very Low** | Output is unreliable | Consider human-assisted extraction |

## Output Format

### Summary Badge
```
Extraction Confidence: [Band] — [Overall]%
```

### Per-Section Breakdown
| Section | Score | Band | Key Issues |
|---|---|---|---|
| Rules Extraction | X% | Band | ... |
| Mechanics | X% | Band | ... |
| Game Loop | X% | Band | ... |
| Balance | X% | Band | ... |
| Adaptation | X% | Band | ... |
| **Overall** | **X%** | **Band** | |

### Issue Details
For each section scoring below 80%, list:
- What drove the score down
- What would improve it
- Whether manual intervention is needed

### Low-Confidence Escalation
If overall confidence < 60%, explicitly state:
> "This extraction is rated Low confidence. The output should be treated as a first draft requiring significant manual review. Consider: (1) reviewing all flagged ambiguities, (2) cross-referencing key mechanics against the original rulebook, (3) playtesting balance parameters before implementation."

## Output

Write to `output/{game-slug}/Confidence.md`. Display the summary badge and per-section breakdown to the user.
