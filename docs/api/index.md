# API Reference

The Nexplane control plane exposes a REST API on port 8000. All endpoints use JSON for request and response bodies.

## Interactive Documentation

Interactive OpenAPI (Swagger UI) docs are available at:

```
http://localhost:8000/docs
```

ReDoc format:

```
http://localhost:8000/redoc
```

## Authentication

All endpoints require a Bearer JWT obtained from `POST /auth/login`:

```bash
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@acme.example", "password": "admin123"}'
```

Response includes a `token` field. Pass it as:

```
Authorization: Bearer <token>
```

## Endpoint Groups

| Group | Description |
|-------|-------------|
| `/auth/*` | Login, logout, current user |
| `/assets/*` | Asset inventory CRUD, tag management, bulk tagging, ingest |
| `/connectors/*` | Connector CRUD, test connection, credentials, schedule, ingest |
| `/projects/*` | Project CRUD, member management, AI planning chat |
| `/change-requests/*` | Full CR lifecycle (plan→approve→execute→verify→rollback) + batch progress |
| `/runbooks/*` | Runbook CRUD, fork, trigger |
| `/executions/*` | Execution status, resume checkpoint, abort |
| `/vulnerability/*` | Findings CRUD, policies, SLA dashboard, CVE blast-radius, patch campaign |
| `/vulnerability/webhooks/vulnerability-findings` | HMAC-verified webhook endpoint for scanner push |
| `/ir/*` | IR playbook templates, execute, forensic bundles |
| `/access-reviews/*` | Collect memberships, reviewer decisions, approve, auto-generate removal CRs |
| `/compliance/*` | Baselines CRUD, drift alerts, evidence ZIP download, freeze windows |
| `/maintenance-windows/*` | Maintenance window CRUD, status check |
| `/agent/*` | Agent registration, job dispatch, result reporting |
| `/settings/*` | AI providers, agent secret, remediation policies |
| `/downloads/*` | Versioned agent binaries + SHA256 checksums + version file |
| `/audit-events/*` | Immutable audit trail (read-only) |

## Key Endpoints

### Change Request Lifecycle

```
POST   /change-requests/           Create a new CR (Draft)
POST   /change-requests/{id}/plan  Submit for safety review → Planned
POST   /change-requests/{id}/approve  Approve → Approved
POST   /change-requests/{id}/execute  Execute → Executing → Completed
POST   /change-requests/{id}/rollback  Trigger rollback → Rolled Back
GET    /change-requests/{id}/progress  Poll batch execution progress
```

### Agent

```
POST   /agent/register             Register a new agent
GET    /agent/jobs/next            Long-poll for next job (agent calls this)
POST   /agent/result               Post job result (agent calls this)
```

### Vulnerability Pipeline

```
POST   /vulnerability/webhooks/vulnerability-findings   HMAC-verified webhook
GET    /vulnerability/findings/                         List findings
GET    /vulnerability/cve/{cve_id}/blast-radius         CVE blast-radius query
POST   /vulnerability/patch-campaign                    Generate patch campaign CRs
GET    /vulnerability/sla/config                        Get SLA config
PUT    /vulnerability/sla/config                        Update SLA config
```

### Compliance

```
GET    /compliance/baselines/                   List compliance baselines
POST   /compliance/baselines/                   Create baseline
GET    /compliance/drift/                       List drift alerts
GET    /compliance/evidence/{id}/download       Download evidence ZIP
POST   /compliance/freeze-windows/              Create change freeze window
DELETE /compliance/freeze-windows/{id}          Remove freeze window
```

## Rate Limiting

No rate limiting is applied in the default configuration. For production deployments, configure rate limiting at the reverse proxy layer.
