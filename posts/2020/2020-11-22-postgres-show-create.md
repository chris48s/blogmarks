<!--
.. title: Show table create statement in Postgres
.. slug: postgres-show-create
.. date: 2020-11-22 00:00:00
.. tags: sql,sql
.. category: sql
.. link: 
.. description: 
.. type: text
-->

MySQL has a handy statement `SHOW CREATE TABLE my-table;`. Postgres doesn't have this but it can be achieved with `pg_dump` and the `--schema-only` flag.

```bash
pg_dump -t 'public.my-table' --schema-only my-database
```
