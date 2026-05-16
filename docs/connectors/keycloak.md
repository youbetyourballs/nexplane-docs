# Keycloak Connector

The Keycloak connector uses the Keycloak Admin REST API to manage users, clients, and realms. It supports password resets, account suspension, and client secret rotation across one or more Keycloak realms.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-keycloak`) |
| Server URL | string | Yes | Keycloak base URL (e.g., `https://auth.example.com`) |
| Admin Username | string | Yes | Keycloak admin account username |
| Admin Password | string | Yes | Keycloak admin account password |
| Realm | string | Yes | Target realm name (e.g., `master` or `myrealm`) |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Reset User Password | Sets a temporary or permanent password on a user | No rollback (old password is not stored) |
| Disable User | Sets `enabled: false` on a user | Re-enable the user |
| Enable User | Sets `enabled: true` on a user | Disable the user |
| Expire User Session | Logs out all active sessions for a user | No rollback |
| Rotate Client Secret | Regenerates the client secret for a confidential client | No rollback (old secret is not stored by Keycloak) |
| Add User to Group | Adds a user to a Keycloak group | Remove from group |
| Remove User from Group | Removes a user from a Keycloak group | Add back to group |

## Minimum Permissions Required

The admin account used by Nexplane needs the `manage-users` and `manage-clients` realm roles in the target realm. In Keycloak:

1. Go to **Realm Settings > Users > Admin User**
2. Click **Role Mappings > Client Roles**
3. Select `realm-management`
4. Assign `manage-users`, `manage-clients`, `view-users`, `view-clients`

Avoid using the global `admin` account in production -- create a dedicated service account with scoped realm roles.

## Known Limitations

- The connector targets a single realm per connector instance. To manage multiple realms, create one connector per realm.
- Keycloak does not return the previous client secret when rotating. Rollback is not possible for client secret rotation -- inform dependent applications of the new secret before rotating.
- Session expiry logs out all active sessions but does not prevent the user from immediately logging back in. It is most useful when combined with disabling the user account.
- Federated users (LDAP-backed or social login) may not support all operations. Password resets, for example, are not available for LDAP-federated users through the Keycloak API -- those must go through the LDAP connector instead.
