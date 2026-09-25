# 🎾 Tennis Analytics Platform

End-to-end data engineering project on professional tennis data: from raw CSVs to a lakehouse,
Elo ratings, a match-outcome model benchmarked against bookmakers, and Monte Carlo simulation
of Grand Slams.

> 🚧 **Work in progress** — Phase 1 under construction.

## What it answers

- **Who is really the best?** Elo ratings per surface (clay, grass, hard) for every ATP player since 1968.
- **Head-to-head** between any two players, overall and by surface.
- **Can a model beat the bookmakers?** Match predictor evaluated against betting-market odds.
- **Who wins the next Grand Slam?** 100k Monte Carlo simulations of the full draw.

## Architecture (Phase 1)

```
TML-Database / tennis-data.co.uk
        │  Python ingestion (orchestrated by Airflow, Docker)
        ▼
Databricks Volume (raw) ──Auto Loader──► BRONZE ──PySpark──► SILVER ──dbt──► GOLD
                                                                         │
                     MLflow model · Monte Carlo on Spark · serving export
                                                                         ▼
                                                        Streamlit public app
```

## Tech stack

| Layer | Tools |
|---|---|
| Ingestion | Python, `databricks-sdk` |
| Storage & compute | Databricks (Unity Catalog, Delta Lake, Auto Loader, serverless) |
| Transformation | PySpark, dbt (`dbt-databricks`) |
| ML | LightGBM, MLflow, Unity Catalog Model Registry |
| Orchestration | Apache Airflow (Docker) |
| Infra as code | Databricks Asset Bundles |
| Quality | pytest, dbt tests, ruff, pre-commit, gitleaks |
| CI | GitHub Actions |
| Serving | Streamlit |

## Repository layout

```
src/tennis_platform/   Python package (ingestion, Elo, ML, simulation)
tests/                 Unit tests
docs/                  Architecture, data dictionary, decision records (ADRs)
```

More folders (`databricks/`, `dbt/`, `airflow/`, `app/`) are added as each block is built.

## Getting started

Requirements: [uv](https://docs.astral.sh/uv/), Docker, a Databricks workspace (Free Edition works).

```bash
uv sync                          # create .venv and install dependencies
cp .env.example .env             # then fill in your Databricks host and token
uv run pre-commit install        # enable git hooks
uv run pytest                    # run tests
```

## Data & attribution

Tennis data by **Jeff Sackmann** and the **TML-Database** project, licensed under
[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Betting odds from
[tennis-data.co.uk](http://www.tennis-data.co.uk/). Data is downloaded at runtime and is not
stored in this repository. Non-commercial use only.

## License

Code: [MIT](LICENSE).
