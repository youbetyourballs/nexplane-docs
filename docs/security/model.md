# Security Model

Nexplane's security model is built around three principles: every action is authenticated, every change is audited, and credentials never leave the control plane unencrypted.

## Authentication

**User authentication:** JWT + bcrypt. Users log in via `POST /auth/login` and receive a JWT. All API endpoints require a valid Bearer JWT. Tokens expire and require re-authentication.

**Agent authentication:** HMAC-SHA256. The agent registers with a shared secret generated in Settings → Agent Configuration. Every job payload dispatched to an agent is signed with this secret. The agent verifies the signature before executing any command — jobs with invalid signatures are rejected unconditionally.

## Authorization

Role-based access control with four roles:

| Role | Capabilities |
|------|-------------|
| Admin | Full access — connectors, settings, agent secret, approvals, all CRs |
| Security Operator | Create and execute change requests; manage assets and connectors |
| Approver | Approve change requests; view all CRs |
| Auditor | Read-only access to CRs, audit log, assets |

Additionally, the `ir_responder` role grants bypass capability for change freeze windows during active incidents.

## Credential Protection

All connector credentials are encrypted with `SecretsService` (Fernet AES-256) before being stored. Credentials are decrypted in memory only during connector operation execution. They are never returned in GET responses and never written to logs.

See [Credential Storage](credentials.md) for the full implementation.

## Agent Security

- Agent jobs are HMAC-SHA256 signed — no unsigned job will execute
- The agent only executes typed commands registered at compile time — no arbitrary shell
- The agent communicates outbound only — no inbound network access required on managed hosts
- Binary integrity: SHA256 checksum verified before self-update takes effect

## Audit Trail

Every state transition on a change request, every approval, every execution, and every rollback is recorded immutably in the `audit_events` table with a timestamp, acting user, and full payload. The audit trail is queryable via `GET /audit-events/` but not deletable or modifiable by any user or role.
