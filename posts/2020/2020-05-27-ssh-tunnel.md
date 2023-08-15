<!--
.. title: Creating SSH Tunnels
.. slug: ssh-tunnel
.. date: 2020-05-27 00:00:00
.. tags: terminal,ssh,terminal
.. category: terminal
.. link: 
.. description: 
.. type: text
-->

Two handy examples:

1) Bind local port 4001 to port 5432 on `postgres-server`:

```bash
ssh user@postgres-server -L 4001:localhost:5432
psql -h 127.0.0.1 -p 4001
```

2) Bind local port 4001 to port 5432 on `postgres-server` using `sentinel` as a jump box:

```bash
ssh user@sentinel -L 4001:postgres-server:5432
psql -h 127.0.0.1 -p 4001
```
