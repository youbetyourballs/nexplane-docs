# Gitea

The Gitea connector manages user accounts on self-hosted Gitea instances.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Gitea Server URL | Yes | e.g. `https://gitea.example.com` |
| API Token | Yes | Gitea admin API token |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `gitea_suspend_user` | Suspend a Gitea user account (sets `prohibit_login=true`) | Unsuspend the user |
