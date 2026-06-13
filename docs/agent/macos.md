# Agent on macOS

This page covers macOS-specific installation details, service management, and troubleshooting for the Nexplane agent.

## Installation

The agent installs as a launchd daemon that runs at system boot. Only the **Apple Silicon (`darwin/arm64`)** binary is published — it is built on macOS hardware and pushed to the same S3 bucket as the Linux and Windows binaries.

```bash
# Apple Silicon (arm64)
curl -fsSL https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/nexplane-agent-darwin-arm64-$(curl -fsSL https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/version) \
  -o /usr/local/bin/nexplane-agent
chmod +x /usr/local/bin/nexplane-agent

sudo nexplane-agent install \
  --control-plane https://nexplane.example.com:8000 \
  --token YOUR_ENROLLMENT_TOKEN
```

The installer:
1. Writes config to `/etc/nexplane-agent/config.yaml`
2. Stores the client certificate at `/etc/nexplane-agent/client.crt`
3. Creates a launchd plist at `/Library/LaunchDaemons/ai.nexplane.agent.plist`
4. Loads and starts the daemon

## Gatekeeper

macOS Gatekeeper may block the agent binary on first run because it is not notarized through the Mac App Store. To allow execution:

```bash
sudo xattr -r -d com.apple.quarantine /usr/local/bin/nexplane-agent
```

Or go to **System Settings > Privacy & Security** and click **Allow Anyway** after the first blocked execution attempt.

## Managing the Daemon

```bash
# Check status
sudo launchctl list | grep nexplane

# Stop the daemon
sudo launchctl unload /Library/LaunchDaemons/ai.nexplane.agent.plist

# Start the daemon
sudo launchctl load /Library/LaunchDaemons/ai.nexplane.agent.plist

# View logs
log stream --predicate 'subsystem == "ai.nexplane.agent"'

# View recent log entries
log show --predicate 'subsystem == "ai.nexplane.agent"' --last 1h
```

## Full Disk Access

Some hardening operations require Full Disk Access permission. On macOS 12+, you must grant this manually:

1. Open **System Settings > Privacy & Security > Full Disk Access**
2. Click the `+` button and add `/usr/local/bin/nexplane-agent`
3. Restart the daemon:
   ```bash
   sudo launchctl unload /Library/LaunchDaemons/ai.nexplane.agent.plist
   sudo launchctl load /Library/LaunchDaemons/ai.nexplane.agent.plist
   ```

Full Disk Access is required for operations that read or write files outside standard locations (e.g., `/etc/ssh/sshd_config`, `/private/etc/`).

## macOS Change Types

The macOS agent exposes **23 macOS change types** across three areas, all driven by the `macos` command package.

### Posture & encryption

| Change type | Action |
|-------------|--------|
| `macos_filevault_enable` | Enable FileVault full-disk encryption (and report status) |
| `macos_gatekeeper_enable` | Enable Gatekeeper |
| `macos_profiles_install` | Install configuration profiles |
| `macos_defaults_write` | Write a managed `defaults` key |
| `macos_sysinfo` | Collect system information |
| `macos_softwareupdate_install` | Audit and install pending software updates |

### Binary authorization (Santa)

Nexplane installs and manages **Santa** — Google / North Pole Security's binary authorization system — on macOS hosts:

| Change type | Action |
|-------------|--------|
| `macos_santa_install` | Install Santa |
| `macos_santa_rule_add` | Add an allow/block rule (by hash, signing ID, or certificate) |
| `macos_santa_mode_set` | Switch between **monitor** and **lockdown** mode |

Santa rules can also be listed and removed, sync can be triggered, decisions exported, and an individual binary checked against the current ruleset.

!!! note "SIP and the Santa system extension"
    When System Integrity Protection blocks the system extension (for example on an unmanaged EC2 `mac2.metal` host), `macos_santa_install` returns `installed: true, activated: false` rather than failing. The change request still completes and signals that MDM/SIP approval is pending, instead of leaving the CR in an error state.

### Observability & hardening

- **Software inventory** via `brew`, MacPorts, and `pkgutil`
- **CIS compliance audit** — SIP, Gatekeeper, the application firewall (ALF), NTP, SSH, auditd, Santa, and screen lock
- **SSH hardening** and **NTP** configuration via `systemsetup`
- **Syslog forwarding** and **network isolation** via `pfctl`
- **Fleet operations** via `launchctl` and `brew`

## macOS Smoke Coverage

macOS support is validated against an EC2 `mac2.metal` instance on a Dedicated Host across the `MAC_AGENT_BOOTSTRAP`, `MAC_POSTURE_AUDIT`, `MAC_AUTH_HARDENING`, and `MAC_OBSERVABILITY` phases.

!!! note "macOS agent usage"
    The macOS agent is used for managing developer laptops, macOS-based build agents, and Apple Silicon fleets. For Linux and Windows server fleets, the corresponding agent builds are used.

## Uninstalling

```bash
sudo nexplane-agent uninstall
```

This unloads the launchd daemon, removes the plist, and deletes the config and certificates.
