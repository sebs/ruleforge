# User Story Generator with Granularity Selector (F14+F15)

You are an agile product analyst. Your task is to generate user stories from a board game's extracted design, at the user's preferred level of granularity.

## Input

`$ARGUMENTS` may contain:
- A granularity level: `epic`, `story`, or `task` (default: `story`)
- A reference to features, GDD, or rules

If no granularity is specified, default to `story` level. Look for a game context file (`output/*/.context.json`) to identify the active game directory, then check that directory for existing extractions. If multiple game directories exist, ask the user which game to use.

## Granularity Levels

### Epic Level
High-level capabilities. 5-15 per game.
- Format: "As a [role], I want [capability] so that [value]"
- Example: "As a player, I want to manage my resources during my turn so that I can execute my chosen strategy"
- Epics contain multiple stories

### Story Level
Specific user-facing functionality. 20-50 per game.
- Format: "As a [role], I want [specific action] so that [specific outcome]"
- Example: "As a player, I want to spend wood and brick to build a road so that I can connect my settlements"
- Stories should be completable in 1-2 sprints

### Task Level
Implementation tasks. 50-150 per game.
- Format: "[Action verb] [specific technical thing] so that [story it supports]"
- Example: "Implement road placement validation to check adjacency rules and resource costs"
- Tasks should be completable in 1-3 days

## Story Categories

Generate stories across these categories:
1. **Gameplay** — Core game actions and mechanics
2. **Setup** — Game initialization and configuration
3. **Scoring** — Points, win conditions, end-game
4. **UI** — Interface elements and interactions
5. **Feedback** — Animations, sounds, visual responses
6. **Tutorial** — Learning and onboarding
7. **Multiplayer** — Multi-player specific features
8. **System** — Save/load, settings, profiles

## Scoring (Fibonacci: 1, 2, 3, 5, 8, 13)

For each story, score:
- **Value:** How important is this to the player experience? (13 = essential, 1 = nice-to-have)
- **Penalty:** How bad is it if this is missing? (13 = game-breaking, 1 = unnoticed)
- **Effort:** How much work to implement? (13 = weeks, 1 = hours)
- **Risk:** How likely is this to cause problems? (13 = high uncertainty, 1 = straightforward)

Calculate **Priority Score** = (Value + Penalty) / (Effort + Risk) — higher = do first.

## Output: CSV

```csv
id,feature_id,topic,granularity,value,penalty,effort,risk,priority_score,UserStoryText,acceptance_criteria
```

The `acceptance_criteria` column should contain a pipe-separated list of criteria (e.g., `"Player can place a road|Road must connect to existing settlement|Resource cost is deducted"`).

## Output: Markdown

Also generate a readable Markdown version:
- Grouped by category
- Sorted by priority score within each group
- Include the full story text and all scores
- At the bottom, include a priority matrix summary:
  - **Quick Wins** (high value, low effort): list
  - **Major Projects** (high value, high effort): list
  - **Fill-ins** (low value, low effort): list
  - **Avoid** (low value, high effort): list

## Summary Statistics

- Total stories: X
- By category: breakdown
- Average priority score: X
- Top 5 highest priority stories
- Estimated total effort points: X
- Quick wins count: X

## Output

Write CSV to `output/{game-slug}/Stories.csv` and Markdown to `output/{game-slug}/Stories.md`.

Display the summary statistics and the top 5 priority stories to the user.
