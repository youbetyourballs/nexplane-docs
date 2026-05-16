# Agent on Windows

This page covers Windows-specific details for the Nexplane Agent.

## Supported Platform

Windows amd64 only. The agent binary is `nexplane-agent-windows-amd64-{VERSION}.exe`.

!!! note "AWS Free Tier"
    AWS Free Tier accounts cannot launch Windows EC2 instances. Windows Server AMIs require a paid AWS account.

## Installation

```powershell
# Download current version
$version = (Invoke-WebRequest -Uri "https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/version").Content.Trim()
New-Item -ItemType Directory -Force "C:\nexplane"
Invoke-WebRequest -Uri "https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/nexplane-agent-windows-amd64-${version}.exe" `
  -OutFile "C:\nexplane\nexplane-agent.exe"

# Install as Windows Service
New-Service -Name "NexplaneAgent" `
  -BinaryPathName "C:\nexplane\nexplane-agent.exe --mode service --poll-interval 30s --control-plane https://nexplane.acme.example:8000 --secret <your-secret>" `
  -DisplayName "Nexplane Agent" `
  -StartupType Automatic
Start-Service NexplaneAgent
```

## Service Management

```powershell
# Check status
Get-Service NexplaneAgent

# View logs
Get-EventLog -LogName Application -Source NexplaneAgent -Newest 50

# Restart
Restart-Service NexplaneAgent

# Stop
Stop-Service NexplaneAgent
```

## Windows Command Packages

### winpatch

Applies Windows updates using the Windows Update Agent (WUA) COM API:

- `apply_windows_patches` — target specific KB article numbers or all pending updates; schedule reboot if required
- `audit_windows_patch_status` — enumerate pending updates without applying

### winharden

Windows security hardening suite:

| Area | Controls |
|------|---------|
| LAPS | Local Administrator Password Solution configuration |
| Credential Guard | Enable/configure Windows Credential Guard |
| PowerShell | Constrained Language Mode enforcement |
| AppLocker | Application allowlist policy |
| SMB | SMB signing enforcement, disable SMBv1 |
| BitLocker | Enable BitLocker drive encryption |
| Windows Firewall | Configure inbound/outbound rules |
| TLS | Disable TLS 1.0/1.1, configure cipher suites |
| RDP | NLA enforcement, session timeout, encryption level |
| Audit Policy | Configure Windows audit policy |
| Registry | Apply security-relevant registry hardening values |

### IP Migration (Windows)

The `changip` package supports Windows IP changes via `netsh`:

- `change_ip` with `tailscale` method — change IP while staying reachable via Tailscale overlay
- `change_ip` with `commit_timer` method — dead man's switch; `pending_rollback.json` written to disk, restored on restart if probe fails
- `change_ip_rollback` — restore full pre-change network state

## Self-Update on Windows

The Windows agent checks for updates on startup and downloads the new binary if one is available. However, the atomic re-exec used on Linux (`syscall.Exec`) is not supported on Windows. The agent logs a message indicating the new version is ready and requires a service restart to apply:

```
New version 0.2.0 downloaded to C:\nexplane\nexplane-agent-0.2.0.exe. Restart the NexplaneAgent service to apply.
```

Restart the service via the Windows Service Manager or PowerShell:

```powershell
Restart-Service NexplaneAgent
```
