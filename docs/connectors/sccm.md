# SCCM / MECM

The SCCM connector manages Microsoft Endpoint Configuration Manager (MECM/SCCM) application deployments and client actions.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| SCCM Server Hostname / IP | Yes | SCCM server address |
| Username | Yes | `DOMAIN\user` or `user@domain` |
| Password | Yes | Password |
| Site Code | Yes | 3-letter SCCM site code (e.g. `PS1`) |
| Verify SSL | No | Verify SSL certificate (default: false) |
| Use NTLM | No | Use NTLM authentication (default: true) |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `deploy_application` | Deploy an SCCM application to a device or user collection | Remove the deployment |
| `remove_deployment` | Remove an application deployment from a collection | N/A |
| `run_script` | Run a pre-approved SCCM script (by GUID) against a collection — no freeform execution | N/A |
| `trigger_client_action` | Trigger a client policy action on all devices in a collection (`MachinePolicyRetrieve`, `HardwareInventory`, `SoftwareInventory`, `UpdateDeploymentReEval`) | N/A |
| `collect_inventory` | Pull hardware and software inventory for a specific device (read-only) | N/A |
