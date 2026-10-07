# Simulating 82-0

Can you build a team that goes **82-0**? This project recreates the 82-0 NBA roster-building game using 66 years of real player stats, then simulates **100,000 rosters** to see how often a perfect season is possible and which players make it happen.

## How the game works
You build a five-man lineup (PG, SG, SF, PF, C). Each pick comes from a randomly drawn **team + decade** (for example, *Golden State Warriors, 1970s*), and you take one player from that team's roster in that decade to fill an open position. A team–decade combo can't be used twice. Your final lineup's strength determines your projected wins.

## Key findings (100,000 simulations)

| Metric | Result |
|---|---|
| Average projected wins | **67.6** |
| Best roster | **92.0** wins |
| Rosters reaching 82 wins | **0.72%** |
| 82-win rosters with Wilt Chamberlain at C | **81%** |

A perfect season is rare, and it almost always runs through Wilt.

<!-- TODO: add a histogram of simulated wins and a table of the most common players in 82-win rosters -->

## Methodology

**1. Data collection.** A Python scraper (requests + BeautifulSoup) pulls regular-season totals for every player, team, and season from Basketball-Reference for 1960–2026. It handles rate limits with exponential backoff and can resume an interrupted run.

**2. Cleaning and aggregation.**
- Relocated and renamed franchises are mapped to their current team (e.g., Seattle SuperSonics → Oklahoma City Thunder, Kansas City Kings → Sacramento Kings).
- Stats are aggregated to **per-game averages by player, team, and decade**.
- **Position eligibility** is minute-weighted from Basketball-Reference play-by-play position data where available, and falls back to listed positions for older seasons.

**3. Player value.** Each player–team–decade gets a weighted box-score value:

```
Value = 0.34·PPG + 0.59·RPG + 0.63·APG + 1.29·SPG + 1.55·BPG
```

Steals and blocks weren't tracked until 1973–74, so players from the 1960s and 1970s receive a flat +3.4 adjustment in place of those terms.

**4. Projected wins.** A roster's total value is converted to wins:

```
Wins = 0.83 · (Total Value) − 9.6
```
<!-- TODO: one sentence on how this mapping was derived (e.g., fit on team results 2014–2025, the game_results_*.rds files) -->

**5. Simulation.** For each of 100,000 runs, the simulator draws random team–decade combos and greedily takes the highest-value player eligible for an open position until all five spots are filled. Results are saved to `simulations-100k.csv` and analyzed by position, player, and win total.

## Files

| File | Description |
|---|---|
| `82-0.ipynb` | Scraper, cleaning, value metric, simulation, and analysis |
| `nba_player_stats_raw.csv` | Raw scraped player–team–season stats |
| `nba_player_stats_by_decade.csv` | Per-game averages and position percentages by player, team, and decade |
| `high-value-players.csv` | Player–decade entries with Value ≥ 17 |
| `simulations-100k.csv` | All 100,000 simulated rosters with values and projected wins |
| `decade-aggregations.csv`, `team-aggregations.csv`, `position-counts.csv` | Summary tables of high-value players by decade, team, and position |
| `game_results_*.rds` | Team game results, 2014–2025 |

## Running it

```bash
pip install pandas numpy requests beautifulsoup4 matplotlib
jupyter notebook 82-0.ipynb
```

Scraping Basketball-Reference takes a while because of its rate limits. To skip that step, load `nba_player_stats_raw.csv` directly.

## Author
**Zack Calvert** · [LinkedIn](https://linkedin.com/in/zack-calvert) · [Substack](https://zcalvert.substack.com)

*Data from [Basketball-Reference](https://www.basketball-reference.com).*
