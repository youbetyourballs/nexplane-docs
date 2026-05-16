# WinRM Connector

The WinRM connector uses the `pywinrm` library to connect to Windows hosts via Windows Remote Management. Like the SSH connector, it does not execute arbitrary PowerShell strings. All operations are typed and validated against an allowlist of permitted PowerShell commands before execution.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `windows-servers`) |
| Host | string | Yes | Hostname or IP address of the target Windows host |
| Port | integer | No | WinRM port (default: 5986 for HTTPS, 5985 for HTTP) |
| Username | string | Yes | Windows username (local or domain, e.g., `CORP\nexplane` or `nexplane`) |
| Password | string | Yes | Account password |
| Use HTTPS | boolean | No | Use WinRM over HTTPS (default: true) |
| CA Certificate | string | No | PEM-encoded CA certificate for TLS validation |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate Local User Password | Sets a new password for a local Windows user | No rollback (old password is not stored) |
| Disable Local User Account | Runs `Disable-LocalUser` | `Enable-LocalUser` |
| Enable Local User Account | Runs `Enable-LocalUser` | `Disable-LocalUser` |
| Disable Windows Service | Runs `Stop-Service` + `Set-Service -StartupType Disabled` | `Set-Service -StartupType Automatic` + `Start-Service` |
| Enable Windows Service | Runs `Set-Service -StartupType Automatic` + `Start-Service` | `Stop-Service` + `Set-Service -StartupType Disabled` |
| Set File ACL | Applies an ACL entry to a file or directory | Restore previous ACL |
| Apply CIS Hardening Profile | Applies a named set of registry and service hardening steps | Restore previous values where possible |

## Allowlist Enforcement

The WinRM connector does not accept freeform PowerShell. Every action generates a specific, parameterized PowerShell command. The parameters are validated before the command is assembled:

- Usernames are checked against `^[a-zA-Z0-9_\-. ]+$`
- Service names are checked against `^[a-zA-Z0-9_\-. ]+$`
- File paths must begin with a drive letter and colon (`C:\...`) and cannot contain shell metacharacters

For example, the `Disable Local User Account` action generates:

```powershell
Disable-LocalUser -Name "validated_username"
```

The username is double-quoted and validated before insertion. PowerShell injection via the username field is not possible.

## Minimum Permissions Required

The WinRM account needs:

- Local Administrator group membership, or
- Delegated WinRM access with specific permissions granted via `winrm configsddl`

For service management, file ACL changes, and local user operations, Local Administrator is the simplest option. Least-privilege WinRM setups are possible but require significant Windows configuration.

## Enabling WinRM on Target Hosts

WinRM must be enabled on target Windows hosts before the connector can connect:

```powershell
# Enable WinRM with HTTPS (recommended)
Enable-PSRemoting -Force
winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Hostname="$env:COMPUTERNAME"; CertificateThumbprint="<cert_thumbprint>"}
```

For lab/testing with self-signed certificates:

```powershell
Enable-PSRemoting -Force
# The connector's "Use HTTPS" field should be false for HTTP-only setups
```

## Known Limitations

- WinRM HTTPS requires a valid TLS certificate on the target host. Self-signed certificates require the CA certificate to be provided in the connector configuration.
- The connector connects to a single host per connector instance. For fleet management, use the Nexplane agent on Windows instead.
- Domain-joined hosts may require Kerberos authentication in some configurations. The connector currently supports only basic authentication with HTTPS. Kerberos support is on the roadmap.
- CIS hardening profiles apply Windows-specific registry changes and service configurations. Test profiles in a non-production environment before applying to production.
