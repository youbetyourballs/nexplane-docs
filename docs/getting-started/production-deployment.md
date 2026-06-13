# Production Deployment

The default `docker-compose.yml` is for local development. For a hardened single-node install, use the production stack, which adds an HTTPS reverse proxy and a built (non-dev) frontend. A Helm chart is also available for Kubernetes.

## Single-Node (Docker Compose)

```bash
docker compose -f docker-compose.prod.yml up -d
```

The production stack adds:

- **`nginx` reverse proxy** (`deploy/nginx.conf`) — terminates TLS (1.2+, strong ciphers; certs at `/etc/nginx/certs/{fullchain,privkey}.pem`), 301-redirects HTTP → HTTPS, proxies `/api/*` (stripping the `/api` prefix) and `/setup/*` to the backend, and serves the SPA from the frontend
- **Built frontend** (`frontend/Dockerfile.prod`) — a Vite production build served by nginx, with `VITE_API_URL` / `VITE_AGENT_DOWNLOAD_URL` baked in at build time
- **Backend** — runs Alembic `upgrade head` on startup, then uvicorn with 2 workers (same image as dev; bundles Tailscale, Terraform, Ansible, the SSM plugin, and Helm)
- **Postgres 16** behind a healthcheck gate

### Required production settings

Set these before exposing the instance:

| Variable | Requirement |
|----------|-------------|
| `SECRET_KEY` | Random string, **≥32 chars** — JWT + Fernet key derivation |
| `WEBHOOK_SECRET` | **≥16 chars** — HMAC key for vulnerability webhook verification |
| `INSTANCE_URL` | Public URL of the instance (used for OIDC redirect URIs, setup links, and email) |
| `CORS_ORIGINS` | Your actual frontend origin(s) |

Add SMTP variables if email notifications are used.

!!! warning "Rotate the dev key"
    The auto-generated `SECRET_KEY` changes on every container restart, which invalidates all active sessions. Always pin a stable value in production:

    ```bash
    python3 -c "import secrets; print(secrets.token_hex(32))"
    ```

## Kubernetes (Helm)

A Helm chart lives in `helm/nexplane/` with three values profiles:

| Profile | Use | Notes |
|---------|-----|-------|
| `values.yaml` | Default / single-node | Bundled Postgres + Redis, 1 replica, ingress off |
| `values-enterprise.yaml` | Self-hosted enterprise | External Postgres + Redis, 2 replicas, ingress + TLS |
| `values-managed.yaml` | Nexplane-managed | Enterprise layout with tuned resource requests/limits |

```bash
helm install nexplane ./helm/nexplane -f helm/nexplane/values-enterprise.yaml
```

## Commercial Edition

To run a `commercial` instance, set `NEXPLANE_EDITION=commercial` and mount the commercial CR catalog/executors at `NEXPLANE_COMMERCIAL_CATALOG_PATH`. A fresh commercial instance is bootstrapped through the first-run setup-token flow before it serves any other traffic.

See [Editions & First-Run Setup](../security/editions.md) for the full bootstrap procedure.

## See also

- [Installation](installation.md) — local development setup and the full environment-variable reference
- [Network Exposure](../security/network-exposure.md) — what is and isn't exposed
- [Authentication & SSO](../security/authentication.md)
