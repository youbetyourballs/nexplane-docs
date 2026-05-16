# Connect a Cloud Account

This guide walks through connecting an AWS account to Nexplane. GCP, Azure, and OCI follow the same pattern -- only the credential fields differ.

## What You'll Need

- An AWS access key ID and secret access key
- The key should belong to an IAM user or role with at minimum: `iam:List*`, `iam:Get*`, `ec2:Describe*`, `s3:List*`
- For full change execution, see [AWS Connector Permissions](../connectors/aws.md#minimum-permissions-required)

## Step 1: Open Connector Settings

In the Nexplane UI, click **Settings** in the left sidebar, then **Connectors**, then **Add Connector**.

Select **AWS** from the connector type dropdown.

## Step 2: Enter Credentials

Fill in the connector form:

| Field | Example | Description |
|---|---|---|
| Name | `prod-aws` | A label for this connector -- shown in the UI |
| AWS Access Key ID | `AKIAIOSFODNN7EXAMPLE` | Your IAM access key |
| AWS Secret Access Key | `wJalrXUtnFEMI/K7MDENG/...` | Your IAM secret key |
| Default Region | `us-east-1` | Region used for regional API calls |
| Account ID | `123456789012` | Optional -- used to display account name |

Click **Save Connector**.

Nexplane encrypts the secret key before storing it. The plaintext secret is never written to disk or logged.

## Step 3: Test the Connection

After saving, click **Test Connection** on the connector row. Nexplane makes a `sts:GetCallerIdentity` call to verify the credentials are valid and returns the AWS account ID and ARN.

If the test fails, see [Connector Credential Errors](../runbooks/connector-credential-errors.md).

## Step 4: Run Asset Discovery

Click **Discover Assets** on the connector. Nexplane will enumerate:

- IAM users and their access keys
- IAM roles
- EC2 instances (all regions)
- S3 buckets
- Security groups

Discovery runs in the background. A spinner shows on the connector card. When it completes, click **View Assets** to see what was found.

!!! info "Discovery frequency"
    After the initial discovery, Nexplane re-discovers assets on a 24-hour schedule. You can trigger a manual re-discovery any time from the connector settings page.

## What's Stored

Nexplane stores asset metadata (IDs, names, regions, tags) -- not the asset contents themselves. No S3 object data, no EC2 user data, no secrets are pulled during discovery.

## Next Step

[Create your first change request](first-change-request.md)
