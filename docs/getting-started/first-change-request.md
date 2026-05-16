# Your First Change Request

This guide walks through creating, approving, and executing a change request to rotate an IAM access key. By the end you will have seen the full Nexplane change lifecycle: draft, risk score, approval, execution, and rollback availability.

## Prerequisites

- AWS connector connected and asset discovery complete (see [Connect a Cloud Account](connect-cloud.md))
- At least one IAM user with an active access key visible in the asset list

## Step 1: Open Change Requests

Click **Change Requests** in the left sidebar, then **New Change Request**.

## Step 2: Fill in the Request

| Field | Value |
|---|---|
| Title | `Rotate prod-deploy IAM key` |
| Connector | `prod-aws` (your AWS connector) |
| Change Type | `Credentials - Rotate IAM Access Key` |
| Target | Select the IAM user from the dropdown |
| Description | `Rotating access key as part of quarterly credential rotation` |

Click **Next: Review Risk**.

## Step 3: Review the Risk Score

Nexplane scores the change before you can submit it. For an IAM key rotation, you will typically see:

- **Risk Level:** Medium
- **Impact:** Application workloads using this key will fail if not updated with the new key
- **Rollback available:** Yes -- old key can be re-activated within the retention window

The risk score is calculated from the change type, the target's blast radius (how many systems use this identity), and the connector environment label (prod vs staging).

Review the risk summary and click **Submit for Approval**.

## Step 4: Approve the Change

If you are the only admin, you can self-approve. Click **Approve** on the change request detail page.

In a team setup, the approver receives an email notification and approves from their own session.

!!! note "Approval policies"
    You can configure approval policies in Settings > Approval Policies to require two approvers for high-risk changes, or to bypass approval for low-risk changes in non-production environments.

## Step 5: Execute

Once approved, the change moves to **Ready to Execute**. Click **Execute Now**.

Nexplane will:

1. Create a new IAM access key for the user
2. Store the new key value in the change record (encrypted)
3. Deactivate the old key
4. Record the old key ID for rollback

Execution typically completes in 3-5 seconds. The status will update to **Completed**.

## Step 6: View the Result

Click **View Execution Details** to see:

- The new access key ID (the secret is shown once, then encrypted and stored)
- The old key ID that was deactivated
- Timestamps for each step
- A link to the rollback action

## Step 7: Verify Rollback is Available

On the change detail page, click **Rollback**. You will see a preview of the rollback action:

- Re-activate the old key
- Deactivate the new key

You can execute the rollback immediately if needed. Rollback is a first-class operation in Nexplane -- it goes through the same approval flow as the original change.

## What Just Happened

You have seen the full Nexplane change lifecycle:

1. **Draft** -- structured change with a target and change type
2. **Risk Score** -- automatic assessment before submission
3. **Approval** -- gated human sign-off
4. **Execution** -- connector-backed, audited operation
5. **Verification** -- execution details logged and stored
6. **Rollback** -- typed inverse operation, ready on demand

Every change type in Nexplane follows this same flow.

## Next Step

[Deploy the agent to a host](deploy-agent.md)
