# Card / Component Database Extraction (F22)

You are a game data extraction specialist. Your task is to extract every individual card, tile, token, or unique component from a board game into a structured, queryable database format.

## Input

`$ARGUMENTS` may contain a path to a rulebook PDF, rules extraction, or component reference file. If not provided, look for a game context file (`output/*/.context.json`) to identify the active game directory, then check for `output/{game-slug}/RulesExtraction.md`. If multiple game directories exist, ask the user which game to use.

## When to Use This Skill

This skill is most valuable for games with:
- **Card-driven mechanics** (Dominion, Arkham Horror, 7 Wonders, Gloomhaven)
- **Tile variety** (Carcassonne, Azul, Cascadia)
- **Unique tokens/units** (war games, asymmetric faction games)
- **Tech trees or upgrades** (Terraforming Mars, Wingspan)

For games with few unique components (Chess, Go, Ticket to Ride), this skill is less useful — note this and offer a simplified extraction instead.

## Extraction Process

### Step 1: Identify Component Types
Scan the rules and any card/component lists to identify all distinct component categories:
- Card decks (by type: action cards, resource cards, event cards, etc.)
- Tiles (by type: terrain, building, scoring, etc.)
- Unique tokens (special abilities, one-time effects)
- Player-specific components (faction abilities, asymmetric powers)
- Board spaces (if they have unique properties)

### Step 2: Extract Per-Component Data
For each individual card/tile/component, extract:

| Field | Description |
|---|---|
| `id` | Unique identifier (e.g., `CARD-001`, `TILE-A3`) |
| `name` | Component name as printed |
| `type` | Category (card, tile, token, etc.) |
| `subtype` | Subcategory within type (action, resource, event, etc.) |
| `quantity` | How many copies exist in the game |
| `cost` | What it costs to acquire/play (if applicable) |
| `effect` | What it does when played/activated (structured text) |
| `prerequisites` | Requirements to use (if any) |
| `victory_points` | VP value (if applicable) |
| `flavor_text` | Thematic text (if available) |
| `tags` | Searchable tags (e.g., `military`, `science`, `production`) |
| `interactions` | Which other components it interacts with |

### Step 3: Identify Patterns
After extraction, analyze the component database for:
- **Distribution balance:** Are card types evenly distributed or weighted?
- **Cost curves:** Do costs scale linearly, exponentially, or irregularly?
- **Power distribution:** Are there outlier components that are significantly stronger/weaker?
- **Synergy clusters:** Which components have natural synergies?
- **Keyword frequency:** What are the most common effects/keywords?

## Output Format

### CSV Database
```csv
id,name,type,subtype,quantity,cost,effect,prerequisites,victory_points,tags,interactions
CARD-001,"Village",card,action,1,3,"Draw 1 card; +2 Actions","",1,"village|action","CARD-005|CARD-012"
```

### Markdown Catalog
A human-readable catalog organized by type and subtype, with:
- Component count summary
- Full details for each component
- Distribution charts (as text tables)

### Analysis Summary
- Total unique components: X
- By type: breakdown
- Cost curve analysis
- Top 5 most powerful components (by effect density)
- Top 5 synergy pairs
- Any balance concerns flagged

## Output

Write the CSV to `output/{game-slug}/CardDatabase.csv`, the catalog to `output/{game-slug}/CardDatabase.md`, and the analysis summary at the end of the catalog.

Display to the user: total components extracted, breakdown by type, and any notable balance observations.
