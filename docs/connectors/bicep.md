# Azure Bicep

The Azure Bicep connector manages ARM/Bicep template deployments.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Tenant ID | Yes | Azure AD tenant ID |
| Client ID | Yes | Service principal client ID |
| Client Secret | Yes | Service principal secret |
| Subscription ID | Yes | Azure subscription ID |

## Ingest

Discovers all ARM deployments (name, status, mode, timestamp) and deployment operations.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `validate_template` | Validate an ARM/Bicep template without deploying | N/A |
| `create_deployment` | Deploy an ARM/Bicep template to a resource group | Delete the deployment |
| `delete_deployment` | Delete an ARM deployment record | N/A |
| `cancel_deployment` | Cancel an in-progress deployment | N/A |
