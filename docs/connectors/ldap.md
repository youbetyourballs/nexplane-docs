# LDAP Connector

The LDAP connector uses the `python-ldap` library to communicate with any RFC 4511-compliant LDAP directory, including Microsoft Active Directory, OpenLDAP, and FreeIPA. It supports user account management, group membership changes, and password resets.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `corp-ad`) |
| Server URL | string | Yes | LDAP server URL (e.g., `ldaps://dc01.corp.example.com:636`) |
| Bind DN | string | Yes | Distinguished name of the bind account (e.g., `CN=nexplane,CN=Users,DC=corp,DC=example,DC=com`) |
| Bind Password | string | Yes | Password for the bind account |
| Base DN | string | Yes | Base DN for searches (e.g., `DC=corp,DC=example,DC=com`) |
| Use TLS | boolean | No | Use LDAPS or STARTTLS (default: true) |
| CA Certificate | string | No | PEM-encoded CA certificate for TLS validation |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Reset User Password | Sets a new password on a user account | No rollback (old password is not stored) |
| Lock User Account | Sets `userAccountControl` to disable login (AD) or `pwdAccountLockedTime` (OpenLDAP) | Unlock the account |
| Unlock User Account | Clears the locked status | Lock the account |
| Add User to Group | Adds a user DN to a group's `member` attribute | Remove from group |
| Remove User from Group | Removes a user DN from a group | Add back to group |
| Expire User Password | Forces password change on next login | Clear the expiry flag |

## Minimum Permissions Required

The bind account must have:

- `Read` on all user and group objects in the base DN (for discovery)
- `Write` on `userPassword` or `unicodePwd` attribute for password resets
- `Write` on `userAccountControl` for account lock/unlock (Active Directory)
- `Write` on `member` attribute on group objects for group membership changes

In Active Directory, the built-in `Account Operators` group grants most of these permissions. For least-privilege setups, use a custom role with delegated control scoped to the relevant OU.

## Known Limitations

- Active Directory requires password resets to be performed over an LDAPS or STARTTLS-secured connection. Password resets over plain LDAP will be rejected by AD with error code 53.
- The connector does not support Kerberos authentication. Use LDAP simple bind with a service account.
- `unicodePwd` changes for Active Directory require the password to be wrapped in double quotes and encoded as UTF-16LE. The connector handles this automatically.
- Group membership changes using the `member` attribute may not be reflected in `memberOf` on the user object immediately due to AD replication lag.
- Discovery is limited to objects within the configured Base DN. Objects in other partitions (e.g., the Schema partition) are not discovered.
