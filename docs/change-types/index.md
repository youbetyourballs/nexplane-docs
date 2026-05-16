# Change Types

A change type is a structured, typed operation that Nexplane knows how to execute and roll back. Every change request is associated with exactly one change type. Change types define:

- What connector they require
- What parameters they accept
- What the execution steps are
- What the rollback operation is

## Categories

### Compute

Compute changes affect running infrastructure -- virtual machines, containers, and cloud instances.

| Change Type | Connectors | Description |
|---|---|---|
| Snapshot EC2 Instance | AWS | Create an EBS snapshot before a risky change |
| Stop/Start Instance | AWS, GCP, Azure, OCI | Power operations with rollback |
| Cordon/Uncordon Node | Kubernetes | Control workload scheduling |
| Patch Deployment | Kubernetes | Update image or configuration |

See [Compute](compute.md).

### Identity

Identity changes affect user accounts, service accounts, and authentication configurations.

| Change Type | Connectors | Description |
|---|---|---|
| Lock User Account | LDAP, Keycloak, AWS, GCP, PostgreSQL, MongoDB | Disable a user account |
| Unlock User Account | LDAP, Keycloak, AWS, GCP, PostgreSQL, MongoDB | Re-enable a user account |
| Add to Group | LDAP, Keycloak | Modify group membership |
| Remove from Group | LDAP, Keycloak | Modify group membership |

See [Identity](identity.md).

### Credentials

Credential changes rotate or revoke secrets, API keys, and passwords.

| Change Type | Connectors | Description |
|---|---|---|
| Rotate IAM Access Key | AWS | Create new key, deactivate old key |
| Rotate Service Account Key | GCP | Create new key, delete old key |
| Rotate Client Secret | Azure, Keycloak | Generate new secret |
| Rotate OCI API Key | OCI | Upload new key, delete old key |
| Rotate KV Secret | Vault | Write new secret version |
| Rotate Database Password | PostgreSQL, MongoDB, Redis | Update password |
| Rotate Local Password | SSH, WinRM | Set new OS account password |

See [Credentials](credentials.md).

### Hardening

Hardening changes improve the security posture of a host or system by applying configuration baselines.

| Change Type | Connectors | Description |
|---|---|---|
| Apply CIS Profile | SSH, WinRM, Agent | Apply a CIS benchmark profile |
| Disable Unused Service | SSH, WinRM, Agent | Disable a named system service |
| Set File Permission | SSH, Agent | Fix insecure file permissions |
| Set Sysctl Parameter | SSH, Agent | Apply kernel hardening settings |

See [Hardening](hardening.md).

### Vulnerability Remediation

Vulnerability changes address specific CVEs or misconfigurations identified by a scanner.

| Change Type | Connectors | Description |
|---|---|---|
| Install Package Update | SSH, Agent | Update a package to a specific version |
| Remove Vulnerable Package | SSH, Agent | Remove a package with no available fix |
| Revoke Exposed Credential | AWS, GCP, Azure, LDAP, Vault | Immediately revoke a known-compromised credential |

See [Vulnerability Remediation](vulnerability.md).

## Risk Scoring

Each change type has a base risk score. The final risk score for a change request is calculated from:

- Base risk score of the change type
- Environment label of the connector (prod scores higher than staging)
- Blast radius of the target (how many systems depend on it)
- Whether rollback is available for this change type

Risk levels:

| Score | Level | Default Approval Required |
|---|---|---|
| 1-3 | Low | None (auto-approved in non-prod) |
| 4-6 | Medium | Single approver |
| 7-9 | High | Two approvers |
| 10 | Critical | Two approvers + time delay |
