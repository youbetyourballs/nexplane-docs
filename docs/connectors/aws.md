# AWS Connector

The AWS connector uses the boto3 SDK to interact with AWS services. It supports asset discovery across IAM, EC2, S3, and VPC, and can execute identity, compute, and credential change types against the discovered assets.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-aws`) |
| AWS Access Key ID | string | Yes | IAM access key ID (`AKIA...`) |
| AWS Secret Access Key | string | Yes | IAM secret access key |
| Default Region | string | Yes | Default region for regional API calls (e.g., `us-east-1`) |
| Account ID | string | No | AWS account ID -- used for display only |
| Role ARN | string | No | If set, Nexplane assumes this role before making API calls |
| External ID | string | No | External ID for cross-account role assumption |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate IAM Access Key | Creates a new access key, deactivates the old one | Re-activate old key, deactivate new key |
| Delete IAM Access Key | Deletes an inactive access key | No rollback (deleted keys cannot be recovered) |
| Lock IAM User | Attaches a deny-all inline policy to the user | Remove the deny policy |
| Unlock IAM User | Removes the Nexplane deny-all inline policy | Re-attach the deny policy |
| Modify Security Group Rule | Adds or removes an inbound or outbound rule | Reverse the rule change |
| Snapshot EC2 Instance | Creates an EBS snapshot of all volumes | Delete the snapshot |
| Stop EC2 Instance | Stops a running instance | Start the instance |
| Start EC2 Instance | Starts a stopped instance | Stop the instance |
| Update S3 Bucket Policy | Replaces the bucket policy | Restore the previous policy |
| Block S3 Public Access | Enables the S3 block public access settings | Disable block public access (use with caution) |

## Minimum Permissions Required

For asset discovery only:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:ListUsers",
        "iam:ListAccessKeys",
        "iam:ListRoles",
        "iam:GetUser",
        "ec2:DescribeInstances",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeRegions",
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation",
        "sts:GetCallerIdentity"
      ],
      "Resource": "*"
    }
  ]
}
```

For full change execution, add:

```json
{
  "Effect": "Allow",
  "Action": [
    "iam:CreateAccessKey",
    "iam:UpdateAccessKey",
    "iam:DeleteAccessKey",
    "iam:PutUserPolicy",
    "iam:DeleteUserPolicy",
    "ec2:CreateSnapshot",
    "ec2:StopInstances",
    "ec2:StartInstances",
    "ec2:AuthorizeSecurityGroupIngress",
    "ec2:RevokeSecurityGroupIngress",
    "ec2:AuthorizeSecurityGroupEgress",
    "ec2:RevokeSecurityGroupEgress",
    "s3:PutBucketPolicy",
    "s3:PutBucketPublicAccessBlock"
  ],
  "Resource": "*"
}
```

## Known Limitations

- IAM access keys can only be rotated if the user currently has fewer than 2 active keys. If the user already has 2 keys, you must delete one before rotating.
- EC2 snapshots are created asynchronously. Nexplane waits up to 10 minutes for the snapshot to reach the `completed` state before reporting success.
- Security group rules are matched by the exact protocol, port, and CIDR. Rules that use references to other security groups (rather than CIDRs) are not currently supported for modification.
- Cross-region operations require the connector to be configured with appropriate permissions in each region. The `Default Region` field controls which region is used for regional calls.
- The connector uses the default boto3 retry configuration (3 retries with exponential backoff). Throttling errors from AWS will be retried automatically.
