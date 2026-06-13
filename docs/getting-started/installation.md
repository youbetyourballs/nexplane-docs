# Installation

Nexplane runs as a set of Docker containers managed by Docker Compose. This page walks you through getting the control plane running locally or on a server.

## Prerequisites

- Docker 24.0 or later
- Docker Compose v2.20 or later
- 2 GB RAM minimum (4 GB recommended)
- Ports 8000 (API) and 3000 (UI) available

## Step 1: Clone the Repository

```bash
git clone https://github.com/youbetyourballs/nexplane.git
cd nexplane
```

## Step 2: Start the Stack

```bash
docker compose up --build
```

This starts three containers:

| Service | Port | Description |
|---------|------|-------------|
| `backend` | 8000 | FastAPI control plane API (bound to 127.0.0.1 — not public internet) |
| `frontend` | 3000 | React UI |
| `db` | 5432 | PostgreSQL (internal only) |

Wait about 30 seconds for database migrations to complete. Watch the logs:

```bash
docker compose logs -f backend
```

When you see `Application startup complete`, the backend is ready.

## Step 3: Open the UI

Navigate to [http://localhost:3000](http://localhost:3000).

Log in with the demo credentials:

| Email | Password | Role |
|-------|----------|------|
| admin@acme.example | admin123 | Admin |
| operator@acme.example | operator123 | Security Operator |
| approver@acme.example | approver123 | Approver |
| auditor@acme.example | auditor123 | Auditor |

## Step 4: Verify the API

Interactive API docs are at [http://localhost:8000/docs](http://localhost:8000/docs).

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SECRET_KEY` | dev key | JWT + Fernet key derivation — **change in production** |
| `DATABASE_URL` | internal Postgres | Override for external database |
| `CORS_ORIGINS` | `http://localhost:3000,http://localhost:5173` | Allowed frontend origins |
| `ENVIRONMENT` | `development` | Environment name |
| `AI_MODEL` | `claude-sonnet-4-6` | Anthropic model for AI planning |
| `WEBHOOK_SECRET` | dev key | HMAC key for vulnerability scanner webhook verification (≥16 chars outside development) |
| `NEXPLANE_AGENT_DOWNLOAD_URL` | S3 base URL | Override for self-hosted agent binary distribution |
| `INSTANCE_URL` | `http://localhost:8000` | Public instance URL used for OIDC redirect URIs and email |

!!! warning "Production deployments"
    Always set `SECRET_KEY` to a random 32-byte hex string in production. The auto-generated key changes on container restart, which invalidates all active sessions.

    ```bash
    python3 -c "import secrets; print(secrets.token_hex(32))"
    ```

## Going to Production

The steps above run the development stack. For a hardened single-node install (HTTPS reverse proxy, built frontend) or a Kubernetes deployment via Helm, see [Production Deployment](production-deployment.md).

## Next Step

[Connect your first cloud account](connect-cloud.md)
