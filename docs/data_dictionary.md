# Data Dictionary

Profiled in [`notebooks/02_eda.ipynb`](../notebooks/02_eda.ipynb) on 2026-09-25.
Source decision: [ADR-0004](decisiones/0004-fuentes-de-datos-revisadas.md).

## Sources overview

| Source | Role | Coverage | Format | Refresh |
|---|---|---|---|---|
| Sackmann archive (`Aneeshers/tennis-sackmann-archive`) | Primary historical source | 1968-01 → 2026-05-25 (Roland Garros 2026) | CSV, one file per year | Frozen (June 2026 snapshot) |
| tennis-data.co.uk | Odds + live results after the archive ends | 2000 → 2026-09-13 | `.xls` (≤2012), `.xlsx` (≥2013) | Weekly during the season |
| TML-Database | **Discarded** (stale since Jan 2026, broken files) | 1968 → 2026-01-17 | CSV | Stopped |

---

## 1. `atp_matches_YYYY.csv` (Sackmann archive) — main tour matches

**Grain:** one row per match, written from the winner's perspective.
**Primary key:** (`tourney_id`, `match_num`) — verified unique over 199,389 rows.

| Column | Type | Description | Quality notes |
|---|---|---|---|
| `tourney_id` | string | `YYYY-CODE`, e.g. `2025-520` | The **code** (`520` = Roland Garros, `540` Wimbledon, `560` US Open, `580` Australian Open) is the stable tournament identifier |
| `tourney_name` | string | Tournament name | **Inconsistent over time** (`US Open` vs `Us Open`) → never join on names |
| `surface` | string | `Hard`, `Clay`, `Grass`, `Carpet` | 2,990 nulls (Davis Cup, old events) |
| `draw_size` | int | Players in the draw | |
| `tourney_level` | string | `G` Slam · `M` Masters 1000 · `A` other ATP · `F` Finals · `D` Davis Cup · `O` Olympics | |
| `tourney_date` | int `YYYYMMDD` | **Tournament start date (Monday), not the match date** | All rounds share it → ordering needs `round` too |
| `match_num` | int | Match number within the tournament | **Not chronological** in 82 of 4,621 tournaments |
| `winner_id` / `loser_id` | int | Player id → `atp_players.player_id` | 100 % referential integrity |
| `winner_seed` / `loser_seed` | int | Seed | Mostly null (unseeded) |
| `winner_entry` / `loser_entry` | string | `Q` qualifier, `WC` wildcard, `LL` lucky loser, `PR` protected ranking, `SE` special exempt, `Alt` | Dirty casing (`Alt` / `ALT`) |
| `winner_name` / `loser_name` | string | Full name | |
| `winner_hand` / `loser_hand` | string | `R`, `L`, `U` unknown, `A` ambidextrous | |
| `winner_ht` / `loser_ht` | int | Height (cm) | 1–3 % null since 1990 |
| `winner_ioc` / `loser_ioc` | string | Country (IOC code) | |
| `winner_age` / `loser_age` | float | Age at tournament start | |
| `score` | string | e.g. `7-6(5) 6-4`, `6-3 2-1 RET`, `W/O` | 97.1 % regular; 2.0 % `RET`, 0.65 % `W/O`, 0.08 % `DEF`, 134 odd values (`UNK`, `NA`, `?`) |
| `best_of` | int | 3 or 5 | 36 rows with `1` |
| `round` | string | `R128`…`R16`, `QF`, `SF`, `F`, `RR` round robin, `BR` bronze, `ER` early rounds | |
| `minutes` | int | Match duration | Null before 1991 |
| `w_ace`, `w_df`, `w_svpt`, `w_1stIn`, `w_1stWon`, `w_2ndWon`, `w_SvGms`, `w_bpSaved`, `w_bpFaced` | int | Winner serve stats: aces, double faults, serve points, 1st serves in, 1st/2nd-serve points won, service games, break points saved/faced | **100 % null before 1991**, ~80 % available in the 1990s, >90 % since 2000 |
| `l_*` | int | Same stats for the loser | Same |
| `winner_rank` / `winner_rank_points` | int | ATP ranking at tournament start | Ranking exists since 1973 |
| `loser_rank` / `loser_rank_points` | int | Same for the loser | |

## 2. `atp_matches_qual_chall_YYYY.csv` (Sackmann archive)
Same schema as §1. Challenger (`C`) matches and qualifying rounds of bigger events.
~15,500 Challenger matches in 2025–26 (5× the main tour), with serve stats.

## 3. `atp_players.csv` (Sackmann archive)

**Grain:** one row per player. **Primary key:** `player_id` (66,912 rows).

| Column | Type | Description | Quality notes |
|---|---|---|---|
| `player_id` | int | Player id | Unique |
| `name_first`, `name_last` | string | Names | Some nulls |
| `hand` | string | `R`, `L`, `U`, `A` | |
| `dob` | int `YYYYMMDD` | Date of birth | 28 % null |
| `ioc` | string | Country | |
| `height` | int | cm | 94 % null (mostly low-level players) |
| `wikidata_id` | string | Wikidata reference | |

## 4. `atp_rankings_*.csv` (Sackmann archive)

**Grain:** one row per player per ranking week. **Primary key:** (`ranking_date`, `player`).

| Column | Type | Description |
|---|---|---|
| `ranking_date` | int `YYYYMMDD` | Ranking week (Monday) |
| `rank` | int | ATP rank |
| `player` | int | → `atp_players.player_id` |
| `points` | int | Ranking points |

Files per decade (`_70s` … `_20s`) plus `_current` (2026-01-05 → 2026-06-08).

## 5. tennis-data.co.uk yearly files (`YYYY.xlsx`)

**Grain:** one row per match. **No player ids.**

| Column | Description | Quality notes |
|---|---|---|
| `ATP` | Tournament number within the season | |
| `Location`, `Tournament` | Venue and name | Names differ from Sackmann's |
| `Date` | Match date | Read as text it's an **Excel serial number** (`45655` = 2024-12-29) → `DATE '1899-12-30' + n` |
| `Series` | ATP250, ATP500, Masters 1000, Grand Slam… | |
| `Court`, `Surface` | Indoor/Outdoor, surface | |
| `Round`, `Best of` | `1st Round`…`The Final` | Different vocabulary from Sackmann |
| `Winner`, `Loser` | `"Surname I."` (e.g. `"Nadal R."`) | Entity resolution needed; compound surnames and Asian name order break naive keys |
| `WRank`, `LRank`, `WPts`, `LPts` | Rank and points | |
| `W1`…`L5`, `Wsets`, `Lsets` | Games per set, sets won | |
| `Comment` | `Completed`, `Retired`, `Walkover` | |
| `B365W/L` | Bet365 odds | 0.5 % null; a few `'-'` and ≤ 1 values |
| `PSW/L` | Pinnacle odds | **97 % null in 2026** |
| `MaxW/L`, `AvgW/L` | Max and average market odds | Complete → main benchmark |
| `BFEW/L` | Betfair Exchange odds | |

Average bookmaker margin: Bet365 5.3 %, Pinnacle 2.7 %. The bookmaker favourite wins 65.5 % of matches.
