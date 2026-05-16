# Connectors

A connector is a credential-backed integration that allows Nexplane to discover assets and execute changes in an external system. Each connector stores its credentials encrypted in the Nexplane backend and exposes a set of typed actions that map to Nexplane change types.

## Connector Categories

### Cloud

| Connector | Assets Discovered | Key Actions |
|---|---|---|
| [AWS](aws.md) | IAM users, roles, EC2, S3, security groups | Rotate IAM key, modify security group, snapshot EC2 |
| [GCP](gcp.md) | Service accounts, GCE instances, GCS buckets | Rotate service account key, modify firewall rule |
| [Azure](azure.md) | Service principals, VMs, storage accounts | Rotate client secret, modify NSG rule |
| [OCI](oci.md) | IAM users, compute instances, object storage | Rotate API key, modify security list |

### Identity

| Connector | Assets Discovered | Key Actions |
|---|---|---|
| [LDAP](ldap.md) | Users, groups, OUs | Reset password, lock account, add/remove group membership |
| [Keycloak](keycloak.md) | Users, realms, clients | Reset password, disable user, rotate client secret |

### Secrets

| Connector | Assets Discovered | Key Actions |
|---|---|---|
| [HashiCorp Vault](vault.md) | Secret paths, policies, auth methods | Rotate secret, revoke token, update policy |

### Orchestration

| Connector | Assets Discovered | Key Actions |
|---|---|---|
| [Kubernetes](kubernetes.md) | Pods, services, secrets, RBAC bindings | Rotate service account token, update secret, patch deployment |

### Hosts

| Connector | Assets Discovered | Key Actions |
|---|---|---|
| [SSH](ssh.md) | Reachable Linux/Unix hosts | Run hardening commands, rotate local password, manage services |
| [WinRM](winrm.md) | Reachable Windows hosts | Run PowerShell hardening, rotate local password, manage services |

### Databases

| Connector | Assets Discovered | Key Actions |
|---|---|---|
| [PostgreSQL](postgres.md) | Databases, users, roles | Rotate password, revoke privileges, create read-only user |
| [Redis](redis.md) | Redis instance metadata | Rotate AUTH password, flush volatile keys |
| [MongoDB](mongodb.md) | Databases, users, collections | Rotate password, revoke role, create read-only user |

## Adding a Connector

1. Go to **Settings > Connectors > Add Connector**
2. Select the connector type
3. Fill in the credential fields (see the connector's page for field details)
4. Click **Save** -- credentials are encrypted before storage
5. Click **Test Connection** to verify
6. Click **Discover Assets** to populate the asset inventory

## Credential Security

Connector credentials are encrypted with AES-256-GCM before being written to the database. The encryption key is derived from the `SECRET_KEY` environment variable. Credentials are decrypted in memory only when a connector operation is being executed, and are never logged or exposed in API responses.

See [Credential Storage](../security/credentials.md) for full details.
