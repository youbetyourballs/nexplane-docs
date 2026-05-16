# Agent Installation & Configuration

## Prerequisites

- Network access to the Nexplane control plane URL (outbound HTTP/HTTPS)
- An agent secret generated in **Settings → Agent Configuration**

## Download

The **Deploy Agent** panel in Nexplane Settings pre-fills all install commands with your control plane URL, secret, and current version.

=== "Linux (one-liner, x86_64)"

    ```bash
    VERSION=$(curl -fsSL https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/version)
    curl -fsSL "https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/nexplane-agent-linux-amd64-${VERSION}" \
      -o /usr/local/bin/nexplane-agent
    chmod +x /usr/local/bin/nexplane-agent
    ```

=== "Linux (ARM64)"

    ```bash
    VERSION=$(curl -fsSL https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/version)
    curl -fsSL "https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/nexplane-agent-linux-arm64-${VERSION}" \
      -o /usr/local/bin/nexplane-agent
    chmod +x /usr/local/bin/nexplane-agent
    ```

=== "Windows (PowerShell)"

    ```powershell
    $version = (Invoke-WebRequest -Uri "https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/version").Content.Trim()
    Invoke-WebRequest -Uri "https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/nexplane-agent-windows-amd64-${version}.exe" `
      -OutFile "C:\nexplane\nexplane-agent.exe"
    ```

## Run as Persistent Service

=== "Linux (systemd)"

    Create `/etc/systemd/system/nexplane-agent.service`:

    ```ini
    [Unit]
    Description=Nexplane Agent
    After=network.target

    [Service]
    ExecStart=/usr/local/bin/nexplane-agent \
      --control-plane https://nexplane.acme.example:8000 \
      --secret sk-agent-<your-secret> \
      --mode service \
      --poll-interval 30s
    Restart=on-failure
    RestartSec=5s

    [Install]
    WantedBy=multi-user.target
    ```

    ```bash
    systemctl daemon-reload
    systemctl enable --now nexplane-agent
    ```

=== "Windows (PowerShell)"

    ```powershell
    New-Service -Name "NexplaneAgent" `
      -BinaryPathName "C:\nexplane\nexplane-agent.exe --mode service --poll-interval 30s --control-plane https://nexplane.acme.example:8000 --secret <your-secret>" `
      -StartupType Automatic
    Start-Service NexplaneAgent
    ```

## Flag / Environment Variable Reference

| Flag | Env Var | Default | Description |
|------|---------|---------|-------------|
| `--control-plane` | `NP_CONTROL_PLANE` | (required) | Control plane URL |
| `--secret` | `NP_SECRET` | (required) | HMAC secret from Settings |
| `--mode` | `NP_MODE` | `service` | `service` (persistent poll) or `ephemeral` (run once and exit) |
| `--hostname` | `NP_HOSTNAME` | OS hostname | Override the registered hostname |
| `--poll-interval` | `NP_POLL_INTERVAL` | `30s` | Long-poll interval |

## Verify Registration

After the agent starts, it appears as an Asset in **Asset Inventory** within seconds. The asset shows hostname, OS, agent version, and last-seen timestamp.

## Self-Hosted Binary Distribution

If you host agent binaries internally, set `NEXPLANE_AGENT_DOWNLOAD_URL` in the backend environment to point to your distribution server. The Settings deploy panel will generate install commands using your URL.
