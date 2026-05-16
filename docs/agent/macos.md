# Agent on macOS

This page covers macOS-specific installation details, service management, and troubleshooting for the Nexplane agent.

## Installation

The agent installs as a launchd daemon that runs at system boot.

```bash
# Apple Silicon
curl -fsSL https://github.com/youbetyourballs/nexplane/releases/latest/download/nexplane-agent-darwin-arm64 \
  -o /usr/local/bin/nexplane-agent
chmod +x /usr/local/bin/nexplane-agent

# Intel
curl -fsSL https://github.com/youbetyourballs/nexplane/releases/latest/download/nexplane-agent-darwin-amd64 \
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

## Supported Operations on macOS

| Operation | Notes |
|---|---|
| Disable/Enable Service | Uses `launchctl` to load/unload service plists |
| Set File Permission | Uses `chmod` and `chown` |
| Rotate Local User Password | Uses `dscl` to set the password |
| Lock Local User Account | Uses `dscl` to set `AuthenticationAuthority` to disable login |

!!! note "macOS agent usage"
    The macOS agent is primarily used for managing developer laptops and macOS-based build agents. For server fleets, Linux or Windows agents are more commonly used.

## Uninstalling

```bash
sudo nexplane-agent uninstall
```

This unloads the launchd daemon, removes the plist, and deletes the config and certificates.
