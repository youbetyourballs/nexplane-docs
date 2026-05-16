# Nexplane Documentation

Nexplane is a security control plane that gives your security team a single, auditable interface for making changes across your entire infrastructure -- cloud accounts, identity systems, secrets managers, databases, and bare-metal hosts.

Every change in Nexplane is structured: it has a risk score, an approval gate, an execution step, a verification step, and a typed rollback. Nothing runs ad hoc. Nothing is irreversible by accident.

---

## What Nexplane Does

| Capability | Description |
|---|---|
| Change Requests | Structured, approved, auditable changes across any connector |
| Risk Scoring | Automatic risk assessment before any change executes |
| Approval Gates | Single or multi-party approval with policy enforcement |
| Typed Rollback | Every change type knows its own inverse operation |
| Asset Discovery | Continuous inventory across all connected systems |
| Connector Library | 14 connectors covering cloud, identity, secrets, databases, and hosts |
| Agent Execution | Go-based agent runs hardening and local changes on-host |

---

## Quick Start

If you want to be up and running in 15 minutes, start here:

1. [Install Nexplane with Docker Compose](getting-started/installation.md)
2. [Connect your first cloud account](getting-started/connect-cloud.md)
3. [Create your first change request](getting-started/first-change-request.md)
4. [Deploy the agent to a host](getting-started/deploy-agent.md)

---

## Architecture at a Glance

Nexplane has three layers:

- **Control Plane** -- FastAPI backend and React frontend, hosted by Nexplane or self-hosted in your VPC
- **Connectors** -- credential-backed integrations that talk to cloud APIs, identity systems, and databases
- **Agent** -- a Go binary that runs on Linux, Windows, and macOS hosts and executes local changes

All connector credentials are encrypted at rest. The agent communicates with the control plane over mutual TLS. No credentials ever leave the control plane unencrypted.

See [Architecture Overview](architecture/overview.md) for the full picture.

---

## Connectors

Nexplane ships connectors for:

**Cloud:** [AWS](connectors/aws.md) -- [GCP](connectors/gcp.md) -- [Azure](connectors/azure.md) -- [OCI](connectors/oci.md)

**Identity:** [LDAP](connectors/ldap.md) -- [Keycloak](connectors/keycloak.md)

**Secrets:** [HashiCorp Vault](connectors/vault.md)

**Orchestration:** [Kubernetes](connectors/kubernetes.md)

**Hosts:** [SSH](connectors/ssh.md) -- [WinRM](connectors/winrm.md)

**Databases:** [PostgreSQL](connectors/postgres.md) -- [Redis](connectors/redis.md) -- [MongoDB](connectors/mongodb.md)

---

## Need Help?

- Browse the [Runbooks](runbooks/index.md) for common operational issues
- Check the [API Reference](api/index.md) for integration docs
- Email [hello@nexplane.ai](mailto:hello@nexplane.ai)
