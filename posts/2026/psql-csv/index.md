<!--
.. title: Exporting a CSV from Postgres
.. slug: psql-csv
.. date: 2026-07-31 00:00:00
.. tags: terminal,sql
.. category: 
.. link: 
.. description: 
.. type: text
-->

There are many ways to run a SQL statement in psql and export the result as a CSV. These are the two I usually reach for.

From a postgres repl:

```
\copy (SELECT * FROM users) 
TO 'users.csv' 
WITH (FORMAT csv, HEADER true);
```

Shell one-liner:

```bash
psql --csv -c "SELECT * FROM users;" --host 127.0.0.1 --port 5432 --user fred --dbname my_db > users.csv
```
