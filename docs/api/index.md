# API Reference

The Nexplane control plane exposes a REST API on port 8000. All endpoints use JSON for request and response bodies.

## Authentication

All API requests (except `/auth/login` and `/health`) require a Bearer token in the `Authorization` header:

```
Authorization: Bearer <token>
```

Obtain a token by calling `/auth/login`:

```bash
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com", "password": "yourpassword"}'
```

Response:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

Tokens expire after 1 hour. Use `/auth/refresh` to obtain a new token without re-entering credentials.

---

## Endpoints

### Health

```
GET /health
```

Returns the service status. No authentication required.

**Response:**
```json
{
  "status": "ok",
  "db": "connected",
  "version": "0.1.0"
}
```

---

### Connectors

```
GET    /connectors           List all connectors
POST   /connectors           Create a connector
GET    /connectors/{id}      Get a connector
PUT    /connectors/{id}      Update a connector
DELETE /connectors/{id}      Delete a connector
POST   /connectors/{id}/test Test connector connectivity
POST   /connectors/{id}/discover  Trigger asset discovery
```

**Create connector (AWS example):**

```bash
curl -X POST http://localhost:8000/connectors \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "prod-aws",
    "type": "aws",
    "credentials": {
      "aws_access_key_id": "AKIAIOSFODNN7EXAMPLE",
      "aws_secret_access_key": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
      "region": "us-east-1"
    }
  }'
```

**Note:** Credential fields are write-only. The `GET /connectors/{id}` response omits all credential values.

---

### Assets

```
GET /assets                       List all discovered assets
GET /assets?connector_id={id}     Filter by connector
GET /assets?type={type}           Filter by asset type (e.g., iam_user, ec2_instance)
GET /assets/{id}                  Get a single asset
```

---

### Change Requests

```
GET    /change-requests           List change requests
POST   /change-requests           Create a change request
GET    /change-requests/{id}      Get a change request
POST   /change-requests/{id}/submit    Submit for approval
POST   /change-requests/{id}/approve   Approve
POST   /change-requests/{id}/reject    Reject
POST   /change-requests/{id}/execute   Execute an approved change
POST   /change-requests/{id}/rollback  Initiate rollback
```

**Create a change request:**

```bash
curl -X POST http://localhost:8000/change-requests \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Rotate prod-deploy IAM key",
    "connector_id": "conn_abc123",
    "change_type": "rotate_iam_access_key",
    "target": {
      "iam_user": "prod-deploy"
    },
    "description": "Quarterly credential rotation"
  }'
```

**Change request states:**

```
draft -> submitted -> approved -> executing -> complete
                  -> rejected
                            -> failed
                                     -> rolledback
```

---

### Agents

```
GET    /agents                    List registered agents
GET    /agents/{id}               Get agent details
DELETE /agents/{id}               Revoke agent registration
POST   /agents/enrollment-tokens  Create an enrollment token
```

---

### Audit Log

```
GET /audit                              List all audit events
GET /audit?change_request_id={id}       Filter by change request
GET /audit?user_id={id}                 Filter by user
GET /audit?from={iso8601}&to={iso8601}  Filter by time range
```

All audit events include:
- `timestamp`
- `actor` (user email or `system`)
- `event_type`
- `resource_type` and `resource_id`
- `payload` (event-specific details)

---

## Rate Limits

The API is rate-limited to 100 requests per minute per authenticated user. Rate limit headers are included in all responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1716000000
```

---

## OpenAPI / Swagger

The full OpenAPI schema is available at:

```
http://localhost:8000/openapi.json
```

An interactive Swagger UI is available at:

```
http://localhost:8000/docs
```
