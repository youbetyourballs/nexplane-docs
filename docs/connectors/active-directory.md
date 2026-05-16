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
