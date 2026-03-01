# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

dborch is a database orchestrator for local development. Today it's a Docker Compose stack that consolidates MySQL, PostgreSQL, and MongoDB instances into a single setup. It's evolving into a Python CLI tool for managing databases across projects and environments.

See `README.md` for vision and current capabilities. See `docs/commits.md` for commit conventions.

## Commands

```bash
docker compose up -d          # Start all services
docker compose down           # Stop all services (keeps data)
docker compose ps             # List running services
docker compose logs -f <svc>  # Follow logs for a service
```

Service names: `mysql8-1`, `mysql5-6`, `pg17`, `mongo8`.

## Architecture

The stack runs four database services behind Docker Compose, all bound to `127.0.0.1`:

- **mysql8-1** (MySQL 8.1) on port 3306 — primary MySQL, uses `mysql_native_password`
- **mysql5-6** (MySQL 5.6) on port 3307 — legacy, runs via `linux/amd64` on Apple Silicon
- **pg17** (PostgreSQL 17) on port 5432 — shared memory set to 128mb
- **mongo8** (MongoDB 8.0) on port 27017 — single-node replica set, auth user `admin`, requires `.mongo-keyfile`

Data lives in `.db-data/<engine>/` (bind mounts, gitignored). SQL dumps go in `dumps/` (also gitignored). Environment passwords are in `.env` (gitignored), templated from `.env.example`.

Services use `com.dborch.*` labels for identification: `service`, `engine`, `version`, `port`, `description`. Project-to-database mappings live in `docker-compose.override.yml` (gitignored, local only).

## Conventions

- **Commits:** Conventional Commits format — `<type>(<scope>): <subject>` with optional narrative body explaining *why*. Types: `feat`, `fix`, `docs`, `chore`, `refactor`. Scopes: `mysql`, `pg`, `mongo`, `docs`, `config`. Subject max 50 chars, imperative mood, lowercase. Body wraps at 72 chars.
- **Never** add `Co-Authored-By` trailers or AI attribution to commits.
- **Documentation** is written in English, technical-narrative tone — explain the why, be concise, stay coherent.
- **YAML** uses 2-space indentation (see `.editorconfig`).
- All services bind to `127.0.0.1`, never `0.0.0.0`.
- `docker compose down -v` destroys data — never run it without explicit user confirmation.
