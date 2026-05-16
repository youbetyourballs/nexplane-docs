# GitLab CE

The GitLab connector manages user accounts and personal access tokens on self-hosted GitLab instances.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| GitLab Server URL | Yes | e.g. `https://gitlab.example.com` |
| Personal Access Token (admin) | Yes | GitLab admin PAT with `api` scope |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `gitlab_suspend_user` | Block a GitLab user account via `PUT /api/v4/users/{id}/block` | Unblock the user |
| `gitlab_rotate_token` | Revoke all active personal access tokens for a user and create a new one | Revoke the newly created token |
