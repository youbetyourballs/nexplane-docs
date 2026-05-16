# Connect a Cloud Account

This guide walks through connecting an AWS account to Nexplane. Azure, GCP, and other connectors follow the same pattern — only the credential fields differ.

## What You'll Need

- An AWS IAM access key and secret access key
- For discovery only: `iam:List*`, `iam:Get*`, `ec2:Describe*`, `s3:List*`, `sts:GetCallerIdentity`
- For full change execution: see [AWS Connector](../connectors/aws.md#required-permissions)

## Step 1: Open Connectors

In the Nexplane UI, click **Connectors** in the left sidebar, then **Add Connector**.

Select **AWS** from the connector type dropdown.

## Step 2: Enter Credentials

| Field | Example | Description |
|-------|---------|-------------|
| Name | `prod-aws` | Label shown in the UI |
| AWS Access Key ID | `AKIAIOSFODNN7EXAMPLE` | IAM access key |
| AWS Secret Access Key | `wJalrXUtnFEMI/K7MDENG/...` | IAM secret key |
| Default Region | `us-east-1` | Region for regional API calls |
| Account ID | `123456789012` | Optional — for display |
| Role ARN | `arn:aws:iam::...` | Optional — for cross-account access |
| External ID | `nexplane-12345` | Optional — for cross-account assume-role |

Click **Save Connector**.

Nexplane encrypts the secret key before storing it using `SecretsService` (Fernet AES-256). The plaintext secret is never written to disk or returned in subsequent API responses.

## Step 3: Test the Connection

After saving, click **Test Connection** on the connector row. Nexplane makes a `sts:GetCallerIdentity` call to verify the credentials are valid.

## Step 4: Run Asset Discovery

Click the connector to open it, then click **Trigger Ingest**. Nexplane discovers:

- EC2 instances across all regions
- Key pairs
- IAM users and roles
- S3 buckets
- Security groups
- And more, depending on configured permissions

Discovery runs in the background. Assets appear in the **Asset Inventory** once complete.

!!! info "Scheduled discovery"
    After the initial discovery, Nexplane re-discovers assets on the schedule configured on the connector (default: daily). You can trigger a manual re-discovery any time.

## What's Stored

Nexplane stores asset metadata (IDs, names, regions, tags) — not the asset contents themselves. No S3 object data, no EC2 user data, no secrets are pulled during discovery.

## Next Step

[Create your first change request](first-change-request.md)
