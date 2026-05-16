# CrowdStrike Connector

The CrowdStrike connector uses the `falconpy` SDK to manage endpoints and respond to threats.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Client ID | Yes | OAuth2 client ID |
| Client Secret | Yes | OAuth2 client secret |
| Base URL | Yes | API base URL (e.g., `https://api.crowdstrike.com`) |

## Ingest

Discovers hosts and their containment status. Imports alerts and detections as vulnerability findings.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| Isolate host | Place a host in network containment | Lift containment |
| Lift containment | Remove host from network containment | Re-contain the host |
| RTR command | Execute a Real Time Response command on a host | N/A |
| Update prevention policy | Modify a prevention policy setting | Restore previous policy |

CrowdStrike host containment is used in phase 5 of the offboarding kill switch (optionally isolate endpoints associated with the departing user).
