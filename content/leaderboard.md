
### filter_data 
```sql
SELECT name AS player_id,
       session_id
FROM events_api
WHERE type = 'score'
```

### aggregrate
```sql
SELECT player_id, session_id, count() AS score
FROM filter_data
GROUP BY player_id, session_id
ORDER BY score DESC
LIMIT 10
```


# Materialized View details

### rank_games

```sql
SELECT
 ROW_NUMBER() OVER (ORDER BY total_score DESC, t) AS rank,
 player_id,
 session_id,
 countMerge(scores) AS total_score,
 maxMerge(end_ts) AS t
FROM mv_player_stats
WHERE
 1
 {% if defined(event_param) %}
     AND event = {{ String(event_param, 'Austin Data Council', description="Event to filter on") }}
 {% end %}
GROUP BY player_id, session_id
ORDER BY rank
```

### last_game

```sql
SELECT
 argMax(rank, t) AS rank,
 player_id,
 argMax(session_id, t) AS session_id,
 argMax(total_score, t) AS total_score
FROM rank_games
WHERE
 player_id
 = {{ String(player_param, 'player1', description="Player to filter on", required=True) }}
GROUP BY player_id
```

### endpoint

```sql
SELECT *
FROM
 (
     SELECT rank, player_id, session_id, total_score
     FROM rank_games
     WHERE
         (player_id, session_id)
         NOT IN (SELECT player_id, session_id FROM last_game)
     LIMIT 9
     UNION ALL
     SELECT rank, player_id, session_id, total_score
     FROM last_game
 )
ORDER BY rank
```

