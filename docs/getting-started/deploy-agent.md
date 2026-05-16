# Deploy the Agent

The Nexplane Agent is a cross-platform Go binary that runs on managed machines and executes signed commands dispatched by the control plane. No inbound SSH or firewall rules required — the agent polls outbound.

## How It Works

1. The agent starts and registers itself with the control plane
2. The managed machine appears automatically as an Asset in inventory
3. The agent polls `GET /agent/jobs/next` in a long-poll loop
4. When a job arrives, the agent verifies the HMAC-SHA256 signature, executes the command, and posts the result back
5. The agent self-updates: on startup it checks the S3 `version` file and atomically replaces itself if behind

## Step 1: Generate an Agent Secret

In the Nexplane UI, go to **Settings → Agent Configuration** (admin only) and click **Generate Secret**. Copy the secret — it is shown once. Store it securely.

## Step 2: Install the Agent

The **Deploy Agent** panel in Settings pre-fills the install commands with your control plane URL, secret, and current version.

=== "Linux (one-liner)"

    ```bash
    VERSION=$(curl -fsSL https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/version)
    curl -fsSL "https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/nexplane-agent-linux-amd64-${VERSION}" \
      -o nexplane-agent && chmod +x nexplane-agent
    ./nexplane-agent \
      --control-plane https://nexplane.acme.example:8000 \
      --secret sk-agent-<your-secret> \
      --mode service
    ```

=== "Linux (systemd service)"

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

    [Install]
    WantedBy=multi-user.target
    ```

    ```bash
    systemctl enable --now nexplane-agent
    ```

=== "Windows (PowerShell)"

    ```powershell
    New-Service -Name "NexplaneAgent" `
      -BinaryPathName "C:\nexplane\nexplane-agent.exe --mode service --poll-interval 30s --control-plane https://nexplane.acme.example:8000 --secret <your-secret>" `
      -StartupType Automatic
    Start-Service NexplaneAgent
    ```

## Step 3: Verify Registration

After the agent starts, it registers itself and appears as an Asset in **Asset Inventory** within a few seconds. The asset shows hostname, OS, agent version, and last-seen timestamp.

## Flag / Environment Variable Reference

| Flag | Env var | Default | Description |
|------|---------|---------|-------------|
| `--control-plane` | `NP_CONTROL_PLANE` | (required) | Control plane URL |
| `--secret` | `NP_SECRET` | (required) | Agent HMAC secret |
| `--mode` | `NP_MODE` | `service` | `service` (persistent) or `ephemeral` (run once) |
| `--hostname` | `NP_HOSTNAME` | OS hostname | Override the registered hostname |
| `--poll-interval` | `NP_POLL_INTERVAL` | `30s` | How often to poll for jobs |

## Agent Binary Distribution

Binaries are published to a public S3 bucket after each build:

```
https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/
  nexplane-agent-linux-amd64-{VERSION}
  nexplane-agent-linux-arm64-{VERSION}
  nexplane-agent-windows-amd64-{VERSION}.exe
  + .sha256 sidecar for each
  version  (plain text: current version string)
```

For self-hosted distributions, set `NEXPLANE_AGENT_DOWNLOAD_URL` in the backend environment.

## Next Steps

- [Agent command packages reference](../agent/commands.md)
- [Agent self-update](../agent/self-update.md)
- [Windows agent](../agent/windows.md)
