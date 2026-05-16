# GitHub Connector

The GitHub connector uses the `PyGithub` library to manage organization membership, repository settings, and user access.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Token | Yes | Personal access token or GitHub App token with `admin:org` and `repo` scopes |
| Organization | Yes | GitHub organization name |

## Ingest

Discovers organization members, repositories, and teams.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `remove_org_member` | Remove a user from the GitHub organization | Re-invite the user |
| `revoke_user_pats` | Revoke all personal access tokens for a user | N/A |
| `enforce_branch_protection` | Apply branch protection rules (require PR, require reviews, require status checks) | Remove the protection rules |
| `archive_repo` | Archive a GitHub repository (read-only) | Unarchive the repository |
| `disable_actions` | Disable GitHub Actions on a repository | Enable Actions |
| `enable_actions` | Enable GitHub Actions on a repository | Disable Actions |

GitHub is used in the offboarding kill switch (phase 2: remove org member, revoke PATs) and access reviews (org membership collection).
