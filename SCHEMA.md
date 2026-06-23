# SCHEMA.md

`SCHEMA_VERSION = 0.2.0`. Any column change bumps this and is noted in the changelog below — never silently. Source: API-Football. Missing fields are `null` + flagged, never invented.

## matches (1 row per match)
`fixture_id`, `date`, `status`, `round`, `venue`, `city`, `home_team_id`, `home_team`, `away_team_id`, `away_team`, `goals_home`, `goals_away`.

## team_match (2 rows per match)
Keys: `fixture_id`, `team_id`, `team_name`, `opponent_id`, `is_home`, `goals_for`, `goals_against`, `outcome` (W/D/L).
Stats (from `/fixtures/statistics`): `possession_pct`, `passes_total`, `passes_accurate`, `passes_pct`, `shots_total`, `shots_on_target`, `shots_off_target`, `shots_blocked`, `shots_inside_box`, `shots_outside_box`, `fouls`, `corners`, `offsides`, `yellow_cards`, `red_cards`, `gk_saves`, `xg`.
- `xg`: expected goals for the team in this match. **Coverage-dependent** — `null` when API-Football does not provide it for the fixture. NOT part of `is_complete` (a match with full stats but no xG is still complete). Read only on finished matches, since live xG evolves and starts at 0–0.
`is_complete`: true when possession, passes, shots, fouls are all present (required for metrics). Does not require `xg`.

## player_match (1 row per player per match)
Keys: `fixture_id`, `team_id`, `player_id`, `player_name`, `position`, `is_starter`.
Stats (from `/fixtures/players`): `minutes`, `rating`, `goals`, `assists`, `shots_total`, `shots_on`, `passes_total`, `passes_key`, `passes_accuracy`, `tackles`, `interceptions`, `duels_total`, `duels_won`, `dribbles_attempts`, `dribbles_success`, `fouls_drawn`, `fouls_committed`, `yellow`, `red`.
No per-player xG: API-Football does not expose it at player level.

## events (1 row per event)
`fixture_id`, `minute`, `minute_extra`, `team_id`, `player_id`, `player_name`, `assist_id`, `type` (Goal/Card/subst/Var), `detail`.

## Derived (see METRICS.md)
`player_tournament`, `team_tournament` — cumulative, recomputed each matchday, written under `data/_latest/`.
`team_tournament` includes the xG-derived columns `xg_for`, `xg_against`, `finishing`, `xg_diff` (coverage-dependent, `NaN` when no covered match).

## Per-matchday snapshots (see METRICS.md)
`player_tournament`, `team_tournament` — same schema as the cumulative tables, but frozen to a single matchday's fixtures, written under `data/by_matchday/matchday_NN/`. Each row carries a `matchday` column.

## Index (lookup)
`fixtures_index.csv` / `.parquet` — one row per match (`fixture_id`, `label`, `date`, `round`, `home_team`, `away_team`), sorted by date, written under `data/_index/`. Use it to quickly find a match and its `fixture_id`.

## Changelog
- **0.4.0** — Added `data/_index/fixtures_index.csv`, a single lookup file listing every match (fixture_id, label, date, round) so you can quickly find a match and its fixture_id.
- **0.3.0** — Added per-matchday snapshots (data/by_matchday/matchday_NN/) that freeze team_tournament and player_tournament to each matchday's fixtures, with a matchday column.
- **0.2.0** — added `xg` to `team_match`; added `xg_for`, `xg_against`, `finishing`, `xg_diff` to `team_tournament`. xG is now ingested from `/fixtures/statistics` where coverage provides it.
- **0.1.0** — initial schema.