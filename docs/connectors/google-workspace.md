# Google Workspace Connector

The Google Workspace connector uses the `google-api-python-client` library to manage users, groups, and devices.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Service Account JSON | Yes | Service account key with domain-wide delegation |
| Admin Email | Yes | Email of a Workspace admin to impersonate |

## Ingest

Discovers users, groups, and organizational units.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `remove_from_groups` | Remove a user from all Google Groups | Re-add to groups |
| `reset_2fa` | Reset two-factor authentication enrollment | N/A |
| `revoke_oauth_tokens` | Revoke all OAuth application tokens for a user | N/A |
| `wipe_mobile_device` | Remote wipe a mobile device associated with the account | N/A |
| `suspend_user` | Suspend a Google Workspace user (account disabled) | Unsuspend user |
| `unsuspend_user` | Unsuspend a Google Workspace user | Suspend user |

Google Workspace is used in the offboarding kill switch (phase 1: suspend user, remove from groups) and access reviews (group membership collection).
