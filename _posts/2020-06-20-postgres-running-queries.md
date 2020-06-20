---
layout: post
title: Show Running Queries in PostgreSQL
categories: ['sql']
tags: ['sql']
---

## Show Running Queries in PostgreSQL

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
