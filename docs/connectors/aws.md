# AWS Connector

The AWS connector uses the boto3 SDK to interact with AWS services. It supports asset discovery and change execution across EC2, IAM, S3, Route53, RDS, CloudWatch, ALB, and more.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name (e.g., `prod-aws`) |
| AWS Access Key ID | Yes | IAM access key ID (`AKIA...`) |
| AWS Secret Access Key | Yes | IAM secret access key |
| Default Region | Yes | Default region for regional API calls (e.g., `us-east-1`) |
| Account ID | No | AWS account ID — for display only |
| Role ARN | No | If set, Nexplane assumes this role before making API calls |
| External ID | No | External ID for cross-account role assumption |

## Required Permissions

Create an IAM user with programmatic access and attach these managed policies:

- `AmazonEC2FullAccess`
- `AmazonSSMFullAccess`
- `IAMFullAccess`
- `AmazonRDSFullAccess`
- `AmazonRoute53FullAccess`
- `AmazonS3FullAccess`
- `CloudWatchFullAccess`

Also attach or create a policy granting `sts:GetCallerIdentity`.

!!! note "AWS Free Tier"
    Free Tier accounts cannot launch Windows EC2 instances. Windows Server AMIs require a paid account.

## Capabilities

### EC2

| Action | Description | Rollback |
|--------|-------------|---------|
| `ec2_launch` | Launch an EC2 instance | Terminate the instance |
| `ec2_start` | Start a stopped instance | Stop the instance |
| `ec2_stop` | Stop a running instance | Start the instance |
| `ec2_reboot` | Reboot an instance | N/A |
| `ec2_terminate` | Terminate an instance | Not available |
| `snapshot_asset` | Create EBS snapshot of all attached volumes | Delete snapshots |
| `deploy_nexplane_agent` | Install Nexplane Agent via SSM | Uninstall agent |

### Key Pairs

| Action | Description | Rollback |
|--------|-------------|---------|
| `key_pair_create` | Create an EC2 key pair | Delete the key pair |
| `key_pair_delete` | Delete an EC2 key pair | Not available |

### IAM

| Action | Description | Rollback |
|--------|-------------|---------|
| `iam_user_create` | Create an IAM user | Delete the user |
| `iam_user_delete` | Delete an IAM user | Not available |
| `rotate_api_key` | Create new IAM access key, deactivate old | Re-activate old key |
| Attach/detach policy | Attach or detach managed policy | Reverse attachment |

### S3

| Action | Description | Rollback |
|--------|-------------|---------|
| `s3_bucket_create` | Create an S3 bucket | Delete the bucket |
| `s3_bucket_delete` | Delete an S3 bucket | Not available |
| `s3_lifecycle_configure` | Configure lifecycle rules | Restore previous rules |
| Block public access | Configure public access block settings | Restore previous settings |
| Bucket policy | Apply or remove bucket policy | Restore previous policy |

### Route53

| Action | Description | Rollback |
|--------|-------------|---------|
| `route53_zone_create` | Create a hosted zone | Delete the zone |
| `route53_record_upsert` | Create or update a DNS record | Delete or restore record |
| `route53_record_delete` | Delete a DNS record | Recreate the record |
| DR failover | Update weighted routing for DR | Restore original weights |

### RDS

| Action | Description | Rollback |
|--------|-------------|---------|
| `rds_instance_create` | Create an RDS instance | Delete the instance |
| `rds_instance_delete` | Delete an RDS instance | Not available |
| `rds_snapshot_create` | Create an RDS snapshot | Delete the snapshot |
| `promote_db_replica` | Promote a read replica to standalone + update Route53 CNAME | Not available (`rollback_supported: false`) |

### CloudWatch

| Action | Description | Rollback |
|--------|-------------|---------|
| `cloudwatch_alarm_create` | Create a CloudWatch alarm | Delete the alarm |
| `cloudwatch_alarm_delete` | Delete a CloudWatch alarm | Recreate the alarm |

### ALB

| Action | Description |
|--------|-------------|
| Create/delete ALB | Create or delete an Application Load Balancer |
| Target groups + listeners | Create, delete, modify, register targets |

### Network

| Action | Description | Rollback |
|--------|-------------|---------|
| `security_group_update` | Add or remove security group rules | Restore original ruleset |
| `tailscale_join` | Install Tailscale on an EC2 instance via SSM and join the tailnet | `tailscale_remove` |
| `tailscale_remove` | Remove the instance from the Tailscale tailnet | N/A |

### SSM

| Action | Description |
|--------|-------------|
| `ssm_command` | Run an approved SSM document against an EC2 instance |

Only allow-listed SSM documents can be executed — freeform shell is blocked by the safety engine.

## Cross-Account Access

For cross-account scenarios, set **Role ARN** and optionally **External ID**. Nexplane will call `sts:AssumeRole` before making any AWS API calls. The base IAM user only needs `sts:AssumeRole` permission.
