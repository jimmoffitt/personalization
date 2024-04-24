
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
