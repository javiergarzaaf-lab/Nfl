---
name: nfl-matchup-scout
description: |
  Offensive and defensive situational profiling with cross-team matchup analysis.
  Identifies statistical exploits by comparing one team's strength against another's
  weakness (z-scores, EPA/play, success rate) across 21 split dimensions:
  personnel, formation, field position, down/distance, pressure, coverage, play action.

  Use when: user asks about team matchups, offensive vs defensive advantages, personnel splits,
  formation tendencies, pressure/coverage matchups, play-action effectiveness, or detailed
  exploit analysis between two specific teams.
  Don't use when: user asks for scores, standings, schedules, or general season-wide trends.
license: MIT
metadata:
  author: DarrellJBullock
  version: "1.0.0"
---

# NFL Matchup Scout

Profiles every team's offense and defense across 21 situational splits (personnel, formation,
field position, down/distance, coverage, pressure, play action). Flags z-scores vs league
average and ranks matchup exploits by magnitude × expected frequency.

## Setup

First run downloads nflverse release parquet files (~25 MB per season) into `data/cache/`.
Current season files refresh every 24 hours. No API keys required.

```bash
# Verify the CLI is available
which python && python --version  # requires Python 3.8+
```

## Quick Start

Generate a Markdown report comparing two teams:

```bash
python -m nfl_scout KC LV --seasons 2025 2026 > report.md
python -m nfl_scout SF GB --seasons 2024 2025 > report.md
```

Launch an interactive Streamlit dashboard:

```bash
streamlit run app.py
```

Run tests (synthetic data, no network):

```bash
python -m pytest -q
```

## Commands

### Generate Matchup Report

```bash
python -m nfl_scout <TEAM1> <TEAM2> [--seasons YYYY [YYYY ...]]
```

**Arguments:**
- `<TEAM1>`, `<TEAM2>`: NFL team abbreviations (e.g., `KC`, `SF`, `NYG`)
- `--seasons YYYY [YYYY ...]`: Season years to analyze (default: current season + prior)

**Output:** Markdown report with:
- **High-leverage exploits**: Ranked by z-score magnitude and expected frequency
  - Offensive strength vs defensive weakness (e.g., "CHI off-tackle runs vs NYG run defense")
  - Defensive strength vs offensive weakness
- **Situational profiles**: EPA/play, success rate, sample size per split
- **Personnel**: WR alignments, run personnel (11/12/21), pass rushers
- **Formation**: Shotgun/under-center, motion, play action, screen frequency
- **Coverage/pressure**: Man/zone splits, blitz rate, box count
- **Play action**: Success rate, EPA, when most effective

**Example:**
```bash
python -m nfl_scout KC LV --seasons 2025 2026 > kc-lv-scout.md
# Read the report: cat kc-lv-scout.md
```

### Interactive Dashboard

```bash
streamlit run app.py
```

**Features:**
- Matchup explorer: pick two teams, filter by season and split
- Team profile: offense/defense radar charts, win/loss distributions
- League heatmap: all 32 × 32 team matchups ranked by exploit magnitude
- Methodology: full explanation of splits, z-scores, statistical approaches

## Data Sources

| Source | Used for |
|---|---|
| `nflverse-data` → `pbp` (by `nflverse-pbp`) | EPA, success rate, down/distance, field position, run gap/location, dropback type |
| `nflverse-data` → `pbp_participation` | Offensive/defensive personnel, formation, box count, pass rushers, pressure, man/zone coverage |
| `nflverse-data` → `ftn_charting` | Play action tracking |
| nflverse `nextgen_stats` | Team context: time-to-throw, aggressiveness, CPOE, RYOE, 8-man-box rate |

**Data lag:** Participation data lags play-by-play. For in-progress seasons without participation data,
personnel, pressure, coverage, and box splits are unavailable, and the CLI reports why.

## Limits & Caveats

- **Sample size:** Splits with n < 10 plays are included but flagged; z-scores on tiny samples are unreliable
- **Seasonal consistency:** Early-season splits may shift as teams adjust; end-of-season data is most stable
- **Participation lag:** Current season often lacks participation data; wait for postseason archive
- **Personnel filters:** Some splits combine rare personnel packages (e.g., 3-WR + TE). Frame counts are explicit

## Troubleshooting

**ImportError: No module named 'nfl_scout'**
- Run from the repo root: `python -m nfl_scout ...`
- Or install in editable mode: `pip install -e .`

**Data not found / download fails**
- Check internet connection
- `data/cache/` must be writable
- nflverse CDN may have intermittent issues; retry

**Streamlit dashboard won't start**
- Verify streamlit is installed: `pip install streamlit>=1.38`
- Check port 8501 is available: `lsof -i :8501`

