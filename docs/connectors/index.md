# Connectors

A connector is a credential-backed integration that allows Nexplane to discover assets and execute changes in an external system. Each connector stores its credentials encrypted and exposes typed actions that map to Nexplane change types.

Nexplane includes 38+ connectors across six categories.

## Cloud & Infrastructure

| Connector | Key Capabilities |
|-----------|-----------------|
| [AWS](aws.md) | EC2 lifecycle, IAM, S3, Route53, RDS, CloudWatch, ALB, security groups, SSM, Tailscale, agent deploy |
| [Azure](azure.md) | VM lifecycle, NSG, blob storage, managed identity, RBAC, VNet, DNS, SQL, Monitor alerts, Entra users |
| [GCP](gcp.md) | Compute lifecycle, firewall rules, storage, service accounts, IAM, SCC |
| [Cloudflare](cloudflare.md) | WAF, firewall rules, access policies, block IP, SSL mode, DNS |
| [Palo Alto](paloalto.md) | Address objects, rules, zones, block IP, commit |
| [Tailscale](tailscale.md) | Join/remove nodes, auth key management |

## Identity & Access

| Connector | Key Capabilities |
|-----------|-----------------|
| [Okta](okta.md) | User disable/enable, group management, MFA enforcement, session revoke, API token rotation |
| [Active Directory](active-directory.md) | Discover/disable stale accounts, move OU, rotate service account passwords |
| [Microsoft Entra ID](entra-id.md) | User disable/enable, revoke sessions, assign/remove license, remove from Teams |
| [HashiCorp Vault](vault.md) | Secret discovery, dynamic credentials, secret rotation |
| [GitHub](github.md) | Org member management, PAT revocation, branch protection, repo management |

## Security Tools / EDR

| Connector | Key Capabilities |
|-----------|-----------------|
| [CrowdStrike](crowdstrike.md) | Isolate host, RTR commands, prevention policy, host containment |
| [Tenable](tenable.md) | Launch/pause/resume scans, export findings, trigger remediation verification |
| [SentinelOne](sentinelone.md) | Endpoint discovery, alert ingest |
| [Snyk](snyk.md) | Vulnerability findings ingest |
| [Qualys](qualys.md) | Vulnerability findings ingest, scan launch |
| [Wiz](wiz.md) | Cloud security findings, misconfiguration discovery |
| [RunZero](runzero.md) | Network asset discovery |

## SaaS

| Connector | Key Capabilities |
|-----------|-----------------|
| [Google Workspace](google-workspace.md) | User suspension, group removal, 2FA reset, OAuth token revocation, device wipe |
| [Slack](slack.md) | User deactivation/reactivation |
| [Kubernetes](kubernetes.md) | Deployment restart/scale, network policy, RBAC, secret rotation, Helm |
| [Helm](helm.md) | Upgrade/rollback Kubernetes releases |

## IaC & Configuration

| Connector | Key Capabilities |
|-----------|-----------------|
| [SSH](ssh.md) | Execute approved command templates, check service status |
| [Terraform (Local)](terraform-local.md) | Apply/destroy in backend container, AWS credentials injected |
| [Terraform (Remote)](terraform-remote.md) | Two-phase plan→apply via external workspace |
| [Ansible (Local)](ansible-local.md) | Run playbooks via SSM transport |
| [Ansible (Remote)](ansible-remote.md) | Remote playbook execution |
| [Nexplane Agent Connector](nexplane-agent.md) | Full OS hardening + all 15 agent command packages |

## Workflow & Observability

| Connector | Key Capabilities |
|-----------|-----------------|
| [Jira](jira.md) | Ticket creation and status sync |
| [PagerDuty](pagerduty.md) | Incident creation, alert routing |
| [ServiceNow](servicenow.md) | Change request sync |
| [Splunk](splunk.md) | Event ingest, search query execution |
| [Datadog](datadog.md) | Monitor ingest, alert creation |

## Planned Connectors

The following connectors are in development:

[OCI](oci.md) · [LDAP](ldap.md) · [Keycloak](keycloak.md) · [WinRM](winrm.md) · [PostgreSQL](postgres.md) · [Redis](redis.md) · [MongoDB](mongodb.md)

## Adding a Connector

1. Go to **Connectors → Add Connector**
2. Select the connector type
3. Fill in the credential fields (see each connector's page)
4. Click **Save** — credentials are encrypted before storage
5. Click **Test Connection** to verify
6. Click **Trigger Ingest** to populate asset inventory

## Credential Security

Connector credentials are encrypted with Fernet AES-256 via `SecretsService` before being written to the database. They are decrypted in memory only during connector operation execution and are never logged or returned in API responses.

See [Credential Storage](../security/credentials.md) for full details.
