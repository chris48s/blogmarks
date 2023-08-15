<!--
.. title: Identifying poorly indexed tables in Postgres
.. slug: postgres-indexes
.. date: 2021-03-04 00:00:00
.. tags: sql,sql
.. category: sql
.. link: 
.. description: 
.. type: text
-->

Out of the box the Postgres [statistics collector](https://www.postgresql.org/docs/current/monitoring-stats.html) gives us a variety of useful views we can query to understand the usage and performance of a database. This is a huge topic, so I'm just going to look at one very constrained topic in this post.

Sequential scans are OK on small tables or if we are making them infrequently, but usually if we are performing a lot of sequential scans on large tables this is a sign that we need to create an index. Fortunately Postgres collects some data in `pg_stat_user_tables` that can help us identify this:

* Looking at `seq_scan` and `idx_scan` can show us tables where we are frequently performing sequential scans rather than indexed scans
* Looking at `seq_scan` and `seq_tup_read` can show us tables where sequential scans typically process a large number of rows

Running this query:

```sql
SELECT
    relname, idx_scan, seq_scan, seq_tup_read,

    seq_tup_read / seq_scan
    AS average_tuples_per_scan,

    ((seq_scan / (seq_scan + idx_scan) :: FLOAT) * 100)
    AS percent_sequential_scans
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY average_tuples_per_scan DESC;
```

Should provide a pretty clear picture of tables that need optimisation.
