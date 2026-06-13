# Editions & First-Run Setup

Nexplane ships in two editions, selected by the `NEXPLANE_EDITION` environment variable (default `core`).

## Editions

| Edition | What it includes |
|---------|------------------|
| `core` | The full open platform — every connector, change type, runbook, and agent capability documented on this site |
| `commercial` | Everything in `core`, **plus** the first-run setup-token flow, the setup guard, and a **commercial CR catalog** loaded at runtime |

The `core` edition is fully self-contained — nothing is held back. The `commercial` edition layers additional capabilities on top without forking the platform.

### The commercial overlay

Commercial executors live **outside** the `nexplane` Python package and are mounted into the container at runtime, so **no commercial code ships in the core image**. The overlay is configured with two variables:

| Variable | Purpose |
|----------|---------|
| `NEXPLANE_EDITION=commercial` | Enables the setup flow, setup guard, and commercial catalog |
| `NEXPLANE_COMMERCIAL_CATALOG_PATH` | Filesystem path to the mounted commercial CR catalog and executors |

Commercial deployments support two delivery models:

- **Managed** — Nexplane provisions one isolated instance per client
- **Self-hosted** — the customer runs the Docker Compose bundle, a VM image, or the Helm chart

Either way, a fresh instance is bootstrapped through the first-run setup-token flow described below.

## First-Run Setup

A fresh `commercial`-edition instance ships **locked** until it is bootstrapped with a first admin user. `SetupGuardMiddleware` 307-redirects all non-exempt traffic to `/setup` until the first user exists.

Exempt paths (reachable before setup) include `/setup`, `/health`, `/docs`, `/openapi.json`, and `/api/v1/setup`.

### The two-step token flow

Setup uses a one-time, instance-bound token so that only an operator with machine-level access can create the first admin.

**Step 1 — mint a token (machine-to-machine):**

```
POST /setup/token
X-Ops-Secret: <shared secret>
```

- Authenticated by the `X-Ops-Secret` header, sourced from `NEXPLANE_OPS_SECRET` or the SSM parameter `/nexplane/ops/instance-shared-secret`
- Mints a one-time setup token bound to the instance URL, with a 24-hour TTL
- Returns the setup URL containing the token

**Step 2 — consume the token (browser):**

```
POST /setup/consume
```

- Unauthenticated, but validates the token against the instance URL
- Creates the first organization and admin user (password must be 12+ characters)
- Marks the token used and returns a JWT

After the first admin exists, the setup guard disengages and the instance serves normally.

!!! warning "Token security"
    Setup tokens are **single-use**, **instance-URL-bound**, and expire after **24 hours**. The `/setup/token` endpoint always requires the `X-Ops-Secret` header — there is no unauthenticated way to mint one.

## See also

- [Authentication & SSO](authentication.md)
- [Production Deployment](../getting-started/production-deployment.md)
- [Safety Engine](safety-engine.md)
