# Nexplane

**The control plane for security execution.**

Nexplane connects intent to action across your infrastructure — enabling security engineering and architecture teams to execute infrastructure changes safely, without routing work through sysadmin, network admin, or SRE queues.

> *Execute with Confidence*

---

## What It Does

Security teams identify issues and need to act on them: rotate compromised keys, isolate a compromised endpoint, tighten firewall rules after a scan, deploy an EDR sensor to unprotected hosts, enforce MFA, offboard a departing employee. Today, all of that flows through tickets.

Nexplane gives security teams a governed execution layer:

- **Change Requests** — safety-reviewed, approval-gated, audited, with automatic rollback
- **Projects** — group related change requests into a sequenced plan with dependency tracking
- **AI Planning Assistant** — describe your goal, get a structured change plan referencing your actual asset inventory
- **Composable Runbooks** — chain change types into reusable multi-step workflows with conditional branching and human checkpoints
- **Asset Inventory** — servers, cloud accounts, firewalls, identities, applications — discoverable via connectors
- **Connectors** — 38+ integrations spanning cloud, identity, EDR, IaC, ticketing, and observability
- **Nexplane Agent** — a cross-platform Go binary that runs on managed machines, reaches out to the control plane, and executes signed commands — no inbound SSH required
- **Incident Response Playbooks** — pre-defined fast-path workflows for host isolation, account lockdown, evidence preservation, and phishing response
- **Vulnerability Remediation Pipeline** — close the loop between scanner findings and automated remediation
- **Compliance & Governance** — CIS benchmark enforcement, drift detection, change freeze windows, audit evidence collection

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│  React Frontend  (Vite + TypeScript + Tailwind CSS)                      │
│                                                                          │
│  Dashboard · Projects · Change Requests · Approvals · Asset Inventory   │
│  Connectors · Settings · Runbooks · Compliance · Incident Response       │
│  Vulnerability Remediation · Access Reviews · Maintenance Windows        │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │ HTTP/REST (localhost only)
┌───────────────────────────▼─────────────────────────────────────────────┐
│  FastAPI Backend  (Python 3.12)                                           │
│                                                                          │
│  Safety Engine · Planning Engine · AI Service · Audit Service            │
│  Secrets Service (Fernet AES-256, HSM/Vault-swappable)                   │
│  RunbookExecutor · IRExecutor · FleetExecutor · IaCExecutor              │
│  VulnRemediationEngine · IdentityResolver · DriftDetection               │
│                                                                          │
│  Connectors — change actions + ingest (38+ connectors)                   │
│  aws · azure · gcp · cloudflare · okta · paloalto · ssh                 │
│  active_directory · entra_id · crowdstrike · tenable · kubernetes        │
│  tailscale · terraform_local · ansible_local · ...                      │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │ SQLAlchemy async
┌───────────────────────────▼─────────────────────────────────────────────┐
│  PostgreSQL 16                                                            │
└──────────────────────────────────────────────────────────────────────────┘

                    ┌──────────────────────────────────────┐
                    │  Nexplane Agent (Go)                  │
                    │  linux/amd64 · arm64 · windows/amd64 │
                    │                                       │
                    │  Outbound poll only —                 │
                    │  no inbound SSH needed                │
                    └────────────┬──────────────────────────┘
                                 │ long-poll HTTP (outbound)
                                 └──► /agent/jobs/next
```

---

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, TypeScript, Vite, Tailwind CSS, TanStack Query v5, React Router v6 |
| Backend | Python 3.12, FastAPI, SQLAlchemy 2.0 async, Pydantic v2 |
| Database | PostgreSQL 16 |
| Auth | JWT + bcrypt |
| AI | Anthropic Claude + OpenAI (multi-provider, default configurable) |
| Secrets | Fernet AES-256 with versioning; abstracted for HSM/Vault swap-out |
| Agent | Go, cross-platform (linux/amd64, linux/arm64, windows/amd64) |
| IaC Runtime | Terraform CLI, Ansible + community.aws, AWS session-manager-plugin |
| Deployment | Docker Compose |

---

## Get Started

- [Install Nexplane](getting-started/installation.md)
- [Connect a cloud account](getting-started/connect-cloud.md)
- [Create your first change request](getting-started/first-change-request.md)
- [Deploy the agent](getting-started/deploy-agent.md)
