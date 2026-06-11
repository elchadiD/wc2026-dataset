# METRICS.md

The derived layer — what separates this from a raw dump. Every formula, every threshold, every assumed proxy, stated. Source data: API-Football aggregates (xG where covered; no event coordinates).

## player_tournament (per player, cumulative)
- `goals_p90`, `assists_p90`, `ga_p90` = stat x 90 / minutes (minutes summed across the tournament)
- `key_passes_p90`, `def_actions_p90` = (tackles + interceptions) x 90 / minutes
- `duel_win_pct` = duels_won / duels_total x 100
- `dribble_success_pct` = dribbles_success / dribbles_attempts x 100
- `above_min_threshold` = minutes >= 90 (gate for leaderboards; shown so low-sample lines are flagged, not hidden)

## team_tournament (per team, cumulative)
- `avg_possession` = mean possession across matches
- `conversion_pct` = goals_for / shots_on_target x 100
- `shot_share` = team shots / (team shots + opponent shots) over the tournament
- `ppda` = opponent passes / (team tackles + interceptions + fouls), aggregating player actions to team level
- `xg_for` = sum of the team's match xG over the tournament (covered matches only; `NaN` if none covered)
- `xg_against` = sum of opponents' xG against the team (covered matches only)
- `finishing` = goals_for − xg_for. Positive = scoring above the quality of chances created (clinical / over-performing); negative = wasteful. The headline efficiency signal.
- `xg_diff` = xg_for − xg_against. Chance-quality dominance, often more predictive than scoreline or possession.

## Assumed limits (never hidden)
- xG is ingested from `/fixtures/statistics` **where API-Football covers the fixture**. It is coverage-dependent: `null` per match when absent, and tournament sums (`xg_for`, `xg_against`) span only the covered matches — so they are NOT directly comparable between teams with different coverage. Never back-filled or modelled by us.
- xG is read only on finished matches (it evolves live, starting 0–0), so partial in-play values never enter the data.
- Still locked: event coordinates (no shot maps / positional data this cycle); we never fake them.
- `ppda` is a full-pitch approximation; the canonical PPDA restricts to the opponent's defensive 60%. We can't, so we state it.
- Early-tournament rows may be incomplete (coverage varies); incomplete team rows are excluded from metrics via `is_complete` (which does not require xG).
- Minutes fixed at 90 for per-90 normalization (extra time ignored).