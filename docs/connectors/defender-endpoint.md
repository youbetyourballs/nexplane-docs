# Microsoft Defender for Endpoint

The Microsoft Defender for Endpoint connector uses the Microsoft Security Graph API to discover managed machines, ingest alerts and vulnerabilities, and execute endpoint response actions.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Tenant ID | Yes | Azure AD tenant ID |
| Client ID | Yes | App registration client ID |
| Client Secret | Yes | App registration client secret |

## Ingest

Discovers managed machines (hostname, OS, health/risk/exposure level, last seen), active security alerts, vulnerability findings via Defender TVM, and installed software with vulnerability counts.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `isolate_machine` | Network isolate a machine | `unisolate_machine` |
| `unisolate_machine` | Remove network isolation | N/A |
| `restrict_app_execution` | Restrict code execution to Microsoft-signed binaries only | N/A |
| `run_antivirus_scan` | Trigger a full or quick antivirus scan | N/A |
| `initiate_investigation` | Create an automated investigation on a machine | N/A |
| `get_machine_actions` | Retrieve pending/running machine actions | N/A |
