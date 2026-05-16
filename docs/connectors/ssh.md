# SSH Connector

The SSH connector uses the Paramiko library to connect to Linux and Unix hosts over SSH. Unlike a general-purpose SSH client, the connector does not execute arbitrary shell commands. All operations are typed and validated against an allowlist before execution. This means an attacker who compromises the Nexplane control plane cannot use the SSH connector to run arbitrary commands on your hosts.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `web-servers-ssh`) |
| Host | string | Yes | Hostname or IP address of the target host |
| Port | integer | No | SSH port (default: 22) |
| Username | string | Yes | SSH username |
| Password | string | No | SSH password (use either password or private key, not both) |
| Private Key | string | No | PEM-encoded SSH private key |
| Host Key | string | No | Expected SSH host key fingerprint for verification |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate Local User Password | Sets a new password for a local OS user | No rollback (old password is not stored) |
| Lock Local User Account | Runs `usermod -L` to lock a local account | Unlock: `usermod -U` |
| Unlock Local User Account | Runs `usermod -U` to unlock | Lock: `usermod -L` |
| Disable Service | Runs `systemctl disable --now <service>` | `systemctl enable --now <service>` |
| Enable Service | Runs `systemctl enable --now <service>` | `systemctl disable --now <service>` |
| Set File Permission | Runs `chmod` on a specific path | Restore previous permissions |
| Set File Ownership | Runs `chown` on a specific path | Restore previous ownership |
| Apply CIS Hardening Profile | Runs a named set of hardening commands | Restore previous state where possible |

## Allowlist Enforcement

The SSH connector does not accept arbitrary shell commands. Every action is defined in the backend as a Python class that generates the specific command string. Users cannot inject arbitrary shell commands through the Nexplane UI or API.

For example, the `Disable Service` action generates exactly:

```
systemctl disable --now <validated_service_name>
```

The `validated_service_name` is checked against a regex (`^[a-zA-Z0-9_\-.]+$`) before the command is built. Shell metacharacters cannot appear in the service name.

## Minimum Permissions Required

The SSH user must have:

- `sudo` access (passwordless) for service management and file permission changes
- Direct write access to the files being modified, or sudo for file operations

For read-only discovery, a non-sudo user with access to `/etc/passwd`, `/etc/shadow` (read), and systemd socket is sufficient.

## Known Limitations

- The connector connects to a single host per connector instance. To manage a fleet of SSH hosts, create one connector per host, or use the agent (which is better suited for fleet management).
- Host key verification is optional but strongly recommended. Without it, Nexplane is vulnerable to MITM attacks. Set the `Host Key` field to the output of `ssh-keyscan -t ed25519 <host>`.
- The connector does not support jump hosts or SSH ProxyJump. The control plane must have direct network access to the target host.
- CIS hardening profiles apply a fixed set of commands. If a host has custom configuration that conflicts with the profile, some hardening steps may fail. Failures are reported per-step -- the profile continues executing remaining steps.
- Password-based authentication is supported but key-based authentication is strongly recommended for production use.
