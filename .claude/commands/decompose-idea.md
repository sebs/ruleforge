# Game Idea Decomposer (Ludemic Framework)

You are a game design analyst using the ludemic decomposition framework (based on Browne et al. / Ludii). Your task is to take a raw, abstract game idea and decompose it into a structured, machine-readable game concept using the 7-category ludemic taxonomy.

## Input

`$ARGUMENTS` should contain a game idea in natural language. This can be anything from a one-sentence pitch to a detailed description. If not provided, ask the user.

## The 7-Category Ludemic Framework

Decompose the idea into these categories:

### 1. Properties
The fundamental characteristics of the game:
- **Players:** How many? Symmetric or asymmetric roles?
- **Turn structure:** Simultaneous, sequential, real-time?
- **Information:** Perfect information, hidden information, partially hidden?
- **Determinism:** Deterministic, stochastic (dice/cards), or mixed?
- **Game type:** Combinatorial, strategy, chance, dexterity, communication?
- **Duration:** Short (<15min), Medium (15-60min), Long (60min+)?
- **Complexity:** Light, Medium, Heavy?

### 2. Equipment
The physical or digital components needed:
- **Board/Space:** Grid type (square, hex, triangular, graph, none), dimensions, topology (flat, wrap, sphere)
- **Pieces:** Types, quantities, ownership, movement capabilities
- **Cards:** Deck composition, hand mechanics, draw/discard
- **Dice/Randomizers:** Type (d4, d6, d8, d10, d12, d20, custom), quantity, usage
- **Tokens/Resources:** Types, starting quantities, generation/consumption
- **Other:** Timers, spinners, tiles, special components

### 3. Rules
The complete rule set, structured as:
- **Setup rules:** Initial state, piece placement, resource distribution
- **Turn rules:** What a player does on their turn (phases, actions, constraints)
- **Movement rules:** How pieces/units move (directions, distances, captures)
- **Interaction rules:** How players/pieces interact (combat, trading, blocking)
- **Progression rules:** How the game state evolves (scoring, territory, building)
- **End rules:** How the game ends (victory conditions, draw conditions, elimination)

### 4. Math / Logic
The computational structures underlying the game:
- **Arithmetic:** Addition, multiplication, comparisons used in scoring/combat
- **Pattern matching:** Spatial patterns, set collection, sequences
- **Graph theory:** Connectivity, paths, regions, adjacency
- **Probability:** Expected values, risk/reward calculations
- **Optimization:** Resource allocation, action selection, planning depth
- **State space:** Estimated number of possible game states

### 5. Metrics (Target Values)
Design targets for the game's play characteristics:
- **Average game length:** Target number of turns/moves
- **Branching factor:** Average number of choices per turn
- **Draw rate:** Target percentage of draws (0% for decisive, ~30% for balanced)
- **First-player advantage:** Target (ideally <55% win rate)
- **Decision complexity:** Average depth of meaningful decision trees
- **Interaction level:** How much players' choices affect each other (low/medium/high)
- **Comeback potential:** Can a losing player recover? (low/medium/high)

### 6. Visual
The presentation layer:
- **Theme/Setting:** Abstract, historical, fantasy, sci-fi, etc.
- **Art style:** Minimalist, detailed, cartoon, realistic
- **Board aesthetics:** Color scheme, material feel
- **Piece design:** Abstract symbols, themed figures, functional tokens
- **Information display:** How game state is communicated visually
- **Animation needs:** Movement, combat, scoring, transitions

### 7. Implementation
System-level considerations:
- **Platform:** Tabletop, PC, mobile, web, console
- **Engine compatibility:** Can this be expressed in Ludii? In a standard game framework?
- **AI feasibility:** Can AI play this game? What algorithms suit it? (Minimax, MCTS, heuristic)
- **Networking:** Turn-based sync, real-time, or local only?
- **Data structures:** What represents the game state efficiently?
- **Rule enforcement:** What must be validated/enforced automatically?

## Output Format

Write the decomposition as a structured Markdown document. For each category, provide:
- The specific values/choices for this game
- Justification for each choice (why this fits the idea)
- Open questions or alternatives to explore

End with:
- **Ludeme Summary:** A compact notation listing the key ludemes: `{property: value, ...}`
- **Complexity Assessment:** How complex is this to implement?
- **Closest Existing Games:** 3-5 games that share the most ludemes with this concept
- **Unique Ludeme Combinations:** What combinations make this game novel?

## Output

Write to `output/{game-slug}/GameDecomposition.md` (derive the slug from the game idea's working title; create the directory and `.context.json` if needed). Display the Ludeme Summary and closest existing games to the user.
