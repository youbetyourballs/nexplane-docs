# Connectors

Connectors are the integrations between Nexplane and the external systems it manages. Each connector serves two functions: **ingest** (discovering assets into inventory) and **change actions** (executing change requests against that system).

## Two Functions

**Ingest** — a connector polls its external system on a schedule and syncs discovered resources as assets into Nexplane. EC2 instances, IAM users, Okta users, Kubernetes deployments — all discovered automatically.

**Change actions** — a connector implements the execution logic for one or more change types. When a change request is approved and executed, Nexplane routes it to the correct connector executor, which calls the external API.

## Credential Storage

Connector credentials (API keys, service principal secrets, private keys) are encrypted with `SecretsService` (Fernet AES-256) before being stored. They are never returned in GET responses. The SecretsService abstraction is designed for swap-out to HashiCorp Vault, AWS Secrets Manager, or an HSM without changing connector code.

## Connector Count

Nexplane includes 38+ connectors across six categories:

| Category | Connectors |
|----------|-----------|
| Cloud & Infrastructure | AWS, Azure, GCP, Cloudflare, Palo Alto, Tailscale |
| Identity & Access | Okta, Active Directory, Microsoft Entra ID, HashiCorp Vault, GitHub |
| Security Tools / EDR | CrowdStrike, Tenable, SentinelOne, Snyk, Qualys, Wiz, RunZero |
| SaaS | Google Workspace, Slack, Kubernetes, Helm |
| IaC & Configuration | Terraform (local), Terraform (remote), Ansible (local), Ansible (remote) |
| Workflow & Observability | Jira, PagerDuty, ServiceNow, Splunk, Datadog |
| Host Execution | SSH, Nexplane Agent |

See [Connectors](../connectors/index.md) for the full list and individual connector pages.

## Test Connection

After saving a connector's credentials, use **Test Connection** to verify they are valid. For cloud connectors, this typically calls an identity verification endpoint (e.g., `sts:GetCallerIdentity` for AWS).

## Scheduled Ingest

Each connector has a configurable ingest schedule. You can also trigger manual ingest at any time from the connector detail page.
