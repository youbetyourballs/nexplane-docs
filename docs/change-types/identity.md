# Identity & IAM Change Types

Identity changes affect user accounts, service accounts, and group memberships across connected identity systems.

## User Lifecycle

| Change Type | Description | Connectors |
|-------------|-------------|-----------|
| `offboard_user` | Atomically disables a user across AD, Okta, Entra ID, Google Workspace, GitHub, Slack; optionally isolates CrowdStrike-managed endpoints | AD, Okta, Entra ID, Google Workspace, GitHub, Slack, CrowdStrike |
| `onboard_user` | Provisions user accounts across all connected identity systems from a single form | AD, Okta, Entra ID, Google Workspace, GitHub, Slack |

See [Identity Lifecycle](../features/identity-lifecycle.md) for the full offboarding phase breakdown.

## IAM Users (AWS)

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `iam_user_create` | Create an IAM user with optional group membership and policy attachment | Delete the created user |
| `iam_user_delete` | Delete an IAM user and all associated keys and policies | Not available |

**Connector:** AWS

## SaaS Identity Actions

### Google Workspace

| Change Type | Description |
|-------------|-------------|
| `remove_from_groups` | Remove user from Google Groups |
| `reset_2fa` | Reset two-factor authentication enrollment |
| `revoke_oauth_tokens` | Revoke all OAuth app tokens for a user |
| `wipe_mobile_device` | Remote wipe a mobile device associated with the account |
| `suspend_user` | Suspend a Google Workspace user |
| `unsuspend_user` | Unsuspend a Google Workspace user |

### GitHub

| Change Type | Description |
|-------------|-------------|
| `remove_org_member` | Remove a user from the GitHub organization |
| `revoke_user_pats` | Revoke all personal access tokens for a user |
| `enforce_branch_protection` | Apply branch protection rules to a repository |
| `archive_repo` | Archive a GitHub repository |
| `disable_actions` | Disable GitHub Actions on a repository |
| `enable_actions` | Enable GitHub Actions on a repository |

### Slack

| Change Type | Description |
|-------------|-------------|
| `deactivate_user` | Deactivate a Slack user |
| `reactivate_user` | Reactivate a Slack user |

### Microsoft Entra ID

| Change Type | Description |
|-------------|-------------|
| `remove_from_teams` | Remove user from Microsoft Teams |
| `assign_license` | Assign a Microsoft 365 license |
| `remove_license` | Remove a Microsoft 365 license |
| `revoke_sessions` | Revoke all active sessions |
| `disable_user` | Disable an Entra ID user account |

### Kubernetes

| Change Type | Description |
|-------------|-------------|
| `restart_deployment` | Restart all pods in a deployment |
| `scale_deployment` | Scale a deployment up or down |
| `apply_network_policy` | Apply a Kubernetes NetworkPolicy |
| `update_rbac` | Update a RBAC role binding |
| `rotate_secret` | Rotate a Kubernetes secret value |
| `helm_upgrade` | Upgrade a Helm release |
| `helm_rollback` | Roll back a Helm release to a previous revision |

## Access Reviews

Access reviews auto-generate identity change requests for revoked access. See [Identity Lifecycle](../features/identity-lifecycle.md#access-reviews).
