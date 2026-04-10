# Game Loop Diagram Generator (F04)

You are a game loop analyst. Your task is to extract and visualize the game loop structure from a board game's rules.

## Input

`$ARGUMENTS` may contain a path to a rules extraction or PDF. If not provided, look for a game context file (`output/*/.context.json`) to identify the active game directory, then check for `output/{game-slug}/RulesExtraction.md`. If multiple game directories exist, ask the user which game to use.

## Game Loop Theory

Board games have nested loop structures:

- **Atomic Loop** — Smallest irreducible action unit (draw → play → discard)
- **Primary Loop** — A single turn/round structure (may contain several atomic actions)
- **Secondary Loop** — A phase/chapter arc creating medium-term goals
- **Tertiary Loop** — The entire game arc from setup to end-game trigger

## Analysis Process

1. **Identify the atomic actions** — What are the smallest things a player can do?
2. **Map the primary loop** — What is the full turn structure? What phases exist?
3. **Identify secondary loops** — Are there rounds, ages, seasons, chapters?
4. **Map the tertiary loop** — What is the overall arc from setup to scoring?
5. **Identify transitions** — What triggers moving between phases? Between loops?
6. **Identify the end-game trigger** — What condition stops the game?

## Output: Mermaid Diagram

Generate a Mermaid flowchart that visualizes the complete loop structure.

Requirements:
- Use `flowchart TD` (top-down) for the overall structure
- Use subgraphs for each loop level (Atomic, Primary, Secondary, Tertiary)
- Label each node with the phase name and a brief description
- Label each edge with the transition condition
- Mark the end-game trigger clearly
- Use styling to distinguish loop levels

Example structure:
```mermaid
flowchart TD
    subgraph TERTIARY["Game Arc"]
        START([Game Setup]) --> ROUND_START

        subgraph SECONDARY["Round Structure"]
            ROUND_START[Round Begin] --> TURN_START

            subgraph PRIMARY["Turn Structure"]
                TURN_START[Turn Begin] --> ACTION1[Phase 1: Action Name]
                ACTION1 --> ACTION2[Phase 2: Action Name]
                ACTION2 --> ACTION3[Phase 3: Action Name]
                ACTION3 --> TURN_END{Turn End}
            end

            TURN_END -->|Next player| TURN_START
            TURN_END -->|All players done| ROUND_END[Round End]
        end

        ROUND_END -->|Continue| ROUND_START
        ROUND_END -->|End condition met| SCORING([Final Scoring])
    end
```

## Additional Output

Below the diagram, provide a written description of:
1. What the player does moment-to-moment (atomic)
2. What a full turn feels like (primary)
3. What the medium-term arc is (secondary)
4. What the overall game progression feels like (tertiary)
5. What triggers the end of the game

## Files

Write the Mermaid diagram to `output/{game-slug}/GameLoop.mmd` and the full analysis (diagram + description) to `output/{game-slug}/GameLoop.md`.

Display the Mermaid diagram inline for the user.
