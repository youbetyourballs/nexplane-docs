# Microsoft Entra ID Connector

The Microsoft Entra ID (formerly Azure Active Directory) connector manages cloud identity using the Microsoft Graph API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Tenant ID | Yes | Azure AD tenant ID (GUID) |
| Client ID | Yes | App registration client ID |
| Client Secret | Yes | App registration client secret |

## Ingest

Discovers users, groups, and service principals.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| Disable user | Disable an Entra ID user account | Enable the user |
| Enable user | Enable a disabled Entra ID user | Disable the user |
| Revoke sessions | Revoke all active sign-in sessions | N/A |
| Assign license | Assign a Microsoft 365 license | Remove the license |
| Remove license | Remove a Microsoft 365 license | Reassign the license |
| Remove from Teams | Remove a user from Microsoft Teams | Add back to Teams |

Entra ID is used in the offboarding kill switch (phase 1: disable account) and access reviews (group membership collection).
