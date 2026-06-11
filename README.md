# World Cup 2026 — Football Dataset

Clean, documented match data for the 2026 FIFA World Cup, built from API-Football.

Grains: matches, team_match, player_match, events. Cumulative tables under `data/_latest/`
(player_tournament, team_tournament). Pre-match predictions under `data/predictions/`.
Updated after each matchday. Column dictionary in SCHEMA.md, formulas in METRICS.md.
SCHEMA_VERSION = 0.2.0.

Load: `pd.read_parquet("data/_latest/team_tournament.parquet")` (or the .csv).
Missing values are null, never invented. `xg` is coverage-dependent (can be null).
Source: API-Football.
