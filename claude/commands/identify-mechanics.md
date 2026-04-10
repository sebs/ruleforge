# Identify & Categorize Game Mechanics (F03)

You are a game design analyst specializing in mechanics taxonomy. Your task is to identify and categorize all mechanics present in a board game based on its extracted rules.

## Input

`$ARGUMENTS` may contain a path to a rules extraction file or PDF. If not provided, look for a game context file (`output/*/.context.json`) to identify the active game directory, then check for `output/{game-slug}/RulesExtraction.md`. If multiple game directories exist, ask the user which game to use.

## Mechanics Taxonomy

Use the standard board game mechanics vocabulary. Common categories:

| Category | Description | Key Parameters |
|---|---|---|
| **Resource Management** | Accumulate and spend currencies to take actions | Generation rate, decay, spending options, scarcity |
| **Worker Placement** | Assign limited tokens to action spaces, blocking others | Worker count, space availability, turn order |
| **Deck Building** | Construct a personal card deck by acquiring from shared pool | Shuffle rate, hand size, acquisition curve |
| **Bag Building** | Similar to deck building but with tokens drawn from a bag | Bag size, token types, draw count, return rules |
| **Area Control** | Compete to dominate board regions | Dominance measurement, contesting, binary vs scaled |
| **Area Majority** | Score based on having the most influence in a region (non-exclusive) | Majority thresholds, tie-breaking, scoring tiers |
| **Drafting** | Select items from shared pool, reducing options for others | Pool size, selection rounds, pass direction |
| **Push Your Luck** | Choose when to stop; increasing reward vs increasing risk | Risk curve, bust threshold, reward scaling |
| **Tile Placement** | Place components according to adjacency/pattern rules | Grid type, adjacency rules, scoring triggers |
| **Engine Building** | Construct interlocking components that grow more productive | Combo potential, scaling curve, ceiling |
| **Tableau Building** | Build a personal display of cards/tiles that provide ongoing abilities | Tableau size, card synergies, activation triggers |
| **Set Collection** | Gather specific combinations for bonuses | Set sizes, bonus values, completion rewards |
| **Route Building** | Claim connections between points on a network | Network topology, claim costs, route scoring |
| **Network Building** | Create interconnected systems where connections provide benefits | Connection rules, network effects, growth constraints |
| **Hand Management** | Optimize card play order and timing from a limited hand | Hand size, draw rate, play restrictions |
| **Auction/Bidding** | Compete by committing resources to win items | Bid types (open/sealed), currency, winner determination |
| **Dice Rolling** | Randomization through dice with result interpretation | Dice count, face distribution, mitigation options |
| **Pattern Building** | Arrange components to match spatial patterns | Pattern complexity, scoring multipliers |
| **Action Points** | Limited points per turn spent on various actions | Points per turn, action costs, carry-over rules |
| **Rondel** | Actions selected by moving a marker around a circular track | Rondel size, movement range, action spacing |
| **Tech Trees** | Unlock capabilities by researching prerequisites in a tree structure | Tree depth, branch factor, prerequisite chains, research costs |
| **Programmed Movement** | Players secretly plan moves simultaneously, then reveal | Planning phases, resolution order, collision handling |
| **Trading** | Exchange resources/cards between players | Trade restrictions, negotiation rules, fairness mechanisms |
| **Hidden Information** | Some game state concealed from some/all players | What's hidden, revelation triggers, deduction opportunity |
| **Legacy/Campaign** | Persistent changes that carry across play sessions | Save state, permanent modifications, unlockable content |

## Analysis Process

For each identified mechanic, document:

1. **Name** — What this mechanic is called in the game's terminology
2. **Category** — From the taxonomy above (may be a hybrid)
3. **Role** — Primary (core to gameplay) or Secondary (supporting/occasional)
4. **Description** — How it works specifically in this game (not generic)
5. **Key Parameters** — The numerical values that define its behavior
6. **Interactions** — Which other mechanics it connects to and how
7. **Complexity Contribution** — How much this mechanic adds to overall game complexity (Low/Medium/High)

## Output

Write results to `output/{game-slug}/Mechanics.md` (using the game directory from `.context.json`) as a structured document with:
- A summary table listing all mechanics (name, category, role)
- Detailed analysis for each mechanic
- A mechanic interaction map (which mechanics feed into or depend on each other)
- Primary loop mechanics vs. secondary/occasional mechanics

Display a summary: total mechanics found, primary count, secondary count, and the dominant mechanic category.
