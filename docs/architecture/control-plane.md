# Control Plane

The Nexplane control plane is a FastAPI application backed by PostgreSQL. It exposes a REST API consumed by the React frontend and the Go agent.

## Technology Stack

| Component | Technology |
|---|---|
| API framework | FastAPI (Python) |
| Database ORM | SQLAlchemy with Alembic migrations |
| Database | PostgreSQL 15+ |
| Task queue | Celery + Redis |
| Authentication | JWT (RS256) |
| Credential encryption | AES-256-GCM with per-secret IVs |

## API Structure

The API is organized into the following routers:

| Prefix | Description |
|---|---|
| `/auth` | Login, logout, token refresh |
| `/connectors` | CRUD for connector configurations |
| `/assets` | Asset inventory queries |
| `/change-requests` | Change request lifecycle |
| `/agents` | Agent enrollment and management |
| `/users` | User management |
| `/audit` | Audit log queries |
| `/health` | Liveness and readiness probes |

All endpoints except `/auth/login` and `/health` require a valid JWT.

## Database Schema

The core tables are:

- `connectors` -- connector type, name, encrypted credential blob
- `assets` -- discovered assets with connector FK and metadata JSON
- `change_requests` -- change lifecycle, risk score, approval state
- `change_request_events` -- append-only audit log for each state transition
- `users` -- email, hashed password, role
- `agents` -- hostname, OS, version, certificate fingerprint, last heartbeat
- `enrollment_tokens` -- one-time tokens for agent registration

## Credential Encryption

Connector credentials are encrypted before being written to the `connectors` table. The encryption uses:

- AES-256-GCM
- A unique 96-bit IV per encryption operation
- The master key loaded from the `SECRET_KEY` environment variable (stretched with PBKDF2)

The ciphertext, IV, and authentication tag are stored as a single base64-encoded blob. Plaintext credentials never touch disk.

## Change Orchestration

When a change request is approved and executed, the backend:

1. Loads the connector credentials and decrypts them in memory
2. Instantiates the connector class for the change type
3. Calls `connector.pre_check()` to verify preconditions
4. Calls `connector.execute()` to perform the change
5. Stores the execution result and any rollback state in the change record
6. Calls `connector.verify()` to confirm the change took effect
7. Updates the change request status to `complete` or `failed`

If any step raises an exception, the change moves to `failed` and the error is logged. Rollback is not triggered automatically -- a human must initiate it.

## Background Tasks

Long-running operations (asset discovery, bulk changes) run as Celery tasks in a worker process. Task status is polled by the frontend via a `/tasks/{task_id}` endpoint.

## Horizontal Scaling

The backend is stateless between requests. Multiple backend replicas can run behind a load balancer with a shared PostgreSQL instance. The Celery workers scale independently. Redis is used only for the task queue -- no session state is stored in Redis.
