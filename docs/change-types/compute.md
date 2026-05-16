# Compute Change Types

Compute changes affect the operational state of virtual machines, cloud instances, and related infrastructure.

## EC2

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `ec2_launch` | Launch a new EC2 instance from an AMI | Terminate the launched instance |
| `ec2_start` | Start a stopped EC2 instance | Stop the instance |
| `ec2_stop` | Stop a running EC2 instance | Start the instance |
| `ec2_reboot` | Reboot an EC2 instance | N/A |
| `ec2_terminate` | Terminate an EC2 instance | Not available — termination is irreversible |
| `key_pair_create` | Create an EC2 key pair | Delete the key pair |
| `key_pair_delete` | Delete an EC2 key pair | Not available |
| `snapshot_asset` | Create an EBS snapshot of all attached volumes | Delete the created snapshots |

**Connector:** AWS

**Risk base scores:** Launch/Start = 3, Stop = 7, Reboot = 6, Terminate = 9, Snapshot = 2

---

## SSM Command

**Change type:** `ssm_command`

Runs an approved SSM document against an EC2 instance. Only SSM documents explicitly allow-listed in the connector configuration can be executed — freeform shell commands are blocked unconditionally by the safety engine.

**Connector:** AWS

**Parameters:** document name, target instance ID, document parameters

---

## Azure VM

| Change Type | Description |
|-------------|-------------|
| `azure_vm_create` | Create a new Azure VM |
| `azure_vm_stop` | Stop (deallocate) an Azure VM |
| `azure_vm_start` | Start a deallocated Azure VM |
| `azure_vm_reboot` | Reboot an Azure VM |
| `azure_vm_snapshot` | Create a managed disk snapshot |
| `azure_vm_delete` | Delete an Azure VM |

**Connector:** Azure

---

## GCP Compute

| Change Type | Description |
|-------------|-------------|
| `gcp_instance_create` | Create a new GCP Compute instance |
| `gcp_instance_stop` | Stop a running GCP instance |
| `gcp_instance_start` | Start a stopped GCP instance |
| `gcp_instance_reboot` | Reboot a GCP instance |
| `gcp_instance_snapshot` | Create a persistent disk snapshot |
| `gcp_instance_delete` | Delete a GCP instance |

**Connector:** GCP

---

## IP Migration

| Change Type | Description |
|-------------|-------------|
| `change_ip` | Change the IP address of a managed host via the Nexplane Agent |
| `migrate_ip` | DNS-coordinated IP migration (lowers TTL first for long-TTL records) |
| `ip_campaign` | Orchestrated IP changes across a fleet with batch control and abort threshold |

See [IP Migration](../features/ip-migration.md) for full details on methods, the dead man's switch, and the IP Migration Wizard.

---

## Agent Deploy

**Change type:** `deploy_nexplane_agent`

Installs the Nexplane Agent on a target EC2 instance via SSM. Downloads the binary from S3, sets up the systemd service, and registers the host with the control plane.

**Connector:** AWS (via SSM)
