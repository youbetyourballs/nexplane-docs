# Agent

The Nexplane agent is a statically compiled Go binary that runs as a system service on managed hosts. It handles local operations -- OS hardening, service management, file permissions, local user accounts -- that cannot be reached through a cloud API.

## Design Principles

**No arbitrary command execution.** The agent does not accept shell strings. Every operation is a typed struct with validated parameters. The set of permitted operations is fixed at compile time. An attacker who compromises the control plane cannot use the agent to run arbitrary commands.

**Agent-initiated connections.** The agent polls the control plane for work over mTLS. The control plane never connects outbound to agents. Agents work behind NAT without firewall changes.

**Minimal privilege.** On Linux, the agent runs as `root` only for operations that require it (service management, file ownership changes). On Windows, it runs as a Local System service.

## Enrollment

When the agent starts for the first time, it:

1. Generates an RSA-2048 key pair locally
2. Sends a Certificate Signing Request to the control plane along with the enrollment token
3. The control plane validates the token (single-use), signs the CSR, and returns a certificate
4. The agent stores the signed certificate and private key on disk
5. The enrollment token is invalidated

All subsequent communication uses the client certificate for mutual TLS authentication.

## Polling Loop

The agent maintains a persistent HTTPS connection to the control plane and polls `/agent/tasks` every 5 seconds. When a task is assigned:

1. The agent fetches the full task payload
2. Validates the task type against its allowlist
3. Executes the operation
4. Posts the result back to `/agent/tasks/{task_id}/result`

The polling interval is configurable with `--poll-interval`.

## Allowed Operations

The agent's operation allowlist includes:

| Operation | Description |
|---|---|
| `set_file_permission` | Set mode and ownership on a file or directory |
| `disable_service` | Stop and disable a system service |
| `enable_service` | Enable and start a system service |
| `set_sysctl` | Write a kernel parameter via sysctl |
| `rotate_local_password` | Set a local user account password |
| `lock_local_account` | Lock a local user account |
| `unlock_local_account` | Unlock a local user account |
| `install_package` | Install a named package via the system package manager |
| `remove_package` | Remove a named package |
| `set_file_content` | Write a validated configuration file |
| `run_hardening_profile` | Apply a named CIS benchmark profile |

Each operation validates its parameters before executing. For example, `set_file_permission` rejects paths outside a configurable allowed-path list.

## Pre-change Snapshots

Before executing destructive operations, the agent takes a snapshot of the affected state:

- File content and permissions are recorded before `set_file_content` or `set_file_permission`
- Service state is recorded before `disable_service`
- Package version is recorded before `remove_package`

This snapshot is sent to the control plane and stored with the change record, enabling rollback without requiring a backup restore.

## Logs

Agent logs are written to the system journal (Linux), Windows Event Log, or macOS Unified Log depending on platform. See the platform-specific pages:

- [Windows](../agent/windows.md)
- [macOS](../agent/macos.md)
