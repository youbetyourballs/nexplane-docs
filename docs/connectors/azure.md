# Azure Connector

The Azure connector uses the Azure SDK for Python to interact with Azure Active Directory, virtual machines, and storage. It supports service principal credential rotation, VM management, and network security group operations.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-azure`) |
| Tenant ID | string | Yes | Azure AD tenant ID (GUID) |
| Client ID | string | Yes | Application (client) ID of the service principal |
| Client Secret | string | Yes | Client secret for the service principal |
| Subscription ID | string | Yes | Azure subscription ID (GUID) |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate Service Principal Secret | Creates a new client secret, records the old secret ID | Delete new secret, note old secret cannot be restored |
| Disable Service Principal | Sets the service principal `accountEnabled` flag to false | Re-enable the service principal |
| Enable Service Principal | Sets `accountEnabled` to true | Disable the service principal |
| Modify NSG Rule | Adds, modifies, or removes a network security group rule | Restore the previous rule |
| Stop VM | Deallocates a running virtual machine | Start the VM |
| Start VM | Starts a deallocated virtual machine | Deallocate the VM |

## Minimum Permissions Required

The service principal used by Nexplane needs:

For discovery:
- Azure AD: `Directory.Read.All` (Microsoft Graph)
- Subscription: `Reader` role

For full change execution:
- Azure AD: `Application.ReadWrite.All` (Microsoft Graph)
- Subscription: `Contributor` or scoped `Network Contributor` + `Virtual Machine Contributor` + `Storage Account Contributor`

## Known Limitations

- Azure AD client secrets cannot be retrieved after creation. When rotating a secret, Nexplane stores the new secret value (encrypted) in the change record for handoff, but the old secret value is not stored -- rollback disables the new secret but cannot restore the old one.
- NSG rule priority values must be unique within a rule set. If the restored rule priority conflicts with a rule added after the change, rollback will fail with a conflict error.
- VM start/stop operations are asynchronous. Nexplane polls for up to 10 minutes.
- The connector targets a single subscription. Multi-subscription setups require one connector per subscription.
