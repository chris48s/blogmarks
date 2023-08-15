<!--
.. title: Show Running Queries in PostgreSQL
.. slug: postgres-running-queries
.. date: 2020-06-20 00:00:00
.. tags: sql,sql
.. category: sql
.. link: 
.. description: 
.. type: text
-->

```sql
SELECT
    pid,
    (CURRENT_TIMESTAMP - query_start) as query_time,
    datname,
    usename,
    query
FROM pg_stat_activity
ORDER BY query_time DESC;
```

This is particularly useful for keeping an eye on long-running queries. Bonus - if you need to kill one, grab the PID and run:

```sql
SELECT pg_terminate_backend(12345);
```
