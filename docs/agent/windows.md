# Agent on Windows

This page covers Windows-specific installation details, service management, and troubleshooting for the Nexplane agent.

## Service Installation

Run the installer from an elevated PowerShell prompt:

```powershell
.\nexplane-agent-windows-amd64.exe install `
  --control-plane https://nexplane.example.com:8000 `
  --token YOUR_ENROLLMENT_TOKEN
```

The installer:
1. Copies the binary to `C:\Program Files\NexplaneAgent\nexplane-agent.exe`
2. Writes config to `C:\ProgramData\NexplaneAgent\config.yaml`
3. Stores the client certificate at `C:\ProgramData\NexplaneAgent\client.crt`
4. Creates a Windows Service named `NexplaneAgent` using the Local System account
5. Sets the service to `Automatic` startup and starts it

## Managing the Service

```powershell
# Check status
Get-Service NexplaneAgent

# Start / stop / restart
Start-Service NexplaneAgent
Stop-Service NexplaneAgent
Restart-Service NexplaneAgent

# View recent logs
Get-EventLog -LogName Application -Source NexplaneAgent -Newest 50

# Follow logs in real time (requires PowerShell 5.1+)
Get-EventLog -LogName Application -Source NexplaneAgent -Newest 1 -Wait
```

## Firewall Configuration

The agent initiates outbound HTTPS connections to the control plane. On Windows Server with Windows Firewall enabled, you may need to allow outbound traffic on the control plane port:

```powershell
New-NetFirewallRule `
  -DisplayName "Nexplane Agent - Control Plane" `
  -Direction Outbound `
  -Protocol TCP `
  -RemotePort 8000 `
  -Action Allow
```

Adjust the port to match your control plane configuration.

## Supported Operations on Windows

| Operation | Notes |
|---|---|
| Disable/Enable Local User | Uses `Disable-LocalUser` / `Enable-LocalUser` |
| Rotate Local Password | Uses `Set-LocalUser -Password` |
| Disable/Enable Service | Uses `Stop-Service` + `Set-Service -StartupType` |
| Set File ACL | Uses `Set-Acl` with a validated ACL entry |
| Apply CIS Profile | Applies registry, service, and audit policy changes per CIS Windows Server benchmark |
| Set Registry Value | Uses `Set-ItemProperty` with validated path and value |

## CIS Hardening on Windows

CIS Windows Server profiles apply changes to:

- Windows Firewall rules
- Audit policy settings
- Local security policy (account lockout, password complexity)
- Registry-based security settings
- Disabled unnecessary services (Telnet server, FTP server, SNMP, etc.)

CIS Level 1 profiles are appropriate for most production servers. Level 2 profiles include more restrictive settings that may break some workloads -- test in staging before applying to production.

## Troubleshooting

**Service fails to start after installation:**

Check the Event Log for errors:

```powershell
Get-EventLog -LogName Application -Source NexplaneAgent -Newest 20 | Format-List
```

Common causes:
- Network access to control plane blocked by firewall
- Enrollment token already used (tokens are single-use)
- Clock skew between the agent host and control plane (mTLS certificate validation is time-sensitive -- ensure NTP is synchronized)

**Agent registered but tasks are not executing:**

The agent runs as Local System by default. If a task requires access to a network share or domain resource, the service may need to run as a domain service account. Change the service logon account in Services MMC (`services.msc`) and restart.

**Uninstalling:**

```powershell
& "C:\Program Files\NexplaneAgent\nexplane-agent.exe" uninstall
```

This stops the service, removes the service definition, and deletes all agent files. It does not affect Windows Event Log entries from the agent's operation.
