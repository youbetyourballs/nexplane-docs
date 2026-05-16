# Slack Connector

The Slack connector manages user accounts in your Slack workspace using the `slack-sdk` library.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Bot Token | Yes | Slack bot OAuth token with `admin.users:write` scope |

## Ingest

Discovers workspace members and channels.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `deactivate_user` | Deactivate a Slack user (they can no longer sign in) | Reactivate user |
| `reactivate_user` | Reactivate a deactivated Slack user | Deactivate user |

Slack is used in the offboarding kill switch (phase 2: deactivate user).

!!! note
    Slack deactivation requires the Slack admin API, which is only available on paid Slack plans (Business+ or Enterprise Grid).
