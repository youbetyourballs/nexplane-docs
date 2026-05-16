# Azure Connector

The Azure connector uses the Azure SDK for Python to interact with Azure Virtual Machines, networking, storage, identity, and monitoring.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name (e.g., `prod-azure`) |
| Tenant ID | Yes | Azure AD tenant ID (GUID) |
| Client ID | Yes | Service principal application ID |
| Client Secret | Yes | Service principal secret |
| Subscription ID | Yes | Azure subscription ID |

## Required Roles

Assign these roles to the service principal at the **subscription scope**:

- `Contributor` — required for all resource create/delete/modify operations
- `User Access Administrator` — required for RBAC role assignment operations

!!! warning
    `Contributor` alone is **not sufficient**. Role assignment operations fail without `User Access Administrator`.

## Capabilities

### Virtual Machines

| Action | Description | Rollback |
|--------|-------------|---------|
| `azure_vm_create` | Create a VM | Delete the VM |
| `azure_vm_stop` | Stop (deallocate) a VM | Start the VM |
| `azure_vm_start` | Start a VM | Stop the VM |
| `azure_vm_reboot` | Reboot a VM | N/A |
| `azure_vm_snapshot` | Create a managed disk snapshot | Delete the snapshot |
| `azure_vm_delete` | Delete a VM | Not available |

### Network Security Groups

| Action | Description | Rollback |
|--------|-------------|---------|
| `azure_nsg_update` | Add or update an NSG rule | Restore previous rule |
| `azure_nsg_restore` | Restore an NSG to a snapshot state | N/A |

### Storage

| Action | Description | Rollback |
|--------|-------------|---------|
| Create storage account | Create a blob storage account | Delete the account |
| Delete storage account | Delete a storage account | Not available |
| Create container | Create a blob container | Delete the container |
| Delete container | Delete a blob container | Not available |

### Identity

| Action | Description | Rollback |
|--------|-------------|---------|
| Create managed identity | Create a user-assigned managed identity | Delete the identity |
| Delete managed identity | Delete a managed identity | Not available |
| Create RBAC assignment | Assign a role to a principal | Delete the assignment |
| Delete RBAC assignment | Remove a role assignment | Not available |

### Network

| Action | Description | Rollback |
|--------|-------------|---------|
| Create VNet | Create a virtual network and subnet | Delete the VNet |
| Create subnet | Add a subnet to an existing VNet | Delete the subnet |

### DNS

| Action | Description | Rollback |
|--------|-------------|---------|
| Create DNS zone | Create an Azure DNS zone | Delete the zone |
| Create A record | Create a DNS A record | Delete the record |

### SQL

| Action | Description | Rollback |
|--------|-------------|---------|
| Create SQL Server | Create an Azure SQL Server | Delete the server |
| Create SQL Database | Create a database on a SQL Server | Delete the database |

### Monitor

| Action | Description | Rollback |
|--------|-------------|---------|
| Create metric alert | Create a Monitor metric alert rule | Delete the alert |
| Delete metric alert | Delete a Monitor metric alert rule | Recreate the alert |

### Entra ID Users

| Action | Description |
|--------|-------------|
| Disable user | Disable an Entra ID user account |
| Enable user | Enable an Entra ID user account |
| Revoke sessions | Revoke all active sessions for a user |
| Assign license | Assign a Microsoft 365 license |
| Remove license | Remove a Microsoft 365 license |

### IaC

| Action | Description |
|--------|-------------|
| Terraform local | Run `terraform apply` in backend container |
| Ansible local | Run Ansible playbook via SSM transport |

### Resource Tagging

Apply or update tags on any Azure resource.
