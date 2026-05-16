# The Nexplane Agent

The Nexplane Agent is a cross-platform Go binary that runs on managed machines and executes commands dispatched by the control plane. It reverses the connection direction: the agent calls out to the control plane — no inbound SSH, no VPN, no open firewall ports required.

## Why the Agent Exists

Cloud APIs (AWS, Azure, GCP) can manage cloud resources but cannot reach inside the operating system of a running instance. Tasks like patching packages, hardening SSH configuration, rotating local credentials, or collecting forensic evidence require executing commands on the host. The Nexplane Agent handles this without requiring the control plane to initiate a connection to the host.

## Outbound Poll Model

```
Managed Host                     Nexplane Control Plane
─────────────────────────────────────────────────────
Agent starts
  │
  ├─► POST /agent/register      ← registers with HMAC secret
  │
  └─► GET /agent/jobs/next      ← long-poll (waits for a job)
        │
        ← job payload arrives (signed with HMAC-SHA256)
        │
  agent verifies signature
  agent executes command
        │
  ├─► POST /agent/result        ← posts result
        │
  └─► GET /agent/jobs/next      ← immediately polls again
```

The agent never opens a listening port. All communication is outbound HTTP from the agent to the control plane.

## HMAC-SHA256 Job Signing

Every job dispatched to an agent is signed with an HMAC-SHA256 key derived from the agent secret (configured in **Settings → Agent Configuration**). The agent verifies the signature before executing any command. Jobs with invalid signatures are rejected.

## Self-Registration

When the agent starts for the first time, it registers itself at `POST /agent/register`. The managed machine appears automatically as an Asset in Nexplane's inventory with its hostname, OS, and agent version. No manual asset creation required.

## Self-Update

On startup, the agent fetches the `version` file from S3, downloads and SHA256-verifies the new binary if behind, and atomically replaces itself via `os.Rename` + `syscall.Exec`. See [Self-Update](../agent/self-update.md) for details.

## Command Packages

The agent executes typed commands — no arbitrary shell. All supported operations are registered at compile time. Commands are grouped into 15 packages covering patching, hardening, IaC, fleet operations, forensics, compliance, database administration, and more.

See [Command Packages](../agent/commands.md) for the full reference.

## Platforms

| Platform | Architecture |
|----------|-------------|
| Linux | amd64, arm64 |
| Windows | amd64 |
