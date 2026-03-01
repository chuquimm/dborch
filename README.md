# dborch

![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?logo=mysql&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

A single stack to manage all your development databases.

## Why

Every project needs a database. Some need two. Before long you're juggling multiple MySQL and PostgreSQL instances, each with its own port, its own credentials, its own way of starting up. You remember the `mysqldump` flags for one project but not the `pg_dump` flags for another. Pulling production data to debug a local issue means cobbling together SSH tunnels, dump commands, and restore scripts from memory every time.

**dborch** started as a way to consolidate all of that into a single Docker Compose stack — one `docker compose up` instead of five. But the real goal is broader: a tool that makes database management across projects feel simple and predictable, whether you're spinning up a local instance, backing up before a migration, or pulling production data to reproduce a bug.

Today, dborch is the foundation — a well-structured Compose stack with health checks, structured labels, and sensible defaults. The CLI and automation tooling will come next, built on top of this base.

## Services

| Service | Image | Port | Data |
|---------|-------|------|------|
| `mysql8-1` | `mysql:8.1` | `127.0.0.1:3306` | `.db-data/mysql8/` |
| `mysql5-6` | `mysql:5.6` | `127.0.0.1:3307` | `.db-data/mysql56/` |
| `pg17` | `postgres:17` | `127.0.0.1:5432` | `.db-data/pg17/` |
| `mongo8` | `mongo:8.0` | `127.0.0.1:27017` | `.db-data/mongo8/` |

All services bind to `127.0.0.1` only — not exposed to the local network.

## Quick start

```bash
# Clone
git clone https://github.com/chuquimm/dborch.git
cd dborch

# Configure
cp .env.example .env
openssl rand -base64 756 > .mongo-keyfile
chmod 400 .mongo-keyfile

# Start
docker compose up -d

# Verify
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

## Connecting

### From Docker containers

Applications running inside Docker connect via `host.docker.internal`:

```yaml
# In your project's docker-compose.yml
services:
  my_app:
    environment:
      DB_HOST: host.docker.internal
      DB_PORT: 3306  # or 3307, 5432
```

### From the host

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p       # MySQL 8.1
mysql -h 127.0.0.1 -P 3307 -u root -p       # MySQL 5.6
psql -h 127.0.0.1 -p 5432 -U postgres       # PostgreSQL 17
mongosh -h 127.0.0.1 -u admin -p            # MongoDB 8.0
```

## Configuration

Copy `.env.example` to `.env` and adjust as needed:

| Variable | Default | Description |
|----------|---------|-------------|
| `MYSQL_ROOT_PASSWORD` | `root` | Root password for MySQL instances |
| `POSTGRES_PASSWORD` | `postgres` | Superuser password for PostgreSQL |
| `MONGO_ROOT_PASSWORD` | `mongo` | Root password for MongoDB |

## Customization

### Project metadata

Create a `docker-compose.override.yml` (gitignored) to track which projects use which services:

```yaml
services:
  mysql8-1:
    labels:
      com.dborch.projects: "my-app, my-api"
      com.dborch.notes: "databases: my_app_dev, my_api_dev"
```

Docker Compose merges this automatically with the base file.

### Adding services

Extend the stack with additional database engines by adding services to `docker-compose.yml` or via the override file. Follow the existing pattern: bind to `127.0.0.1`, add a health check, include `com.dborch.*` labels, and mount data under `.db-data/`.

## Labels

All services use structured labels under `com.dborch.*` for filtering and identification:

```bash
docker ps --filter "label=com.dborch.service"        # All dborch services
docker ps --filter "label=com.dborch.engine=mysql"   # Filter by engine
```

| Label | Description |
|-------|-------------|
| `com.dborch.service` | Service name |
| `com.dborch.engine` | Database engine (`mysql`, `postgresql`) |
| `com.dborch.version` | Engine version |
| `com.dborch.port` | Exposed host port |
| `com.dborch.description` | Human-readable description |

## Features

- **Auto-start** with Docker Desktop (`restart: unless-stopped`)
- **Health checks** on all services
- **Log rotation** (10MB x 3 files per service)
- **Graceful shutdown** (60s grace period)
- **Localhost-only** binding
- **Bind-mount volumes** for direct data access

## Data persistence

All data is persisted via bind mounts in `.db-data/`:

```
.db-data/
├── mysql8/    # MySQL 8.1
├── mysql56/   # MySQL 5.6
├── pg17/      # PostgreSQL 17
└── mongo8/    # MongoDB 8.0
```

> **Warning**: `.db-data/` is gitignored. Back up your data separately.

## Troubleshooting

### Port already in use

```
Error: Bind for 127.0.0.1:3306 failed: port is already allocated
```

Find and stop the conflicting process:

```bash
lsof -i :3306
```

### `host.docker.internal` not resolving (Linux)

`host.docker.internal` works out of the box on Docker Desktop (Mac/Windows). On native Linux Docker Engine, add this to your project's service:

```yaml
services:
  my_app:
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

### Container exits immediately

Check if `.env` exists and inspect the logs:

```bash
ls -la .env              # If missing: cp .env.example .env
docker logs mysql8-1     # Check for errors
```

### Data lost after restart

Data survives `docker compose down` + `docker compose up`. It is **only deleted** if you explicitly remove volumes:

```bash
docker compose down -v   # DELETES all data
docker compose down      # Safe — stops containers, keeps data
```

### Slow performance on macOS

Bind mounts on macOS Docker Desktop can be slower than native. Allocate more resources in **Docker Desktop > Settings > Resources**.

## What's next

dborch is heading toward becoming a CLI tool that wraps around this Compose stack. The direction includes:

- **A `dborch` command** to manage services, create databases, and open interactive clients — without remembering engine-specific syntax.
- **Backup and restore automation** — dump and restore databases with a single command, organized by project and date.
- **Multi-environment support** — define connection profiles for dev, staging, and production, and pull remote data to local for debugging.
- **A console dashboard** — live view of running services, health, disk usage, and project associations.
- **More engines** — MongoDB and Redis support alongside MySQL and PostgreSQL.

## License

[MIT](LICENSE)
