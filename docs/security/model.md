# Security Model

This page describes Nexplane's security architecture: what trust assumptions it makes, where it enforces boundaries, and what threats it is and is not designed to address.

## Threat Model

Nexplane is designed to address:

- **Uncontrolled infrastructure changes** -- changes made outside a review and approval process
- **Credential sprawl** -- credentials that are never rotated or are over-privileged
- **Missing audit trails** -- inability to answer "who changed what, when, and why"
- **Slow incident response** -- hours or days to revoke a compromised credential

Nexplane is not designed to replace:

- Network firewalls or security groups
- Endpoint detection and response (EDR)
- Runtime security monitoring
- Vulnerability scanning

## Authentication

**User authentication:**
Users authenticate with email and password. Passwords are hashed with bcrypt (cost factor 12). Nexplane does not store plaintext passwords. JWT tokens (RS256, 1-hour TTL) are issued on login.

SSO integration (SAML, OIDC) is on the roadmap. Until then, Nexplane manages its own user database.

**Agent authentication:**
Agents authenticate with mutual TLS. Each agent has a unique RSA-2048 key pair. The private key is generated on the agent host at enrollment time and never leaves the host. The control plane issues a signed certificate valid for 1 year. The agent uses the certificate for all communication with the control plane.

## Authorization

**Role-based access control:**

| Role | Capabilities |
|---|---|
| Admin | Full access to all connectors, change requests, users, and settings |
| Operator | Can create, submit, and execute change requests; cannot manage connectors or users |
| Approver | Can approve or reject change requests; cannot create or execute |
| Viewer | Read-only access to change requests and assets |

Roles are assigned per user. Future versions will support connector-scoped roles (e.g., "Operator for prod-aws only").

## Connector Credential Security

Connector credentials (AWS keys, LDAP passwords, etc.) are stored encrypted in PostgreSQL. The encryption uses AES-256-GCM with a key derived from the `SECRET_KEY` environment variable.

- Credentials are encrypted before writing to the database
- Credentials are decrypted in memory only during connector operations
- Plaintext credentials are never logged, returned in API responses, or written to disk
- The encryption key is only available in the backend process environment

See [Credential Storage](credentials.md) for implementation details.

## Agent Command Allowlist

The Go agent does not execute arbitrary commands. All operations are typed structs defined at compile time. The control plane cannot instruct the agent to run a shell command that is not in the allowlist. This limits the impact of a compromised control plane on agent-managed hosts.

## Network Security

- All communication uses TLS 1.2 or higher
- Agent connections use mutual TLS (both sides present certificates)
- The agent initiates connections to the control plane -- the control plane never connects outbound to agents
- Connector API calls use the connector's own credentials -- Nexplane does not proxy arbitrary API calls

## Audit Trail

Every state transition in Nexplane is recorded in the audit log:

- Immutable -- audit records are append-only and cannot be modified or deleted through the API
- Comprehensive -- includes the actor, timestamp, event type, resource, and full payload
- Queryable -- available via the UI and API with time-range and resource filters

## What Nexplane Does Not Protect Against

- A compromised admin account -- if an attacker obtains admin credentials, they can create and execute change requests. Enable MFA (SSO) and follow least-privilege principles.
- A compromised `SECRET_KEY` -- this key can decrypt all stored connector credentials. Store it in a secrets manager and rotate it periodically.
- Supply chain attacks on the Nexplane codebase itself
- Changes made outside Nexplane (directly in the AWS console, direct SSH, etc.) -- Nexplane has no way to detect or block out-of-band changes
