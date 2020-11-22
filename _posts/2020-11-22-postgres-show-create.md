---
layout: post
title: Show table create statement in Postgres
categories: ['sql']
tags: ['sql']
---

## Show table create statement in Postgres

MySQL has a handy statement `SHOW CREATE TABLE my-table;`. Postgres doesn't have this but it can be achieved with `pg_dump` and the `--schema-only` flag.

```bash
pg_dump -t 'public.my-table' --schema-only my-database
```
