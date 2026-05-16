# PagerDuty Connector

The PagerDuty connector creates incidents and routes alerts.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| API Key | Yes | PagerDuty API key |
| Service ID | Yes | PagerDuty service ID to create incidents under |

## Capabilities

When a high or critical risk change request is executed, or when an incident response playbook is triggered, Nexplane can create a PagerDuty incident automatically. Alert routing follows the service's configured escalation policy.
