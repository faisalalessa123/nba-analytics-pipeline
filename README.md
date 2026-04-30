# NBA Analytics Pipeline

An end-to-end data engineering project that pulls live NBA statistics directly 
from NBA.com, transforms and normalizes the data into a relational SQLite 
database, and computes player value scores to identify undervalued and 
overvalued players relative to their salary.

---

## What This Project Does

Most NBA analytics projects start with a pre-cleaned Kaggle dataset. This one 
pulls raw data directly from the NBA.com stats API, handles all the messy 
real-world ETL challenges that come with it — trade duplicates, mismatched 
player names across sources, inconsistent column naming — and loads everything 
into a properly normalized relational database. The end result is a clean, 
queryable warehouse that supports both descriptive analytics and player 
valuation analysis.

---

## Project Structure
```
nba-analytics-pipeline/
├── Data/
│   ├── player_traditional.csv
│   ├── player_advanced.csv
│   ├── player_defense.csv
│   ├── player_hustle.csv
│   ├── player_shooting.csv
│   ├── player_bios.csv
│   ├── player_salary.csv
│   ├── team_stats.csv
│   ├── playoff_traditional.csv
│   ├── playoff_advanced.csv
│   ├── playoff_defense.csv
│   └── playoff_team.csv
├── nba_warehouse.db            — SQLite database (11 tables, 3NF schema)
├── NBA_ETL.ipynb               — ETL pipeline, database build, and value scoring
├── NBA_Analysis.ipynb          — Analysis playground (no setup required)
└── README.md
```

---

## Data

All statistics were pulled directly from NBA.com using the 
[nba_api](https://github.com/swar/nba_api) Python library. No API key required. 
Salary data was sourced from Kaggle. Data covers the **2021-22 through 2024-25** 
regular seasons and playoffs. COVID-affected seasons (2019-20, 2020-21) were 
excluded due to shortened schedules that skew per-game averages.

**Data sources:**
- Traditional stats — points, assists, rebounds, shooting splits
- Advanced metrics — Net Rating, True Shooting %, Player Impact Estimate, Usage Rate
- Defensive stats — Defensive Win Shares, opponent paint points, second chance points
- Hustle stats — deflections, contested shots, charges drawn, screen assists, box outs
- Shooting splits — 2PT vs 3PT attempt rates and efficiency
- Player bios — position, height, weight, college, country, draft info
- Salary data — season-level contract values

---

## Database Schema

8-table normalized relational schema following 3NF principles:

| Table | Type | Description |
|---|---|---|
| Player | Dimension | Static bio info — position, height, draft year, country |
| Team | Dimension | Team name and abbreviation |
| Season | Dimension | Season labels (e.g. 2023-24) |
| PlayerSeasonStats | Fact | All player metrics merged into one row per player per season |
| TeamSeasonStats | Fact | Team-level ratings, win percentage, pace |
| PlayerSalary | Fact | Season salary matched to PLAYER_ID via fuzzy name matching |
| PlayoffPlayerStats | Fact | Same structure as PlayerSeasonStats for playoff performance |
| PlayoffTeamStats | Fact | Same structure as TeamSeasonStats for playoff performance |

**Key ETL decisions:**
- Trade duplicates handled by keeping the stint with the most games played
- Salary data linked to player IDs via fuzzy string matching using `thefuzz`
- All rank columns dropped — only meaningful metrics retained
- COVID seasons excluded as an intentional data quality decision

---

## Player Value Analysis

Three individual value scores and one composite score are computed to rank 
players by efficiency relative to salary:

- **PIE Value Score** — Player Impact Estimate / Salary (millions)
- **Net Rating Value Score** — Net Rating / Salary (millions)
- **True Shooting Value Score** — True Shooting % / Salary (millions)
- **Composite Value Score** — Equal-weighted average of all three (normalized within season)

Players are bucketed into **Undervalued**, **Fair Value**, and **Overvalued** 
tiers using 25th/75th percentile thresholds within each season. Normalizing 
within season ensures rankings reflect performance relative to that year's 
league rather than being distorted by year-over-year changes in league averages.

---

## How to Run

### Analysis Notebook (recommended — no setup required)
Open directly in Google Colab. No Drive, no installation, no credentials needed.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/faisalalessa123/nba-analytics-pipeline/blob/main/NBA_Analysis.ipynb)

### ETL Notebook (internal use only)
Rebuilds the database from scratch using the NBA API and Google Drive. Requires personal Drive access — not intended for public use.

1. Mount your Google Drive in Colab
2. Update the file paths to match your Drive folder
3. Run all cells top to bottom

### Local
git clone https://github.com/faisalalessa123/nba-analytics-pipeline.git
cd nba-analytics-pipeline
pip install pandas numpy sqlite3 thefuzz python-Levenshtein matplotlib
jupyter notebook NBA_Analysis.ipynbnotebook NBA_ETL.ipynb
```

---

## Tech Stack
- **Python** — pandas, numpy, sqlite3, thefuzz
- **nba_api** — live data extraction from NBA.com
- **SQLite** — relational database
- **Google Colab** — development environment
