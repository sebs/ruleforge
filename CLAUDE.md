# RuleForge — Board Game to Digital Concept Extractor

## Project Overview

RuleForge transforms board game rulebooks (PDFs) into development-ready digital game concepts. It extracts mechanics, game loops, balance parameters, and produces GDDs, feature lists, user stories, and architecture diagrams.

## Available Skills (Slash Commands)

The full pipeline and individual extraction skills are in `.claude/commands/`:

### Pipeline Skills

| Command | Features | Description |
|---|---|---|
| `/ruleforge` | F01-F19 | Full pipeline: PDF → complete developer bundle (16 stages, resumable) |
| `/extract-rules` | F01+F02 | Extract and summarize rules from a PDF |
| `/identify-mechanics` | F03 | Identify and categorize game mechanics (25 mechanic types) |
| `/game-loop` | F04 | Generate game loop diagram (Mermaid) |
| `/validate-loop` | F05 | Validate loop for structural correctness + state reachability |
| `/adaptation-gap` | F06 | Digital adaptation gap report + accessibility considerations |
| `/generate-gdd` | F07 | Full Game Design Document (chunked for complex games) |
| `/balance-sheet` | F08+F09 | Balance parameters with digital annotations + sensitivity analysis |
| `/interaction-model` | F10 | Component interaction model |
| `/onboarding-design` | F11 | Tutorial and onboarding design |
| `/feature-list` | F12 | Prioritized feature list (CSV + MD + dependency diagram) |
| `/architecture-diagram` | F13 | System architecture diagram (supports Unity/Godot/Phaser/Web/generic) |
| `/user-stories` | F14+F15 | User stories with granularity selector + acceptance criteria |
| `/prototype-prompts` | F16 | AI prototyping prompts (supports Rosebud/v0/Bolt/Lovable/generic) |
| `/confidence-score` | F17 | Extraction confidence assessment |
| `/complexity-estimate` | F18 | Pre-extraction complexity estimate (fast scan) |
| `/flag-ambiguities` | F19 | Ambiguous rule flagging |
| `/stakeholder-export` | F20 | Polished stakeholder-ready report (MD → PDF) |
| `/dev-bundle` | F21 | Package and validate developer bundle (with Mermaid validation) |
| `/card-database` | F22 | Extract individual card/tile/component data into structured database |
| `/economy-flow` | F23 | Resource economy flow diagram (sources → sinks → conversions) |
| `/accessibility-audit` | F24 | Digital accessibility audit across 5 dimensions |
| `/game-comparison` | F25 | Side-by-side comparison of two game extractions |

### Translation Pipelines

| Command | Description |
|---|---|
| `/realtime-forge` | Translate RuleForge output into a complete real-time interactive game design (2D/3D). 7 waves, ~30 output files covering analysis, RTGDD, revised docs, architecture, balance, assets, prototypes, deployment. |

### Standalone Skills

| Command | Description |
|---|---|
| `/pdf-to-markdown` | Convert any PDF to clean, well-structured Markdown |
| `/procedural-generator` | Design procedural generation systems (Watson et al. 2008 workflow) |
| `/decompose-idea` | Decompose a game idea into 7-category ludemic framework |
| `/ludeme-generator` | Generate Ludii game description (.lud) from a concept |
| `/game-mixer` | Mix mechanics from 2+ games into hybrid designs (with iteration) |
| `/playtest-design` | Design automated playtesting plan with fitness functions |
| `/game-fitness` | Analyze game concept fitness across 6 dimensions |

## Typical Workflow

1. `/complexity-estimate path/to/rulebook.pdf` — Quick assessment before committing
2. `/ruleforge path/to/rulebook.pdf` — Full pipeline (or run individual skills)
3. `/card-database` — Extract card/component data (for card-heavy games)
4. `/economy-flow` — Map the resource economy
5. `/accessibility-audit` — Check for accessibility barriers
6. `/dev-bundle` — Validate and package all output files
7. `/realtime-forge` — Translate into a real-time interactive game design (2D/3D)

## Output Directory

All skills write to **game-scoped directories**: `output/{game-slug}/` where `{game-slug}` is derived from the game title in lowercase kebab-case. Each game directory contains a `.context.json` metadata file used by downstream skills.

Example structure:
```
output/
  catan/
    .context.json
    RulesExtraction.md
    Mechanics.md
    GDD.md
    Features.csv
    ...
  terraforming-mars/
    .context.json
    ...
```

Key files per game:
- `.context.json` — Game metadata (title, slug, source PDF, extraction date)
- `GDD.md`, `Features.csv`, `Stories.csv`, `Architecture.mmd`, `GameLoop.mmd`

## Research

The `research/` directory contains the product research behind RuleForge:
- `Vision.md` — Product vision
- `Topic.md` — Game design theory (mechanics, loops, balance, adaptation)
- `Design-Thinking.md` — Emotional journey map, Walt Disney method, Wizard of Oz prototype
- `Features.csv` — 25-feature inventory
- `Stories.csv` — 33 user stories with scoring
- `Market.md` — Market analysis
- `Prototype.md` — Rosebud.ai prototype specifications
- `Architecture.mmd` — System architecture diagram

## Game Design Domain Knowledge

When working on this project, use standard game design terminology:
- **Mechanics:** Resource Management, Worker Placement, Deck Building, Bag Building, Area Control, Area Majority, Drafting, Push Your Luck, Tile Placement, Engine Building, Tableau Building, Set Collection, Route Building, Network Building, Hand Management, Auction/Bidding, Dice Rolling, Pattern Building, Action Points, Rondel, Tech Trees, Programmed Movement, Trading, Hidden Information, Legacy/Campaign
- **Loops:** Atomic (smallest action), Primary (turn), Secondary (round/phase), Tertiary (game arc)
- **Balance:** Catch-up mechanics, diminishing returns, opportunity cost, tempo, sensitivity analysis
- **Adaptation:** No Change Required / Simple Adaptation / Redesign Required
- **Accessibility:** Visual, Motor, Cognitive, Hearing, Communication dimensions
