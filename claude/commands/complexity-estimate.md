# Pre-Extraction Complexity Estimate (F18)

You are a board game complexity analyst. Your task is to quickly assess a rulebook's complexity BEFORE running the full extraction pipeline, so the user can decide whether to proceed.

## Input

`$ARGUMENTS` should contain a path to a PDF rulebook. If not provided, ask.

## Quick Assessment (do NOT do full extraction)

For speed, read only the **first 3 pages** (title, overview, setup) and **last 2 pages** (scoring, FAQ, appendix) to form the estimate. Only scan additional pages if the initial read is insufficient to classify complexity.

Assess:

### 1. Document Metrics
- **Page count:** total pages
- **Estimated word count:** approximate
- **Has images/diagrams:** yes/no (increases parsing difficulty)
- **Has tables:** yes/no
- **Has sidebars/callouts:** yes/no
- **Multi-column layout:** yes/no
- **PDF quality:** text-based / image-based / mixed

### 2. Game Complexity Indicators
Scan for indicators of complexity:
- **Number of distinct sections:** (more sections = more rules)
- **Glossary present:** yes/no (glossary suggests complex terminology)
- **FAQ/appendix length:** (long FAQ = many edge cases)
- **"Exception" / "instead" / "unless" frequency:** (high = exception-heavy rules)
- **Component types mentioned:** count distinct types
- **Card/tile variety:** estimated number of unique types
- **Player interaction type:** solo/competitive/cooperative/mixed

### 3. Complexity Classification

| Level | Pages | Mechanics (est.) | Edge Cases | Examples |
|---|---|---|---|---|
| **Simple** | 1-8 | 1-2 | Few | Ticket to Ride, Azul, Codenames |
| **Medium** | 8-20 | 2-4 | Moderate | Catan, 7 Wonders, Wingspan |
| **Complex** | 20-40 | 4-6 | Many | Dominion, Terraforming Mars |
| **Very Complex** | 40+ | 6+ | Extensive | Arkham Horror, Twilight Imperium, Gloomhaven |

### 4. Confidence Preview

Based on complexity, estimate extraction confidence:
- Simple → Expected confidence: **High (80-95%)**
- Medium → Expected confidence: **Medium-High (65-85%)**
- Complex → Expected confidence: **Medium (50-70%)**
- Very Complex → Expected confidence: **Low (30-55%)** — recommend human-assisted tier

### 5. Extraction Risk Factors
Flag any specific risks:
- Image-only PDF (OCR needed — significant quality loss)
- Non-English text
- Heavily visual rules (diagrams carry rule information)
- References external materials ("see campaign guide")
- Modular/expansion rules mixed with base game

## Output Format

```
========================================
COMPLEXITY ESTIMATE: [Game Title]
========================================

Complexity Level:  [Simple / Medium / Complex / Very Complex]
Expected Confidence: [High / Medium / Low]
Estimated Extraction Time: [Fast / Standard / Extended]
Recommendation: [Proceed / Proceed with caution / Consider human-assisted]

Document: X pages, ~X words
Layout: [text-based / image-heavy / mixed]
Estimated Mechanics: X
Risk Factors: [list or "None identified"]
========================================
```

## Decision Support

Based on the estimate, advise the user:
- **Proceed** — Good candidate for automated extraction
- **Proceed with caution** — Will produce useful output but expect some manual review
- **Consider human-assisted** — Automated extraction likely to produce low-confidence output; suggest breaking into smaller parts or supplementing with manual review

## Output

Write to `output/{game-slug}/ComplexityEstimate.md` (create the game-scoped directory using the game title found during assessment, and write the `.context.json` file if it doesn't already exist). Display the summary box to the user inline.
