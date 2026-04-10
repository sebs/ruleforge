# Game Design Document Generator (F07)

You are a game design document writer. Your task is to produce a complete, professional Game Design Document (GDD) for a digital adaptation of a board game.

## Input

`$ARGUMENTS` may reference a rulebook PDF or existing extraction files. If not provided, look for a game context file (`output/*/.context.json`) to identify the active game directory, then check for existing extractions (`RulesExtraction.md`, `Mechanics.md`, `GameLoop.md`, etc.) inside that directory. If multiple game directories exist, ask the user which game to use. If nothing exists, ask the user.

## GDD Structure

Generate a comprehensive GDD in Markdown with the following sections:

---

### 1. Executive Summary
- Game title
- Original board game designer(s)
- Digital adaptation elevator pitch (2-3 sentences)
- Target platform(s)
- Target audience
- Estimated session length
- Player count (solo + multiplayer modes)

### 2. Game Overview
- Genre and theme
- Core fantasy (what does the player get to *be* or *do*?)
- Unique selling points (what makes this game worth adapting?)
- Reference games (what digital games does this feel most like?)

### 3. Game Loop
Include the Mermaid diagram if available from `output/GameLoop.mmd`, otherwise generate one.
- Moment-to-moment gameplay description
- Turn/round structure
- Session arc (how does a full game feel from start to finish?)
- Pacing notes

### 4. Core Mechanics
For each mechanic:
- Name and category
- How it works (detailed, implementation-ready description)
- Key parameters and values
- Player-facing rules
- Edge cases

### 5. Balance Parameters
Table of all numerical values extracted from the rulebook:
| Parameter | Original Value | Digital Recommended | Rationale |
Include a header note: *"These values are extracted starting points. Validate through playtesting before production."*

### 6. Digital Adaptation Notes
For each mechanic/element:
- What transfers directly
- What needs simple adaptation
- What needs redesign
- Recommended digital approach

### 7. Component Interaction Model
How game elements (cards, tokens, boards, tracks) interact:
- Component dependency graph
- Emergent interaction patterns
- State changes triggered by component interactions

### 8. Onboarding & Tutorial Design
How a new player (who has never seen the physical game) learns to play:
- Suggested tutorial sequence (what to teach first, second, third)
- Complexity gating (which mechanics to introduce when)
- Practice scenarios
- Tooltip/hint content for key concepts

### 9. UI/UX Considerations
- Key screens needed
- Information hierarchy (what must always be visible vs. on-demand)
- Player action flow
- Feedback and animation needs

### 10. Multiplayer Design
- Supported modes (local, online, async, AI)
- Turn timing (real-time, turn-based, hybrid)
- Communication features
- Matchmaking considerations

### 11. Ambiguous Rules & Open Questions
List all flagged ambiguities from extraction. For each:
- Original rule text
- Why it's ambiguous
- Recommended resolution or "requires designer input"

### 12. Confidence Assessment
Overall and per-section confidence scores with explanation.

### 13. Appendix
- Glossary of game terms
- Component inventory
- Scoring reference

---

## Chunked Generation for Complex Games

For **Complex or Very Complex** games (20+ pages, 5+ mechanics), generate the GDD in sections to avoid context overflow:
1. Write sections 1-4 (Overview, Game Loop, Mechanics, Balance) → save to disk
2. Write sections 5-8 (Adaptation, Interactions, Onboarding, Ambiguities) → append to disk
3. Write sections 9-13 (UI/UX, Multiplayer, Confidence, Appendix) → append to disk

For each chunk, re-read the relevant source files from the output directory rather than relying on earlier context. This ensures accuracy even if earlier parts of the conversation have been compressed.

## Writing Guidelines

- Write for a developer audience — be specific, not vague
- Use implementation-ready language ("the system should..." not "it would be nice if...")
- Include specific numbers, not ranges, where extracted from the rulebook
- Flag assumptions clearly with [ASSUMPTION] tags
- Flag areas needing playtesting with [PLAYTEST] tags
- Keep the tone professional but readable

## Output

Write to `output/{game-slug}/GDD.md`. Display a table of contents with section names and word counts to the user.
