---
title: pegged
---

**pegged** manages local PostgreSQL databases in Docker — the sqlrest development database tool. It is a thin shell over [go-pgdocker](https://gomatic.github.io/docs.go-pgdocker/); all lifecycle behavior lives in the library.

```console
$ pegged start 5498
$ psql "postgres://postgres@127.0.0.1:5498/postgres?sslmode=disable"
$ pegged snapshot create 5498 baseline   # stop first: snapshots are cold PGDATA clones
$ pegged stop 5498
```

## Commands

| Command | Does |
| --- | --- |
| `pegged start [port]` | Start an instance; waits for `pg_isready`; prints the instance with a ready DSN. Flags: `--image`, `--database`, `--user`, `--password`, `--snapshot`, `--volume reuse\|fresh`, `--retain keep\|remove`, `--listen`, `--init-sql`, `--platform`. |
| `pegged stop [port]` | Stop and apply volume retention (`--retain`, `--grace`). |
| `pegged inspect [port]` | One port's container and volumes. |
| `pegged list` | Every pegged-managed port. |
| `pegged snapshot create [port] [name]` | Cold PGDATA clone of the port's newest volume. |
| `pegged snapshot list` / `delete <name>` | Manage stored snapshots. |
| `pegged init [port] [--] [command...]` | Run an image/command against the running instance (migrations, seeds) in its network namespace; exit code passes through. |

## The port policy (pegged's own)

The host port is the instance's identity. The policy below is pegged's product convention — the underlying library defaults to fresh volumes and keep-on-stop with no port opinion; pegged resolves the policy at start time into explicit choices the container records. **At or below 5432** an instance is durable: `start` reuses the newest existing volume and `stop` keeps it. **Above 5432** it is ephemeral: fresh volume on start, removed on stop. Override per call with `--volume` and `--retain`; the choice made at start is recorded on the container and honored by `stop`.

## Defaults are development-easy

Trust auth (no password) unless `--password` is set; loopback-only binding unless `--listen` widens it; `postgres:17-alpine`; tuned dev server settings via `postgres -c` flags — no config files anywhere. Port resolution: positional argument, then `PGPORT`, then 5432 (a conflicting argument and `PGPORT` is an error, never a silent pick).

## Snapshots — major-version bound

Snapshots are physical PGDATA clones taken while the database is stopped. They restore only onto the same PostgreSQL major; `pegged start --snapshot <name>` refuses a mismatch unless forced. Restores clone the snapshot — the stored snapshot itself is never mounted.

## Environment

Every flag reads `PEGGED_<FLAG>`; `--port` also honors `PGPORT`. All containers and volumes carry `pegged.*` labels — discovery is label-based, so pegged coexists with other tools (and other go-pgdocker namespaces) on the same daemon.
