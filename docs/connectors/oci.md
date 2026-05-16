# OCI Connector

The OCI connector uses the Oracle Cloud Infrastructure Python SDK to manage compute, networking, storage, IAM, databases, and monitoring across OCI tenancies.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name (e.g., `prod-oci`) |
| Tenancy OCID | Yes | `ocid1.tenancy.oc1..` |
| User OCID | Yes | `ocid1.user.oc1..` |
| API Key Fingerprint | Yes | `xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx` |
| Home Region | Yes | e.g., `us-ashburn-1` |
| PEM Private Key | Yes | RSA private key corresponding to the registered API key fingerprint |

## Ingest

Discovers: compartments, compute instances, VCNs and subnets, object storage buckets, block volumes, security lists, NSGs, load balancers, DNS zones, IAM users/groups/policies, Vault secrets, Autonomous Databases, MySQL HeatWave DB Systems, monitoring alarms.

## Capabilities

### Compute

| Action | Description | Rollback |
|--------|-------------|---------|
| `oci_instance_create` | Launch a compute instance (defaults to VM.Standard.E2.1.Micro, always-free eligible) | Terminate the instance |
| `oci_instance_stop` | Stop a running instance | Start the instance |
| `oci_instance_start` | Start a stopped instance | Stop the instance |
| `oci_instance_reboot` | Graceful reboot (SOFTRESET) | N/A |
| `oci_instance_delete` | Terminate an instance | Not available |
| `oci_block_volume_snapshot` | Create a boot volume backup (incremental or full) | Delete the backup |

### Networking

| Action | Description | Rollback |
|--------|-------------|---------|
| `oci_vcn_create` | Create a VCN with internet gateway and default route | Delete the VCN |
| `oci_vcn_delete` | Delete a VCN and its internet gateway | N/A |
| `oci_subnet_create` | Create a subnet with SSH + ICMP security list | Delete the subnet |
| `oci_security_list_add_rule` | Add an inbound or outbound rule to a Security List | Remove the rule |
| `oci_security_list_remove_rule` | Remove a rule from a Security List | Add the rule back |
| `oci_nsg_create` | Create a Network Security Group | Delete the NSG |
| `oci_nsg_rule_add` | Add a rule to an NSG | Remove the rule |
| `oci_nsg_rule_remove` | Remove a rule from an NSG | Add the rule back |
| `oci_load_balancer_create` | Create a flexible Load Balancer | Delete the load balancer |
| `oci_load_balancer_delete` | Delete a Load Balancer | N/A |
| `oci_backend_set_create` | Create a backend set on a Load Balancer | Delete the backend set |
| `oci_listener_create` | Create a listener on a Load Balancer | Delete the listener |

### Object Storage

| Action | Description | Rollback |
|--------|-------------|---------|
| `oci_bucket_create` | Create an object storage bucket | Delete the bucket |
| `oci_bucket_delete` | Delete a bucket (must be empty) | N/A |
| `oci_bucket_lifecycle_set` | Configure lifecycle rules on a bucket | Restore previous rules |
| `oci_bucket_block_public` | Set public access type to NoPublicAccess | Restore previous setting |

### Block Volumes

| Action | Description | Rollback |
|--------|-------------|---------|
| `oci_block_volume_create` | Create a block volume | Delete the volume |
| `oci_block_volume_attach` | Attach a block volume to an instance | Detach the volume |
| `oci_block_volume_detach` | Detach a block volume from an instance | Attach the volume |
| `oci_block_volume_delete` | Delete a block volume (must be detached) | N/A |
| `oci_block_volume_backup` | Create an incremental block volume backup | N/A |

### DNS

| Action | Description | Rollback |
|--------|-------------|---------|
| `oci_dns_zone_create` | Create a DNS zone (PRIMARY type) | Delete the zone |
| `oci_dns_zone_delete` | Delete a DNS zone | N/A |
| `oci_dns_record_upsert` | Create or update a DNS record | Delete the record |

### IAM

| Action | Description | Rollback |
|--------|-------------|---------|
| `oci_iam_user_create` | Create an IAM user (tenancy-scoped) | Delete the user |
| `oci_iam_user_delete` | Remove group memberships and delete a user | N/A |
| `oci_iam_user_disable` | Block console login and API key access | Enable the user |
| `oci_iam_user_enable` | Restore console login and API key access | Disable the user |
| `oci_iam_group_create` | Create an IAM group, optionally add users | Delete the group |
| `oci_iam_group_delete` | Remove all memberships and delete a group | N/A |
| `oci_iam_policy_create` | Create an IAM policy with policy statements | Delete the policy |
| `oci_iam_policy_delete` | Delete an IAM policy | N/A |
| `oci_compartment_create` | Create a child compartment | Delete the compartment |
| `oci_compartment_delete` | Delete a compartment (must be empty) | N/A |

### Vault

| Action | Description | Rollback |
|--------|-------------|---------|
| `oci_vault_secret_create` | Create a secret in OCI Vault | Schedule secret for deletion |
| `oci_vault_secret_delete` | Schedule a secret for deletion (minimum 1-day deferred) | N/A |

### Databases

| Action | Description | Rollback |
|--------|-------------|---------|
| `oci_adb_create` | Provision an Autonomous Database instance | Delete the ADB |
| `oci_adb_stop` | Stop a running Autonomous Database | Start the ADB |
| `oci_adb_start` | Start a stopped Autonomous Database | Stop the ADB |
| `oci_adb_delete` | Delete an Autonomous Database (destructive) | Not available |
| `oci_adb_backup` | Create a manual ADB backup | N/A |
| `oci_mysql_create` | Provision a MySQL HeatWave DB System (~20 min) | Delete the DB system |
| `oci_mysql_stop` | Stop a MySQL HeatWave DB System | Start the DB system |
| `oci_mysql_start` | Start a stopped MySQL HeatWave DB System | Stop the DB system |
| `oci_mysql_delete` | Delete a MySQL HeatWave DB System (destructive) | Not available |

### Monitoring

| Action | Description | Rollback |
|--------|-------------|---------|
| `oci_alarm_create` | Create a Monitoring alarm using MQL query syntax | Delete the alarm |
| `oci_alarm_delete` | Delete a Monitoring alarm | N/A |
| `oci_logging_enable` | Create a Log Group and Log for compartment-level logging | Disable logging |
| `oci_logging_disable` | Delete a Log Group and contained logs | N/A |

## Smoke Test

Live end-to-end smoke tests run phases OCI_A through OCI_N:

```bash
docker exec nexplane-backend-1 python tests/smoke/test_oci_live.py \
  --base-url http://localhost:8000 \
  --email admin@acme.example \
  --password admin123
```

Phases: compartment/VCN (A), compute launch (B), instance lifecycle (C), snapshot (D), object storage (F), block volumes (G), security/NSG (H), DNS (J), IAM (K), Vault (L), monitoring/logging (N).

OCI is also included in the multi-cloud parallel test via `test_multicloud_live.py`.
