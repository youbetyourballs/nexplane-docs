# Connectors

A connector is a credential-backed integration that allows Nexplane to discover assets and execute changes in an external system. Each connector stores its credentials encrypted and exposes typed actions that map to Nexplane change types.

## Cloud & Infrastructure

| Connector | Key Capabilities |
|-----------|-----------------|
| [AWS](aws.md) | EC2 lifecycle, IAM, S3, Route53, RDS, CloudWatch, ALB, security groups, SSM, Tailscale, agent deploy |
| [Azure](azure.md) | VM lifecycle, NSG, blob storage, managed identity, RBAC, VNet, DNS, SQL, Monitor alerts, Entra users |
| [GCP](gcp.md) | Compute lifecycle, firewall rules, storage, service accounts, IAM, SCC |
| [OCI](oci.md) | Compute lifecycle, VCN/NSG/security lists, object storage, block volumes, IAM, ADB, MySQL, DNS, Vault, monitoring |
| [Cloudflare](cloudflare.md) | WAF, firewall rules, access policies, block IP, SSL mode, DNS |
| [Palo Alto](paloalto.md) | Address objects, rules, zones, block IP, commit |
| [OPNsense](opnsense.md) | Firewall rules, block host |
| [Zscaler](zscaler.md) | URL/IP blocking, user suspension, URL category management |
| [Tailscale](tailscale.md) | Join/remove nodes, auth key management |

## Identity & Access

| Connector | Key Capabilities |
|-----------|-----------------|
| [Okta](okta.md) | User disable/enable, group management, MFA enforcement, session revoke, API token rotation |
| [Active Directory](active-directory.md) | Discover/disable stale accounts, move OU, rotate service account passwords |
| [Microsoft Entra ID](entra-id.md) | User disable/enable, revoke sessions, assign/remove license, remove from Teams |
| [Azure AD (Graph API)](azure-ad.md) | User disable, user create via Microsoft Graph |
| [LDAP](ldap.md) | Disable user accounts via `pwdAccountLockedTime` |
| [FreeIPA](freeipa.md) | Disable user accounts via JSON-RPC |
| [Keycloak](keycloak.md) | Disable user and revoke sessions |
| [Teleport](teleport.md) | Lock user via `tctl` |
| [HashiCorp Vault](vault.md) | Secret discovery, dynamic credentials, secret rotation |
| [Infisical](infisical.md) | Secret rotation |
| [Microsoft LAPS](laps.md) | Retrieve and rotate LAPS-managed local admin passwords |

## Code & Version Control

| Connector | Key Capabilities |
|-----------|-----------------|
| [GitHub](github.md) | Org member management, PAT revocation, branch protection, repo management |
| [GitLab](gitlab.md) | User suspension, PAT rotation |
| [Gitea](gitea.md) | User suspension |
| [JFrog Xray](jfrog.md) | Artifact scanning, policy violation sync |

## Security Tools & EDR

| Connector | Key Capabilities |
|-----------|-----------------|
| [CrowdStrike](crowdstrike.md) | Isolate host, RTR commands, prevention policy, host containment |
| [Microsoft Defender for Endpoint](defender-endpoint.md) | Machine isolation, antivirus scan, investigation, TVM vulnerability ingest |
| [SentinelOne](sentinelone.md) | Endpoint discovery, alert ingest |
| [Wazuh](wazuh.md) | Agent registration |
| [Falco](falco.md) | Runtime security rule management via SSM |

## Vulnerability Scanners

| Connector | Key Capabilities |
|-----------|-----------------|
| [Tenable](tenable.md) | Launch/pause/resume scans, export findings |
| [Snyk](snyk.md) | Vulnerability findings ingest |
| [Qualys](qualys.md) | Vulnerability findings ingest, scan launch |
| [OpenVAS](openvas.md) | Run scans, import CVE findings |
| [Nessus](nessus.md) | Run vulnerability scans |
| [Wiz](wiz.md) | Cloud security findings, misconfiguration discovery |
| [RunZero](runzero.md) | Network asset discovery |
| [Elastic Security](elastic.md) | Security alert sync, detection rule creation |

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
| [Terraform (Local)](terraform-local.md) | Apply/destroy in backend container, AWS credentials injected |
| [Terraform (Remote)](terraform-remote.md) | Two-phase plan→apply via external workspace |
| [Ansible (AWX)](ansible-awx.md) | Launch AWX job templates, manage inventories |
| [Ansible (Local)](ansible-local.md) | Run playbooks via SSM transport |
| [Ansible (Remote)](ansible-remote.md) | Remote playbook execution |
| [SaltStack](saltstack.md) | Run states/functions on minions, manage keys |
| [AWS CloudFormation](cloudformation.md) | Stack management, drift detection, change sets |
| [Azure Bicep](bicep.md) | ARM/Bicep template deployments |
| [Pulumi](pulumi.md) | Stack discovery, cancel/refresh operations |
| [Checkov](checkov.md) | IaC security scanning, secrets detection |
| [Chef InSpec](chef-inspec.md) | Compliance scanning via Chef Automate |
| [step-ca](step-ca.md) | TLS certificate rotation and expiry checking |

## Windows Management

| Connector | Key Capabilities |
|-----------|-----------------|
| [Microsoft Intune](intune.md) | Device discovery, policy sync, script deployment |
| [SCCM / MECM](sccm.md) | Application deployment, script execution, inventory collection |
| [Windows Update for Business](wufb.md) | Update ring management, compliance checking |
| [WinRM](winrm.md) | Agent bootstrap on Windows hosts |

## Workflow & Observability

| Connector | Key Capabilities |
|-----------|-----------------|
| [Jira](jira.md) | Ticket creation and status sync |
| [PagerDuty](pagerduty.md) | Incident creation, alert routing |
| [ServiceNow](servicenow.md) | Change request sync |
| [Splunk](splunk.md) | Event ingest, search query execution |
| [Datadog](datadog.md) | Monitor ingest, alert creation |

## Databases

| Connector | Key Capabilities |
|-----------|-----------------|
| [PostgreSQL](postgres.md) | User password rotation via direct connection |
| [Redis](redis.md) | AUTH password rotation |
| [MongoDB](mongodb.md) | User password rotation |

## Host Execution

| Connector | Key Capabilities |
|-----------|-----------------|
| [SSH](ssh.md) | Execute approved command templates on Linux/Unix hosts |
| [Nexplane Agent Connector](nexplane-agent.md) | Full OS hardening + all 15 agent command packages |

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
