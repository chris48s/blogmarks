---
layout: post
title: Creating SSH Tunnels
categories: ['terminal']
tags: ['terminal', 'ssh']
---

## Creating SSH Tunnels

Two handy examples:

1. Bind local port 4001 to port 5432 on `postgres-server`:

    ```
    ssh user@postgres-server -L 4001:localhost:5432
    psql -h 127.0.0.1 -p 4001
    ```

2. Bind local port 4001 to port 5432 on `postgres-server` using `sentinel` as a jump box:

    ```sh
    ssh user@sentinel -L 4001:postgres-server:5432
    psql -h 127.0.0.1 -p 4001
    ```
