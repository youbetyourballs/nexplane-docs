# Active Directory Connector

The Active Directory connector manages user accounts and service accounts in Microsoft Active Directory.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Server | Yes | AD domain controller hostname or IP |
| Domain | Yes | AD domain (e.g., `acme.local`) |
| Username | Yes | Service account with admin privileges |
| Password | Yes | Service account password |
| Base DN | Yes | LDAP base distinguished name (e.g., `DC=acme,DC=local`) |

## Ingest

Discovers users, groups, OUs, and computers. Discovers stale accounts (last logon > configurable threshold).

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| Disable account | Disable an AD user account | Re-enable the account |
| Enable account | Enable a disabled AD user account | Disable the account |
| Move OU | Move a user or computer object to a different OU | Move back to original OU |
| Rotate service account password | Generate and set a new password for a service account | Restore previous password (requires pre-capture) |
| Add to group | Add a user to an AD security group | Remove from group |
| Remove from group | Remove a user from an AD security group | Add back to group |

Active Directory is used in the offboarding kill switch (phase 1: disable account) and access reviews (group membership collection).

## Tier-Zero Operations

Tier-zero operations affect domain-wide security posture and require domain admin credentials. All are subject to approval tier 2 (high-risk) by default.

| Change Type | Description |
|-------------|-------------|
| `ad_domain_functional_level_upgrade` | Raise the forest or domain functional level. **Irreversible** — cannot be downgraded once applied. |
| `ad_trust_create` | Create a forest or domain trust with Kerberos validation. |
| `ad_gpo_deploy` | Deploy a Group Policy Object with optional pilot OU scoping before domain-wide rollout. |
| `ad_pso_manage` | Create, update, or delete a Fine-Grained Password Settings Object. |
| `ad_stale_computer_cleanup` | Discover and disable computer accounts with no recent logon. Supports `dry_run` mode. |
| `ad_dc_parallel_upgrade` | Upgrade a domain controller using the Microsoft-prescribed sequence: promote new DC, replicate, transfer FSMO roles, demote old DC. |

See [Active Directory Operations](../change-types/active-directory.md) for full parameter reference.
