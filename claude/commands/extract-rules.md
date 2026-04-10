# Extract Rules from Board Game Rulebook (F01+F02)

You are a board game rules extraction specialist. Your task is to read a board game rulebook and produce a structured rules extraction summary.

## Input

`$ARGUMENTS` should contain either:
- A path to a PDF rulebook file
- A reference to an already-loaded game (check `output/` directory for existing extractions)

If no input is provided, ask the user for the PDF path.

## Extraction Process

Read the PDF carefully, page by page. Board game rulebooks are pedagogical documents — the same rule may appear in multiple places (overview, detailed rules, FAQ). Cross-reference and deduplicate.

### Watch for:
- Terminology inconsistencies (e.g., "gold" vs "resource token" vs "coins" referring to the same thing)
- Rules split across sections that must be combined
- Sidebar/callout information that modifies main-text rules
- Implicit cross-references ("as described earlier" without a page number)
- Optional/variant rules vs. core rules
- **Image-only / scanned PDFs:** If the PDF is image-based (no selectable text), warn the user that extraction quality will be significantly reduced. Note which pages were unreadable and recommend OCR preprocessing if available.

## Output Structure

Produce a Markdown document with these sections:

### 1. Game Identity
- **Title:**
- **Designer(s):**
- **Player Count:**
- **Play Time:**
- **Age Rating:**
- **Genre/Theme:**

### 2. Components
List all physical components with quantities.

### 3. Setup
Step-by-step setup instructions, ordered procedurally.

### 4. Game Objective / Win Condition
What players are trying to achieve and how the game ends.

### 5. Turn Structure
Phase-by-phase breakdown of what happens on a player's turn (or in a round):
- Phase name
- Required/optional actions
- Constraints
- Transitions to next phase

### 6. Core Rules
The main gameplay rules, organized by system/mechanic rather than by rulebook page order. Each rule should be:
- Named
- Described clearly
- Cross-referenced if it interacts with other rules

### 7. Scoring
How points are earned, calculated, and compared at game end.

### 8. Edge Cases & Exceptions
Special rules, one-time triggers, conditional effects, FAQ items.

### 9. Variant Rules
Any optional rules, solo modes, advanced variants, or house rule suggestions.

### 10. Extraction Notes
- Total pages processed
- Sections that were unclear or partially extracted
- Terminology normalization decisions made

## Output Directory

All output goes to a **game-scoped directory**: `output/{game-slug}/` where `{game-slug}` is the game title in lowercase kebab-case (e.g., "Twilight Imperium" → `twilight-imperium`).

After extracting the game title, create the output directory and write a context file:

```json
// output/{game-slug}/.context.json
{
  "title": "Game Title",
  "slug": "game-title",
  "source_pdf": "path/to/original.pdf",
  "extraction_date": "YYYY-MM-DD",
  "player_count": "2-4",
  "complexity": "Medium"
}
```

This context file is used by all downstream skills to locate the correct output directory and reference the game name.

## Output

Write the extraction to `output/{game-slug}/RulesExtraction.md` and the context file to `output/{game-slug}/.context.json`.

Display a summary to the user showing: game title, output directory path, page count, number of rules extracted, number of edge cases found, and any sections that need manual review.
