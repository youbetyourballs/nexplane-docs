# Identity Change Types

Identity changes affect user accounts, service accounts, and group memberships across connected identity systems.

## Lock User Account

Disables a user account so the user cannot authenticate. The account is not deleted -- it can be unlocked. This is the recommended first response when a user account is suspected compromised.

**Supported connectors:** LDAP, Keycloak, AWS IAM, GCP IAM, PostgreSQL, MongoDB

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| User | string | User identifier (DN for LDAP, email for Keycloak, ARN for AWS, etc.) |
| Reason | string | Reason for locking (recorded in audit log) |

**Execution per connector:**

| Connector | Mechanism |
|---|---|
| LDAP (AD) | Sets `userAccountControl` bit to disable login |
| LDAP (OpenLDAP) | Sets `pwdAccountLockedTime` |
| Keycloak | Sets `enabled: false` |
| AWS | Attaches a deny-all inline policy |
| GCP | Sets service account `disabled: true` |
| PostgreSQL | `ALTER USER ... NOLOGIN` |
| MongoDB | Sets `disabled: true` (MongoDB 4.4+) |

**Rollback:** Unlocks the account using the inverse operation for each connector.

**Risk base score:** 5 (medium -- may break dependent services if account is a service account)

---

## Unlock User Account

Re-enables a previously locked user account.

**Supported connectors:** LDAP, Keycloak, AWS IAM, GCP IAM, PostgreSQL, MongoDB

**Rollback:** Locks the account again.

**Risk base score:** 5 (medium -- re-enables access for a previously locked account)

---

## Add User to Group

Adds a user to a group, granting any permissions inherited by group membership.

**Supported connectors:** LDAP, Keycloak

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| User | string | User DN (LDAP) or user ID (Keycloak) |
| Group | string | Group DN (LDAP) or group ID (Keycloak) |

**Rollback:** Removes the user from the group.

**Risk base score:** 4-8 depending on group sensitivity (configured per group in Nexplane)

---

## Remove User from Group

Removes a user from a group, revoking inherited permissions.

**Supported connectors:** LDAP, Keycloak

**Rollback:** Adds the user back to the group.

**Risk base score:** 4-7 depending on group sensitivity

---

## Expire User Password

Forces a user to change their password at next login. The user can still authenticate with their current password -- they are just prompted to change it.

**Supported connectors:** LDAP

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| User DN | string | Full LDAP distinguished name of the user |

**Execution:** Sets `pwdMustChange: TRUE` (OpenLDAP) or `pwdLastSet: 0` (Active Directory)

**Rollback:** Clears the must-change flag.

**Risk base score:** 2 (low -- does not block access)

---

## Expire All Active Sessions

Logs out all active sessions for a user without disabling the account. Useful for forcing re-authentication after a suspicious login event.

**Supported connectors:** Keycloak

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| User ID | string | Keycloak user ID |

**Rollback:** Not available (sessions cannot be restored once expired).

**Risk base score:** 3 (low-medium -- user is immediately logged out but can log back in)
