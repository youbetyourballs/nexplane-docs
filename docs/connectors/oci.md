# OCI Connector

The OCI connector uses the Oracle Cloud Infrastructure Python SDK to interact with OCI IAM, Compute, and Object Storage. It supports API key rotation, compute instance management, and security list modifications.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-oci`) |
| Tenancy OCID | string | Yes | OCID of your OCI tenancy |
| User OCID | string | Yes | OCID of the IAM user Nexplane authenticates as |
| Region | string | Yes | OCI region identifier (e.g., `us-ashburn-1`) |
| Fingerprint | string | Yes | Fingerprint of the API signing key |
| Private Key | string | Yes | PEM-encoded RSA private key (4096-bit recommended) |
| Compartment OCID | string | No | Limit discovery to a specific compartment |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate API Key | Uploads new API key, deletes old key | Delete new key, note old key cannot be restored |
| Lock IAM User | Adds a quota policy that blocks all user API calls | Remove the quota policy |
| Unlock IAM User | Removes the Nexplane quota policy | Re-add the quota policy |
| Stop Compute Instance | Stops a running OCI compute instance | Start the instance |
| Start Compute Instance | Starts a stopped compute instance | Stop the instance |
| Modify Security List | Adds or removes an ingress or egress rule | Reverse the rule change |

## Minimum Permissions Required

The IAM user must belong to a group with the following policies:

For discovery:
```
Allow group nexplane-discovery to read users in tenancy
Allow group nexplane-discovery to read instances in tenancy
Allow group nexplane-discovery to read buckets in tenancy
```

For full change execution:
```
Allow group nexplane-ops to manage users in tenancy
Allow group nexplane-ops to manage instances in tenancy
Allow group nexplane-ops to manage security-lists in tenancy
```

## Known Limitations

- OCI API keys are limited to 3 per user. If a user already has 3 keys, rotation will fail. Delete an unused key first.
- OCI compute stop/start operations are asynchronous. Nexplane polls for up to 15 minutes due to OCI's typically slower instance operations.
- Security list rules are order-independent (unlike AWS security groups, which are stateful). All rules are evaluated. Nexplane matches rules by protocol, port range, and CIDR for modification purposes.
- The connector currently requires a user-based API key. Instance Principal and Resource Principal authentication are on the roadmap.
