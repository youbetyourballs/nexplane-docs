# Okta Connector

The Okta connector manages users, groups, and applications in your Okta tenant.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Org URL | Yes | Your Okta organization URL (e.g., `https://acme.okta.com`) |
| API Token | Yes | Okta API token with admin privileges |

## Ingest

Discovers users, groups, and applications and syncs them as Nexplane assets.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| Disable user | Deactivate an Okta user | Re-activate the user |
| Enable user | Activate a deactivated Okta user | Deactivate the user |
| Add to group | Add a user to an Okta group | Remove from group |
| Remove from group | Remove a user from an Okta group | Add back to group |
| Revoke sessions | Revoke all active sessions for a user | N/A |
| Force MFA enrollment | Require a user to enroll in MFA | N/A |
| Rotate API token | Generate a new Okta API token | Re-activate old token |

Okta is used in the offboarding kill switch (phase 1: disable user) and access reviews (group membership collection).
