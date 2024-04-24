
## Data Sources

* events_api
  ```bash
  SCHEMA >
    `name` String `json:$.name`,
    `session_id` String `json:$.session_id`,
    `timestamp` DateTime64(3) `json:$.timestamp`,
    `type` LowCardinality(String) `json:$.type`,
    `event` String `json:$.event`

ENGINE "MergeTree"
 
* kafka_events

```bash
SCHEMA >
    `name` String `json:$.name`,
    `session_id` String `json:$.session_id`,
    `timestamp` DateTime64(3) `json:$.timestamp`,
    `type` LowCardinality(String) `json:$.type`,
    `event` String `json:$.event`

ENGINE "MergeTree"
```

* mv_player_stats

```bash
SCHEMA >
    `event` String,
    `player_id` String,
    `session_id` String,
    `scores` AggregateFunction(countIf, UInt8),
    `purchases` AggregateFunction(countIf, UInt8),
    `start_ts` AggregateFunction(min, DateTime64(3)),
    `end_ts` AggregateFunction(max, DateTime64(3))

ENGINE "AggregatingMergeTree"
```

## Pipes related to Materialized views

Here are the endpoints currently registered in the FlappyBird app:

* /pipes/api_leaderboard_mv.json
* /pipes/api_last_played_games_mv.json
* /pipes/api_player_stats_mv.json
* /pipes/api_personalization_mv.json

Note that all are related to Materialized Views.
They all pull from the `mv_player_stats` Data Source. 






## api_leaderboard_mv

`Returns a leaderboard with the top scores globally, in addition to each player's last game.`

### rank_games Node

`Select from the MV to rank each game by total score.`

```sql
 SELECT
    ROW_NUMBER() OVER (ORDER BY total_score DESC, t) AS rank,
    player_id,
    session_id,
    countMerge(scores) AS total_score,
    maxMerge(end_ts) AS t
FROM mv_player_stats
```

## api_last_played_games_mv

### endpoint Node
`Select from the MV and filter on a given player's last 10 games.`

```sql
SELECT session_id,
       countMerge(scores) AS total_score,
       toDateTime(maxMerge(end_ts)) AS t
FROM mv_player_stats
```

## api_player_stats_mv Pipe

`Select from the MV and filter on a given player's games.`

### agg Node

```sql
SELECT player_id, session_id, countMerge(scores) AS score
FROM mv_player_stats
```


## api_personalization_mv Pipe

```sql
SELECT
    player_id,
    session_id,
    countMerge(scores) AS score,
    countMerge(purchases) AS purchases,
    maxMerge(end_ts) AS t
FROM mv_player_stats
```
