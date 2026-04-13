# RuleForge

Board game to digital concept extractor — transforms board game rulebooks (PDFs) into development-ready digital game concepts.

RuleForge extracts mechanics, game loops, balance parameters, and produces GDDs, feature lists, user stories, architecture diagrams, and more. It can also translate the extraction into a complete real-time interactive game design.

## How It Works

RuleForge is built as a collection of Claude Code slash commands (skills) that form an extraction and analysis pipeline. Each skill handles a specific stage and writes structured output to a game-scoped directory.

## Quick Start

```
# Quick complexity check before committing
/complexity-estimate path/to/rulebook.pdf

# Run the full pipeline (16 stages, resumable)
/ruleforge path/to/rulebook.pdf

# Or run individual stages
/extract-rules path/to/rulebook.pdf
/identify-mechanics
/game-loop
/generate-gdd
```

## Skills

### Full Pipeline

| Command | Description |
|---|---|
| `/ruleforge` | Full pipeline: PDF to complete developer bundle (F01-F19) |

### Extraction & Analysis

| Command | Description |
|---|---|
| `/extract-rules` | Extract and summarize rules from a PDF |
| `/identify-mechanics` | Identify and categorize game mechanics (25 types) |
| `/game-loop` | Generate game loop diagram (Mermaid) |
| `/validate-loop` | Validate loop structure and state reachability |
| `/adaptation-gap` | Digital adaptation gap report |
| `/balance-sheet` | Balance parameters with sensitivity analysis |
| `/interaction-model` | Component interaction model |
| `/card-database` | Extract card/component data into structured database |
| `/economy-flow` | Resource economy flow diagram |
| `/flag-ambiguities` | Flag ambiguous rules |
| `/confidence-score` | Extraction confidence assessment |
| `/complexity-estimate` | Pre-extraction complexity estimate |

### Design Output

| Command | Description |
|---|---|
| `/generate-gdd` | Full Game Design Document |
| `/feature-list` | Prioritized feature list (CSV + MD + dependency diagram) |
| `/architecture-diagram` | System architecture diagram (Unity/Godot/Phaser/Web) |
| `/user-stories` | User stories with acceptance criteria |
| `/onboarding-design` | Tutorial and onboarding design |
| `/prototype-prompts` | AI prototyping prompts (Rosebud/v0/Bolt/Lovable) |
| `/accessibility-audit` | Accessibility audit across 5 dimensions |

### Export & Packaging

| Command | Description |
|---|---|
| `/stakeholder-export` | Polished stakeholder-ready report |
| `/dev-bundle` | Validate and package developer bundle |
| `/game-comparison` | Side-by-side comparison of two game extractions |

### Translation Pipelines

| Command | Description |
|---|---|
| `/realtime-forge` | Translate extraction into a real-time interactive game design (2D/3D) |

### Standalone Tools

| Command | Description |
|---|---|
| `/pdf-to-markdown` | Convert any PDF to clean Markdown |
| `/decompose-idea` | Decompose a game idea into ludemic framework |
| `/ludeme-generator` | Generate Ludii game description (.lud) |
| `/game-mixer` | Mix mechanics from 2+ games into hybrid designs |
| `/playtest-design` | Design automated playtesting plan |
| `/game-fitness` | Analyze game concept fitness across 6 dimensions |
| `/procedural-generator` | Design procedural generation systems |

## Output Structure

All output is written to game-scoped directories:

```
solo-dungeon-bash/
  RulesExtraction.md
  Mechanics.md
  GDD.md
  Features.csv
  Stories.csv
  GameLoop.mmd
  Architecture.mmd
  BalanceSheet.csv
  ...
  realtime/          # /realtime-forge output
    analysis/
    design/
    architecture/
    balance/
    assets/
    prototypes/
    deployment/
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
