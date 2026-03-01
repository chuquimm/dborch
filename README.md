# dborch

![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?logo=mysql&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

Centralized database orchestrator for local development. One `docker compose up` to rule them all.

## Why

Running separate MySQL/PostgreSQL instances per project wastes resources and creates port conflicts. **dborch** consolidates all database services into a single compose stack that starts automatically with Docker Desktop.

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Mac, Windows, or Linux)
- Docker Compose V2 (included with Docker Desktop)

> **Apple Silicon (M1/M2/M3/M4)**: Fully supported. The `mysql5-6` service uses `platform: linux/amd64` and runs via Rosetta emulation automatically.

## Services

| Service | Image | Port | Data |
|---------|-------|------|------|
| `mysql8-1` | `mysql:8.1` | `127.0.0.1:3306` | `.db-data/mysql8/` |
| `mysql5-6` | `mysql:5.6` | `127.0.0.1:3307` | `.db-data/mysql56/` |
| `pg17` | `postgres:17` | `127.0.0.1:5432` | `.db-data/pg17/` |

All services bind to `127.0.0.1` only (not exposed to the local network).

## Quick Start

```bash
# 1. Clone
git clone https://github.com/YOUR_USER/dborch.git
cd dborch

# 2. Configure
cp .env.example .env

# 3. Start
docker compose up -d

# 4. Verify
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

## Connecting from Docker containers

Applications running inside Docker containers connect via `host.docker.internal`:

```yaml
# In your project's docker-compose.yml
services:
  my_app:
    environment:
      DB_HOST: host.docker.internal
      DB_PORT: 3306  # or 3307, 5432
```

## Connecting from the host

Connect directly to `127.0.0.1:PORT`:

```bash
# MySQL 8.1
mysql -h 127.0.0.1 -P 3306 -u root -p

# MySQL 5.6
mysql -h 127.0.0.1 -P 3307 -u root -p

# PostgreSQL 17
psql -h 127.0.0.1 -p 5432 -U postgres
```

## Configuration

Copy `.env.example` to `.env` and adjust as needed:

| Variable | Default | Description |
|----------|---------|-------------|
| `MYSQL_ROOT_PASSWORD` | `root` | Root password for MySQL instances |
| `POSTGRES_PASSWORD` | `postgres` | Superuser password for PostgreSQL |

## Customization

### Adding project-specific metadata

Create a `docker-compose.override.yml` (gitignored) to add labels or settings without modifying the base file:

```yaml
services:
  mysql8-1:
    labels:
      com.dborch.projects: "my-app, my-api"
      com.dborch.notes: "databases: my_app_dev, my_api_dev"
```

Docker Compose merges this automatically.

### Adding new services

Add more database engines by extending `docker-compose.yml` or via the override file:

```yaml
services:
  mongo7:
    image: mongo:7
    container_name: mongo7
    restart: unless-stopped
    ports:
      - "127.0.0.1:27017:27017"
    volumes:
      - ./.db-data/mongo7:/data/db
    labels:
      com.dborch.service: "mongo7"
      com.dborch.engine: "mongodb"
      com.dborch.version: "7"
```

## Labels

All services use structured labels under `com.dborch.*` for filtering and identification:

```bash
# List all dborch services
docker ps --filter "label=com.dborch.service"

# Filter by engine
docker ps --filter "label=com.dborch.engine=mysql"
```

| Label | Description |
|-------|-------------|
| `com.dborch.service` | Service name |
| `com.dborch.engine` | Database engine (mysql, postgresql, mongodb...) |
| `com.dborch.version` | Engine version |
| `com.dborch.port` | Exposed host port |
| `com.dborch.description` | Human-readable description |

## Features

- **Auto-start** with Docker Desktop (`restart: unless-stopped`)
- **Health checks** on all services
- **Log rotation** (10MB x 3 files per service)
- **Graceful shutdown** (60s grace period)
- **Localhost-only** binding (not exposed to the network)
- **Bind-mount volumes** for direct data access

## Data Persistence

All data is persisted via bind mounts in `.db-data/`:

```
.db-data/
├── mysql8/    # MySQL 8.1 data files
├── mysql56/   # MySQL 5.6 data files
└── pg17/      # PostgreSQL 17 data files
```

> **Warning**: `.db-data/` is gitignored. Back up your data separately.

## Troubleshooting

### Port already in use

```
Error: Bind for 127.0.0.1:3306 failed: port is already allocated
```

Another process is using the port. Find and stop it:

```bash
# macOS/Linux
lsof -i :3306

# Then stop the conflicting process or change the port in docker-compose.yml
```

### `host.docker.internal` not resolving (Linux)

`host.docker.internal` works out of the box on **Docker Desktop** (Mac/Windows). On native Linux Docker Engine, add this to your project's service:

```yaml
services:
  my_app:
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

### Container exits immediately

Check if `.env` file exists:

```bash
ls -la .env
# If missing:
cp .env.example .env
```

Check container logs for details:

```bash
docker logs mysql8-1
docker logs mysql5-6
docker logs pg17
```

### Data lost after restart

Data persists with `docker compose down` + `docker compose up`. Data is **only deleted** if you explicitly remove volumes:

```bash
# This DELETES all data:
docker compose down -v    # DON'T do this unless you mean it

# This is safe:
docker compose down       # stops containers, keeps data
docker compose up -d      # starts containers, data intact
```

### Slow performance on macOS

Bind mounts on macOS Docker Desktop can be slower than native. If performance is an issue, check Docker Desktop settings: **Settings > Resources > CPU/Memory** and allocate more resources.

## License

[MIT](LICENSE)
