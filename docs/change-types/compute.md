# Compute Change Types

Compute changes affect the operational state of virtual machines, containers, and cloud compute resources.

## Snapshot EC2 Instance

Creates an EBS snapshot of all volumes attached to an EC2 instance before a risky change. The snapshot is stored in AWS and can be used to restore the instance state if subsequent changes cause problems.

**Connector:** AWS

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Instance ID | string | EC2 instance ID (e.g., `i-0abcd1234efgh5678`) |
| Description | string | Snapshot description |

**Execution:**
1. Calls `ec2:CreateSnapshot` for each attached volume
2. Waits for all snapshots to reach `completed` state (up to 10 minutes)
3. Records snapshot IDs in the change record

**Rollback:** Deletes the created snapshots.

**Risk base score:** 2 (low -- snapshot creation does not affect instance operation)

---

## Stop EC2 Instance

Stops a running EC2 instance. The instance is stopped (not terminated) -- data on EBS volumes is preserved.

**Connector:** AWS

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Instance ID | string | EC2 instance ID |
| Force | boolean | Force stop (equivalent to pulling the power, may cause data loss) |

**Rollback:** Starts the stopped instance.

**Risk base score:** 7 (high -- stops production workloads)

---

## Start EC2 Instance

Starts a stopped EC2 instance.

**Connector:** AWS

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Instance ID | string | EC2 instance ID |

**Rollback:** Stops the started instance.

**Risk base score:** 2 (low)

---

## Stop / Start GCE Instance

Equivalent operations for Google Compute Engine instances via the GCP connector.

**Risk base score:** Stop: 7, Start: 2

---

## Stop / Deallocate Azure VM

Stop and start operations for Azure Virtual Machines. Azure VMs can be stopped (OS shutdown, still billed) or deallocated (OS shutdown, billing stops, public IP released).

**Risk base score:** Stop: 7, Start: 2

---

## Cordon Kubernetes Node

Marks a Kubernetes node as unschedulable, preventing new pods from being placed on it. Existing pods are not evicted.

**Connector:** Kubernetes

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Node Name | string | Kubernetes node name |

**Rollback:** Uncordons the node.

**Risk base score:** 4 (medium -- degrades cluster capacity)

---

## Patch Kubernetes Deployment

Updates a deployment's container image or replica count.

**Connector:** Kubernetes

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Namespace | string | Kubernetes namespace |
| Deployment Name | string | Deployment name |
| Container Name | string | Container to update |
| Image | string | New container image and tag |

**Rollback:** Restores the previous image tag from the deployment spec snapshot taken before the change.

**Risk base score:** 6 (medium -- rolling update, but incorrect image can break the workload)
