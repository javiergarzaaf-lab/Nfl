---
name: nfl-scheme-matrix
description: |
  Point-in-time personnel, formation, coverage, and pressure matrices with EPA vs each dimension.
  Tracks neutral-situation offensive and defensive schemes across: personnel (11/12/21/13),
  shotgun/under-center/pistol, motion, play action, screens, defensive base/nickel/dime,
  box count, blitz/pressure, man/zone, and Cover 0/1/2/3/4/6/2-Man.
  Includes 4-week live reaction layer and EPA/coverage cross-tabs.

  Use when: user asks about team scheme, formation tendencies, personnel packages, coverage distributions,
  pressure rates, play-action effectiveness, or how schemes match up against opponent formations/coverage.
  Don't use when: user asks for scores, standings, or coach-level tactical explanations beyond data.
license: MIT
metadata:
  author: Alphakiller1
  version: "0.2.0"
---

# NFL Scheme Matrix

A point-in-time scheme-family ranking and matchup dashboard for all 32 teams. Tracks offensive
and defensive schemes (personnel, formation, coverage, pressure, play-call) with EPA by dimension
and a 4-week live reaction layer. Authority gate: RESEARCH_ONLY (analysis only, not betting advice).

## Setup

No external API keys required. The model caches nflverse data (schedules, rosters, play-by-play,
participation, depth charts) on a 6-hour TTL. Completed seasons are cached forever.

```bash
# Verify CLI is available
nfl-model status
```

If `nfl-model` is not found, reinstall:

```bash
pip install -e ".[dev]"
```

## Quick Start

### View Scheme Rankings (Current & Historical)

```bash
# Show all teams' scheme matrix for the current week
nfl-model best-bets > report.md

# Specify a season and week
nfl-model best-bets --season 2025 --week 1 > report.md
nfl-model best-bets --season 2024 --week 10 > historical_report.md

# Limit to top matchups (e.g., 10 highest-leverage schemes)
nfl-model best-bets --limit 10 > report.md

# Export as JSON for custom analysis
nfl-model best-bets --out scheme_report.json
```

### Export Full Scheme Matrix (Current & Historical)

```bash
# Full JSON schema payload for all teams this week
nfl-model export > scheme_export.json

# Specify past season/week for historical comparison
nfl-model export --season 2024 --week 5 --out historical_scheme.json
nfl-model export --season 2023 --week 1 --out 2023_w1_scheme.json
```

### View Authority & Model Status

```bash
# Check RESEARCH_ONLY gate and unmet production requirements
nfl-model status
```

---

## Historical Data & Past Results (NEW)

### Get Past Week Results & Standings

**Using nfl-data skill (ESPN + nflverse):**

```bash
# Get scoreboard for a specific week in the past
sports-skills nfl get_scoreboard --week=3 --season=2025

# Get final standings for a season
sports-skills nfl get_standings --season=2024
sports-skills nfl get_standings --season=2023

# Get team schedule (includes results once played)
sports-skills nfl get_team_schedule --team=KC --season=2025

# Get specific game stats/box score
sports-skills nfl get_game_stats --game_id=<espn_game_id>
```

### Get Play-By-Play & EPA History

```bash
# Get detailed play-by-play for a specific game
sports-skills nfl get_play_by_play --game_id=<game_id> --week=3 --season=2025

# Get weekly team stats (EPA, yards, turnovers, etc.)
sports-skills nfl get_nflverse_team_stats --season=2024
sports-skills nfl get_nflverse_team_stats --season=2023

# Get player stats by week
sports-skills nfl get_nflverse_player_stats --season=2025 --week=3
```

### Head-to-Head Historical Matchups

```bash
# Using nfl-matchup-scout with multiple historical seasons:
python -m nfl_scout KC BUF --seasons 2023 2024 2025

# This generates splits across 3 seasons showing:
# - How KC offense historically performs vs BUF defense
# - How BUF offense historically performs vs KC defense
# - Trends across years (improving/declining matchups)
# - Personnel/formation/coverage consistency
```

### Compare Team Scheme Changes Over Time

```bash
# Export multiple seasons for a team
nfl-model export --season 2023 --out 2023_full_scheme.json
nfl-model export --season 2024 --out 2024_full_scheme.json
nfl-model export --season 2025 --out 2025_full_scheme.json

# Then compare:
# - Personnel distribution changes (more 11 vs 12?)
# - Coverage preferences (more man, less zone?)
# - Pressure rates (blitz more aggressive?)
# - Coaching/coordinator changes reflected in scheme
```

### Use Case: Predicting Week 4 Using Historical Momentum

**Example: NE Patriots for Week 4**

```bash
# 1. Get recent results
sports-skills nfl get_scoreboard --week=3 --season=2025
# → See that NE lost to BUF 27-20 in Week 2

# 2. Get EPA trends across weeks 1-3
sports-skills nfl get_nflverse_team_stats --season=2025 --weeks=1,2,3
# → See NE pass EPA declining: +0.18 (W1) → -0.15 (W2) → -0.21 (W3)

# 3. Compare to historical (2024) same time
sports-skills nfl get_nflverse_team_stats --season=2024 --weeks=1,2,3
# → See if NE's 2025 slump is normal or abnormal

# 4. Run scheme matrix for Week 4
nfl-model best-bets --season 2025 --week 4 --limit 5
# → Factor in momentum: NE's pass EPA trend will downgrade their rating
```

## Scheme Matrix Dimensions

The matrix tracks these offensive and defensive families:

### Offensive Scheme

| Dimension | Tracking |
|-----------|----------|
| **Personnel** | 11 (1 RB, 1 TE), 12 (1 RB, 2 TE), 21 (2 RB, 1 TE), 13 (1 RB, 3+ TE), other |
| **Formation** | Shotgun, under-center, pistol rate; motion; play action; screen frequency |
| **Play-call** | Pass rate (early/mid/late down), RPO (run-pass option), screen vs vertical |
| **EPA** | Offensive efficiency per formation; response EPA vs man/zone coverage |
| **Situational** | Neutral-situation vs goal-line; 3rd-down pass rate; red-zone approach |

### Defensive Scheme

| Dimension | Tracking |
|-----------|----------|
| **Personnel** | Base, nickel, dime packages; player counts and specialty usage |
| **Box count** | Average run defense box count; 8-man-box rate |
| **Coverage** | Man vs zone rate; Cover 0/1/2/3/4/6/2-Man distributions |
| **Pressure** | Blitz/pressure rate by down/distance; edge rushers vs interior |
| **EPA** | Defensive efficiency per coverage; yardage allowed vs each scheme type |

## Data Contract & Boundaries

**Explicit provisos:**

- **nflverse participation source:** FTN Data via nflverse under CC-BY-SA 4.0 (2023+)
- **Participation lag:** Published *after* postseason; current in-progress season often lacks data
- **Run proxies:** `run_gap` (guard/tackle/end) and `run_location` are labeled *proximity proxies*,
  not charted blocking schemes
- **Unavailable fields:** Offensive-line zone/gap/power/man blocking and individual assignments
  are not public; matrix marks these as unavailable rather than inferring them
- **Coverage inference:** Defensive coverage classifications derive from formation and positioning
  — not every play has explicit coverage ID
- **Sample stability:** Early-season splits may shift; end-of-season data most reliable
- **4-week reaction layer:** Capped at 35% of the matchup blend to avoid hot-streak overfit

See `reports/SCHEME_DATA_CONTRACT.md` in the repo for the full field dictionary and update contract.

## EPA Cross-Tabs

The matrix includes offensive efficiency (EPA/play) by dimension, broken into:

- **Pass EPA:** Efficiency on pass plays vs each coverage type
- **Rush EPA:** Efficiency on rushing vs box count / blitz rate
- **Situational EPA:** Early down vs red zone vs third-down ESP
- **Defensive EPA allowed:** Points prevented per coverage / formation faced

## Commands in Detail

### best-bets

Ranked scheme matchups + model gaps for the slate.

```bash
nfl-model best-bets [--season YYYY] [--week W] [--limit N] [--out FILE]
```

**Output:** Markdown (stdout) or JSON (if `--out`)

**Contents:**
- Top spread/total gaps (model vs market)
- High-leverage scheme matchups (offense strength vs defense weakness)
- Scheme-ranked team and player matchup explainers
- Ranked by EPA magnitude × expected frequency

### export

Full JSON contract for programmatic access.

```bash
nfl-model export [--season YYYY] [--week W] [--out FILE]
```

**Output:** Single JSON object

**Keys:** (top-level)
- `meta`: model version, authority gate, week/season
- `teams`: 32 team profiles (name, scheme matrix, personnel splits, EPA by coverage)
- `games`: slate games (DraftKings line, model projection, scheme prediction)
- `players`: role-aware QB/RB/WR/TE/K next-game projections
- `scheme`: full scheme matrix versioning and changelog

### board

This week's slate with DraftKings prices and model projections.

```bash
nfl-model board [--season YYYY] [--week W] [--out FILE]
```

### units

Offensive and defensive rankings (component efficiency scores).

```bash
nfl-model units [--season YYYY] [--week W]
```

## Live Reaction Layer (v2)

The 4-week live reaction component tracks current-season performance:

- **Early-down passing:** Rush vs pass tendency this season
- **EPA trends:** Pass/rush EPA and success rate (sample-weighted, shrunk)
- **Explosive plays:** Play rate > 10+ yards
- **Sacks & QB hits:** Defensive pressure efficiency
- **Turnovers:** Fumble/interception rates
- **Red zone:** TD vs FG vs turnover rates
- **Third-down:** Conversion rate and EPA

Each metric is **bounded**: reaction layer never exceeds 35% of the matchup blend
to prevent recent flakes from overriding season-long patterns.

## Authority Gate

**Current:** `RESEARCH_ONLY` — analysis for strategy development only.

Analysis never becomes a bet, sized wager, or trading signal without explicit promotion
through all twelve production gates. Use for:

- ✓ Game prep and scheme scouting
- ✓ Fantasy football research and player analysis
- ✓ Coaching and coordinator strategy study
- ✓ Broadcast prep and color commentary
- ✗ Betting sizes, wager placement, or trading
- ✗ Confidence claims beyond measured uncertainty

## Example Workflow

### 1. Scout an upcoming matchup

```bash
nfl-model best-bets --limit 5 > week3_exploits.md
cat week3_exploits.md | grep -A 3 "KC" | grep -A 3 "LV"
```

Read: High-leverage schemes where KC's strength meets LV's weakness.

### 2. Deep-dive a team's scheme

```bash
nfl-model export > full_export.json
# Parse JSON for SF's personnel distributions, coverage rates, and EPA by formation
```

### 3. Compare week-to-week changes

```bash
nfl-model export --week 1 --out w1_scheme.json
nfl-model export --week 4 --out w4_scheme.json
# Diff the team profiles to see scheme adjustment
```

## Troubleshooting

**Command not found: nfl-model**
- Reinstall: `pip install -e ".[dev]"`
- Or run directly: `python -m nflmodel.cli status`

**nflverse data fails to load**
- Check internet connection
- Verify nflverse CDN is accessible (https://github.com/nflverse)
- Retry after 6-hour cache expiration

**Scheme matrix unavailable for current week**
- Current season: participation data not yet published by nflverse (released post-week)
- Completed seasons: data should be available; check nflverse release schedule

**How recent is this week's data?**
- nflverse data refreshes every 6 hours during the regular season
- Participation data released after each game completes + postseason publication lag
- Run `nfl-model status` to check freshness

