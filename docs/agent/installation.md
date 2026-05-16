# Agent Installation

The Nexplane agent is a statically compiled Go binary with no runtime dependencies. It installs as a system service and runs in the background, polling for assigned change requests.

## Supported Platforms

| Platform | Architecture | Minimum OS Version |
|---|---|---|
| Linux | amd64, arm64 | Any systemd-based distro (RHEL 7+, Ubuntu 18.04+, Debian 10+) |
| Windows | amd64 | Windows Server 2016, Windows 10 |
| macOS | arm64 (Apple Silicon) | macOS 12 (Monterey) |
| macOS | amd64 (Intel) | macOS 12 (Monterey) |

## Prerequisites

- Network access from the agent host to the Nexplane control plane (port 8000 by default)
- Root or Administrator privileges for installation
- An enrollment token generated in the Nexplane UI (Settings > Agents > New Enrollment Token)

## Linux Installation

### Download

```bash
# Detect architecture
ARCH=$(uname -m)
if [ "$ARCH" = "aarch64" ]; then ARCH="arm64"; else ARCH="amd64"; fi

# Download
curl -fsSL "https://github.com/youbetyourballs/nexplane/releases/latest/download/nexplane-agent-linux-${ARCH}" \
  -o /usr/local/bin/nexplane-agent
chmod +x /usr/local/bin/nexplane-agent
```

### Install as Systemd Service

```bash
nexplane-agent install \
  --control-plane https://nexplane.example.com:8000 \
  --token YOUR_ENROLLMENT_TOKEN \
  --poll-interval 5s
```

This command:
1. Generates a key pair for mTLS authentication
2. Sends a CSR to the control plane with the enrollment token
3. Receives and stores a signed client certificate
4. Writes `/etc/nexplane-agent/config.yaml`
5. Creates and enables `/etc/systemd/system/nexplane-agent.service`
6. Starts the service

### Verify Installation

```bash
systemctl status nexplane-agent
journalctl -u nexplane-agent -n 50
```

## Windows Installation

Download from the releases page and run as Administrator:

```powershell
.\nexplane-agent-windows-amd64.exe install `
  --control-plane https://nexplane.example.com:8000 `
  --token YOUR_ENROLLMENT_TOKEN
```

The installer creates a Windows Service named `NexplaneAgent` and starts it automatically.

See [Windows-specific notes](windows.md) for firewall and WMI configuration.

## macOS Installation

```bash
# Apple Silicon
curl -fsSL https://github.com/youbetyourballs/nexplane/releases/latest/download/nexplane-agent-darwin-arm64 \
  -o /usr/local/bin/nexplane-agent

# Intel Mac
curl -fsSL https://github.com/youbetyourballs/nexplane/releases/latest/download/nexplane-agent-darwin-amd64 \
  -o /usr/local/bin/nexplane-agent

chmod +x /usr/local/bin/nexplane-agent

nexplane-agent install \
  --control-plane https://nexplane.example.com:8000 \
  --token YOUR_ENROLLMENT_TOKEN
```

See [macOS-specific notes](macos.md) for Gatekeeper and Full Disk Access setup.

## Uninstalling

```bash
# Linux
nexplane-agent uninstall

# Windows (run as Administrator)
nexplane-agent-windows-amd64.exe uninstall

# macOS
nexplane-agent uninstall
```

The uninstall command stops the service, removes the service definition, and deletes the agent configuration and certificates. It does not affect any changes that were previously executed by the agent.

## Configuration File

The agent configuration is stored at:

- Linux: `/etc/nexplane-agent/config.yaml`
- Windows: `C:\ProgramData\NexplaneAgent\config.yaml`
- macOS: `/etc/nexplane-agent/config.yaml`

```yaml
control_plane: https://nexplane.example.com:8000
poll_interval: 5s
log_level: info
cert_path: /etc/nexplane-agent/client.crt
key_path: /etc/nexplane-agent/client.key
```

Edit this file and restart the service to change configuration after installation.
