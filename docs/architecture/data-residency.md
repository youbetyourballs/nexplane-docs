# Data Residency

This page describes what data Nexplane stores, where it is stored, and how long it is retained.

## What Nexplane Stores

| Data Category | Examples | Where Stored |
|---|---|---|
| Connector credentials | AWS secret keys, SSH passwords, LDAP bind passwords | PostgreSQL, encrypted at rest |
| Asset metadata | IAM user names, EC2 instance IDs, hostnames | PostgreSQL |
| Change request records | Title, description, risk score, approval history | PostgreSQL |
| Execution results | New key IDs, rollback state, error messages | PostgreSQL |
| Audit log | Every state transition with timestamp and actor | PostgreSQL |
| Agent certificates | Client TLS certificates issued at enrollment | PostgreSQL |
| User accounts | Email, hashed password, role | PostgreSQL |

## What Nexplane Does Not Store

- S3 object contents
- EC2 user data or instance metadata beyond what is shown in the UI
- Database row contents from connected PostgreSQL, Redis, or MongoDB instances
- Plaintext secrets after they are encrypted
- SSH session transcripts or command output beyond structured result payloads

## Self-Hosted vs SaaS

**Self-hosted:** All data stays in your PostgreSQL instance in your VPC. Nexplane SaaS systems have no access to your data. You control backups, retention, and deletion.

**SaaS (future):** When the SaaS offering launches, customers will be able to choose a deployment region. Data will not leave the selected region. Connector credentials will be encrypted with a customer-managed key stored in the customer's own KMS.

## Retention

Nexplane does not automatically delete any records. Change request records and audit logs are retained indefinitely by default. You can configure a retention policy in Settings > Data Retention to automatically purge records older than a specified number of days.

!!! warning "Audit log purging"
    Purging audit log records may violate your compliance requirements (SOC 2, PCI DSS, etc.). Review your compliance obligations before enabling automatic purging.

## Encryption

All data in PostgreSQL is protected by:

- Connector credentials: AES-256-GCM application-layer encryption (see [Credential Storage](../security/credentials.md))
- PostgreSQL disk encryption: depends on your database host's disk encryption configuration -- not managed by Nexplane
- TLS in transit: all connections between Nexplane components use TLS 1.2+

## Backup Recommendations

Nexplane does not manage database backups. For production deployments:

- Enable automated backups on your PostgreSQL host (RDS automated backups, pg_dump cron job, etc.)
- Test restore procedures quarterly
- Ensure backup files are encrypted and stored in a separate failure domain from the primary database
