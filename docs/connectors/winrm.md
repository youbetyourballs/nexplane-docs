# WinRM (Windows Remote Management)

The WinRM connector connects to Windows hosts via Windows Remote Management using `pywinrm`. Its primary purpose is bootstrapping the Nexplane Agent onto Windows hosts that don't yet have it installed.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Hostname / IP | Yes | Target Windows host |
| Username | Yes | Windows username |
| Password | Yes | Windows password |
| Port | No | WinRM port (default: 5985) |
| Use SSL | No | Use HTTPS on port 5986 (default: false) |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `check_prerequisites` | Verify OS version (Windows Server 2016+), disk space, .NET version, and WinRM accessibility | N/A |
| `download_agent` | Download the Nexplane Agent binary to `C:\nexplane\nexplane-agent.exe` via `Invoke-WebRequest` | Remove the downloaded binary |
| `install_agent` | Create and start the nexplane-agent Windows Service | Uninstall the service |
| `uninstall_agent` | Stop and delete the nexplane-agent Windows Service | N/A |
| `execute_template` | Execute a pre-approved PowerShell template by name — no freeform scripts permitted | N/A |
| `collect_diagnostics` | Collect System event log, running services list, and systeminfo from the host | N/A |

!!! tip
    Once the Nexplane Agent is installed via WinRM, further management can go through the [Nexplane Agent Connector](nexplane-agent.md) which offers a much richer command set without requiring WinRM.
