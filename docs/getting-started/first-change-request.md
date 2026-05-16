# Your First Change Request

This guide walks through creating, approving, and executing a change request. By the end you will have seen the full Nexplane change lifecycle.

## Prerequisites

- AWS connector connected and asset discovery complete (see [Connect a Cloud Account](connect-cloud.md))

## Step 1: Open Change Requests

Click **Change Requests** in the left sidebar, then **New Change Request**.

## Step 2: Fill in the Request

Select a change type. For this example we'll use `security_group_update`:

| Field | Value |
|-------|-------|
| Title | `Restrict SSH to office CIDR` |
| Connector | `prod-aws` |
| Change Type | `security_group_update` |
| Target | Select the security group from the asset dropdown |
| Description | `Tighten SSH ingress after vulnerability scan` |

Fill in the change-type-specific parameters (the rule to add or remove), then click **Submit**.

## Step 3: Safety Review

Nexplane scores the change before it proceeds. You'll see:

- **Risk Level** — based on change type, asset environment (prod/staging), and blast radius
- **Rollback available** — yes/no and what rollback does
- **Blocking conditions** — if any (e.g., missing rollback strategy on a critical asset)

Review and click **Submit for Approval**.

## Step 4: Approve the Change

The change moves to **Awaiting Approval**. Click **Approve**.

In a team setup with approval policies configured, the approver receives a notification and approves from their own session. High-risk changes require two approvers.

## Step 5: Execute

Once approved, click **Execute**. Nexplane calls the AWS API, applies the security group rule, and updates the asset inventory.

## Step 6: View the Result

The change request moves through:

```
Draft → Planned → Awaiting Approval → Approved → Executing → Verifying → Completed
```

The execution detail view shows each step, timestamps, API responses, and a link to the audit log entry.

## Step 7: Rollback (Optional)

On the change detail page, click **Rollback** to reverse the change. Rollback goes through the same approval flow as the original change. Every change type in Nexplane knows its own inverse operation.

## Next Step

[Deploy the agent to a host](deploy-agent.md)
