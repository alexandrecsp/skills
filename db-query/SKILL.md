---
name: db-query
description: Query the project's local Postgres running in Docker through the command line, taking the connection settings from the docker-compose file. Use when the user asks to look at, query, inspect or check the local database, its tables, schema or data.
---

Answer the user's question by running SQL against the project's **local Postgres container**, reading its settings from the compose file instead of asking for them.

## Process

### 1. Find the connection

Look in the current directory for `docker-compose.yml`, `docker-compose.yaml`, `compose.yml` or `compose.yaml`. Read the Postgres service (image starting with `postgres`) and take:

- the **service name**
- `POSTGRES_USER` and `POSTGRES_DB` (the image defaults to `postgres` for both when unset)
- the host port from `ports` (`5433:5432` → 5433)

Resolve `${VAR}` references against the `.env` file next to the compose file.

Run `docker compose ps` to confirm the service is up. A stopped service is started only after the user agrees.

Done when service name, user and database are known and the container is running. With no compose file, or several Postgres services, ask the user which to use.

### 2. Learn the schema

Run `\dt` and `\d <table>` for the tables the question touches, so every query is written against the real column names.

### 3. Query

```
docker compose exec -T <service> psql -U <user> -d <db> -P pager=off -c "<SQL>"
```

Run it from the directory holding the compose file. Quote the SQL for the current shell (PowerShell and Bash differ). Add `LIMIT` to exploratory `SELECT`s.

Done when the result answers the user's question; report the answer with the SQL that produced it.

## Writes

`SELECT`, `\d` and `EXPLAIN` run freely. `INSERT`, `UPDATE`, `DELETE`, DDL and `TRUNCATE` run only after the user approves the exact statement, and a `DELETE` or `UPDATE` is first previewed as a `SELECT` with the same `WHERE`.
