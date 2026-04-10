# Automated Playtest Design

You are a game testing analyst. Your task is to design a comprehensive playtesting plan for a game concept — including metrics, fitness functions, test scenarios, and evaluation criteria that could drive automated playtesting via AI agents.

## Input

`$ARGUMENTS` may reference a GDD, game decomposition, or rules file. If not provided, look for a game context file (`output/*/.context.json`) to identify the active game directory, then check that directory for existing files. If multiple game directories exist, ask the user which game to use.

## Playtest Framework

### 1. Define Testable Hypotheses

For each core mechanic, formulate a testable hypothesis:

| Mechanic | Hypothesis | How to Test | Success Criteria |
|---|---|---|---|
| Combat system | Combat outcomes are not predetermined by position | Run N games, measure correlation between initial position and win rate | Win rate variance < 5% across starting positions |
| Resource economy | No single resource strategy dominates | Track resource allocation patterns of winning players | No single pattern wins >60% of games |
| ... | ... | ... | ... |

### 2. Define Fitness Functions

Fitness functions evaluate how "good" a game is. Design functions for:

**Balance Fitness:**
- `first_player_advantage = |win_rate_P1 - 0.5|` → minimize (target: <0.05)
- `strategy_diversity = entropy(winning_strategies)` → maximize
- `comeback_rate = wins_by_trailing_player / total_games` → target range (0.15-0.35)

**Engagement Fitness:**
- `decisiveness = 1 - draw_rate` → target >0.7
- `game_length_consistency = stddev(game_lengths) / mean(game_lengths)` → minimize (<0.3)
- `decision_depth = avg(meaningful_choices_per_turn)` → target range (3-8)
- `interaction_rate = turns_with_opponent_impact / total_turns` → target >0.5

**Accessibility Fitness:**
- `learning_curve = turns_until_competent_play` → minimize
- `rules_complexity = total_unique_rules` → appropriate for target audience
- `cognitive_load = max(simultaneous_considerations_per_decision)` → target <5

**Fun Proxies (measurable surrogates):**
- `tension = variance(score_difference_over_time)` → maximize within bounds
- `surprise = frequency(lead_changes)` → target 2-5 per game
- `agency = correlation(player_skill, win_rate)` → target >0.6

### 3. Design Test Scenarios

**Scenario A: Baseline Balance Test**
- Setup: Default starting positions
- Players: 2 random AI agents of equal strength
- Runs: 10,000 games
- Measure: Win rate per side, average game length, draw rate
- Pass criteria: Win rate between 45-55%, draw rate <30%

**Scenario B: Strategy Diversity Test**
- Setup: Default
- Players: 5 different AI strategies (aggressive, defensive, economic, random, mixed)
- Runs: 1,000 games per strategy pairing
- Measure: Win matrix, dominant strategy detection
- Pass criteria: No strategy wins >65% against all others

**Scenario C: Edge Case Test**
- Setup: Various non-standard starting positions
- Players: Random AI
- Runs: 1,000 per setup
- Measure: Games that end in invalid states, infinite loops, or degenerate positions
- Pass criteria: 0% invalid states, 0% infinite loops

**Scenario D: Player Count Scaling Test** (if applicable)
- Setup: Default for each player count
- Players: Equal AI agents
- Runs: 5,000 per player count
- Measure: Game length, interaction rate, kingmaker frequency
- Pass criteria: All metrics within acceptable range for each count

**Scenario E: Parameter Sensitivity Test**
- Setup: Vary each balance parameter ±20% from default
- Players: Random AI
- Runs: 1,000 per parameter variation
- Measure: How much each parameter change affects fitness functions
- Pass criteria: Identify which parameters are most sensitive

### 4. Design the Evaluation Report

After automated playtesting, the report should include:

**Summary Dashboard:**
- Overall fitness score (weighted combination of all fitness functions)
- Pass/fail for each scenario
- Red flags (any metric critically outside bounds)

**Detailed Metrics:**
- Distribution plots for game length, score differentials, branching factor
- Win rate confidence intervals
- Strategy dominance matrix (heatmap)
- Parameter sensitivity rankings

**Recommendations:**
- Parameters to adjust (with suggested direction)
- Rules to modify (with rationale)
- Mechanics to reconsider (if fundamental issues found)
- Additional testing needed

### 5. Manual Playtest Protocol

For aspects that AI cannot test (fun, theme, social dynamics):

**Session Design:**
- Number of playtesters: 4-8 per session
- Experience mix: 50% target audience, 25% experienced gamers, 25% novices
- Sessions per round: 3-5
- Duration: 2-3 hours including debrief

**Observation Checklist:**
- [ ] Do players understand the rules after explanation?
- [ ] Are there "aha" moments during play?
- [ ] Do players make meaningful choices (or feel railroaded)?
- [ ] Is there social interaction (discussion, negotiation, reaction)?
- [ ] Do players want to play again?
- [ ] What questions do players ask most frequently?

**Post-Game Survey:**
1. Rate your enjoyment (1-10)
2. Was the game too short / just right / too long?
3. Did you feel your decisions mattered?
4. Was any mechanic confusing?
5. Would you play again? Why/why not?
6. What would you change?

## Output

Write to `output/{game-slug}/PlaytestDesign.md`. Display the fitness functions and key test scenarios to the user.
