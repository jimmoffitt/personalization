## api_personalization Pipe

### stats 

```sql
SELECT
    name,
    uniq(session_id) AS n_games,
    countIf(type = 'score') AS scores,
    countIf(type = 'purchase') AS purchases
FROM events_api
WHERE timestamp >= now() - INTERVAL 1 HOUR
AND name = {{String(player_param, 'Frankie', description="Player name to filter on", required=True)}}
GROUP BY name
```

### offer

Using `if(x AND y AND z = 0,1,0)` pattern. 

```sql
SELECT if(n_games >= 3 AND scores <= 15 AND purchases = 0, 1, 0) AS offer
FROM stats
```


