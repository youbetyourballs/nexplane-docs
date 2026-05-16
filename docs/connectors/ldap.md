# LDAP / Active Directory

The LDAP connector supports OpenLDAP-compatible directories.

!!! note
    For Microsoft Active Directory, use the dedicated [Active Directory connector](active-directory.md), which has full AD-specific support (user lifecycle, OU management, service account rotation).

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| (Connection fields configured per-deployment) | | |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `ldap_disable_user` | Disable an LDAP user account by setting `pwdAccountLockedTime` | Re-enable the account |
