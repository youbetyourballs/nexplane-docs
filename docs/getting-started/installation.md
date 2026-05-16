# Installation

Nexplane runs as a set of Docker containers managed by Docker Compose. This page walks you through getting the control plane running locally or on a server in about 5 minutes.

## Prerequisites

- Docker 24.0 or later
- Docker Compose v2.20 or later
- 2 GB RAM minimum (4 GB recommended)
- Ports 8000 (API) and 3000 (UI) available

## Step 1: Pull the Compose File

```bash
curl -O https://raw.githubusercontent.com/youbetyourballs/nexplane/main/docker-compose.yml
```

Or clone the repo:

```bash
git clone https://github.com/youbetyourballs/nexplane.git
cd nexplane
```

## Step 2: Start the Stack

```bash
docker compose up -d
```

This starts:

| Service | Port | Description |
|---|---|---|
| `backend` | 8000 | FastAPI control plane API |
| `frontend` | 3000 | React UI |
| `db` | 5432 | PostgreSQL (internal only) |
| `redis` | 6379 | Task queue (internal only) |

Wait about 30 seconds for the database migrations to complete. You can watch the logs:

```bash
docker compose logs -f backend
```

When you see `Application startup complete`, the backend is ready.

## Step 3: Open the UI

Navigate to [http://localhost:3000](http://localhost:3000).

On first launch, Nexplane will prompt you to create an admin account. Enter your email and a password. This account has full access to all connectors and change requests.

## Step 4: Verify the API

```bash
curl http://localhost:8000/health
```

Expected response:

```json
{"status": "ok", "db": "connected", "version": "0.1.0"}
```

## Environment Variables

The following variables can be set in a `.env` file in the same directory as `docker-compose.yml`:

| Variable | Default | Description |
|---|---|---|
| `SECRET_KEY` | auto-generated | JWT signing key -- set this in production |
| `DATABASE_URL` | internal Postgres | Override to use an external database |
| `REDIS_URL` | internal Redis | Override to use an external Redis |
| `CORS_ORIGINS` | `http://localhost:3000` | Allowed frontend origins |
| `LOG_LEVEL` | `info` | `debug`, `info`, `warning`, `error` |

!!! warning "Production deployments"
    Always set `SECRET_KEY` to a random 32-byte hex string in production. The auto-generated key changes on container restart, which invalidates all active sessions.

    ```bash
    python3 -c "import secrets; print(secrets.token_hex(32))"
    ```

## Upgrading

See the [Upgrade Runbook](../runbooks/upgrade.md) for instructions on upgrading between versions.

## Next Step

[Connect your first cloud account](connect-cloud.md)
