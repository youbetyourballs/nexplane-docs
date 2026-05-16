# Deploy the Agent

The Nexplane agent is a Go binary that runs on Linux, Windows, and macOS hosts. It handles local changes that cannot be made through a cloud API -- OS hardening, local user management, file permission changes, and package updates.

## How the Agent Works

The agent:

1. Registers with the control plane using a one-time enrollment token
2. Establishes a persistent mTLS connection to the control plane
3. Polls for assigned change requests
4. Executes approved changes using a built-in allowlist of permitted operations
5. Reports results back to the control plane

The agent never executes arbitrary commands. All operations are typed (e.g., `set_file_permission`, `disable_service`, `rotate_local_password`) and defined at compile time.

## Step 1: Generate an Enrollment Token

In the Nexplane UI, go to **Settings > Agents > New Enrollment Token**.

Give the token a label (e.g., `web-server-01`) and click **Generate**. Copy the token -- it is shown only once.

## Step 2: Install the Agent

=== "Linux (systemd)"

    ```bash
    curl -fsSL https://github.com/youbetyourballs/nexplane/releases/latest/download/nexplane-agent-linux-amd64 \
      -o /usr/local/bin/nexplane-agent
    chmod +x /usr/local/bin/nexplane-agent
    ```

    Create the systemd service:

    ```bash
    nexplane-agent install \
      --control-plane https://your-nexplane-host:8000 \
      --token YOUR_ENROLLMENT_TOKEN
    ```

    Start the service:

    ```bash
    systemctl enable --now nexplane-agent
    ```

=== "Windows"

    Download `nexplane-agent-windows-amd64.exe` from the releases page and run as Administrator:

    ```powershell
    .\nexplane-agent-windows-amd64.exe install `
      --control-plane https://your-nexplane-host:8000 `
      --token YOUR_ENROLLMENT_TOKEN
    ```

    This installs and starts a Windows Service named `NexplaneAgent`.

=== "macOS"

    ```bash
    curl -fsSL https://github.com/youbetyourballs/nexplane/releases/latest/download/nexplane-agent-darwin-arm64 \
      -o /usr/local/bin/nexplane-agent
    chmod +x /usr/local/bin/nexplane-agent

    nexplane-agent install \
      --control-plane https://your-nexplane-host:8000 \
      --token YOUR_ENROLLMENT_TOKEN
    ```

    This installs a launchd plist at `/Library/LaunchDaemons/ai.nexplane.agent.plist`.

## Step 3: Verify Registration

After the agent starts, return to **Settings > Agents** in the UI. The host should appear with status **Registered** within 30 seconds.

Click the agent row to see:

- Hostname and OS version
- Agent version
- Last heartbeat timestamp
- Assigned change requests

## Step 4: Run a Test Change

With the agent registered, you can assign it a hardening change. Go to **Change Requests > New Change Request**, select change type **Hardening - Disable Unused Services**, and select the registered agent as the target.

Submit, approve, and execute. The agent will receive the task, execute it, and report back with a success or failure status.

## Troubleshooting

If the agent does not appear in the UI after 60 seconds, check the agent logs:

```bash
# Linux
journalctl -u nexplane-agent -f

# Windows
Get-EventLog -LogName Application -Source NexplaneAgent -Newest 50

# macOS
log stream --predicate 'subsystem == "ai.nexplane.agent"'
```

See also: [Agent Registration Failure Runbook](../runbooks/agent-registration-failure.md)
