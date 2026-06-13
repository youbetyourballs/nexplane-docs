# Runbook: Rollback Failure

Use this runbook when:
- A rollback operation fails with an error
- The Rollback button is not available on a completed change
- A rollback succeeds but the system does not return to the expected state

---

## Understanding Nexplane Rollback

Nexplane rollback is a typed inverse operation, not a backup restore. Each change type defines its own rollback:

- **IAM key rotation:** Re-activate the old key, deactivate the new key
- **Account lock:** Unlock the account
- **Service disable:** Re-enable and start the service
- **File permission:** Restore the previous mode and ownership

This means rollback can fail for the same reasons the original change can fail (network errors, permission issues, state drift).

---

## Symptom 1: Rollback button is not available

Some change types do not support rollback because the original operation is irreversible:

| Change Type | Rollback Available? | Reason |
|---|---|---|
| Delete IAM Access Key | No | Deleted keys cannot be recovered |
| Rotate GCP Service Account Key | Partial | Old key was deleted; new key can be disabled |
| Rotate Client Secret (Azure/Keycloak) | No | Old secret was not stored |
| Reset Database Password | No | Old password was not stored |
| Revoke Vault Token | No | Revocation is irreversible |
| Revoke Exposed Credential | No | Intentionally irreversible |

If rollback is not available for your change type, you must create a new change request to manually return to the desired state.

---

## Symptom 2: Rollback fails with "credential no longer valid"

```
ERROR: rollback failed: IAM key AKIAIOSFODNN7EXAMPLE cannot be activated: 
  NoSuchEntityException: The Access Key with id AKIAIOSFODNN7EXAMPLE cannot be found
```

The original credential was deleted after the change was made. Nexplane can only re-activate a deactivated key -- it cannot recreate a deleted one.

**Resolution:** Create a new access key for the user using a new change request (Credentials > Rotate IAM Access Key).

---

## Symptom 3: Rollback fails with "resource state has changed"

```
ERROR: rollback failed: security group rule sg-rule-12345 no longer exists
```

The resource was modified or deleted by another process (manual change, Terraform run, another tool) after the Nexplane change was made. The rollback cannot proceed because the state it expected no longer matches reality.

**Resolution:** Review the current state of the resource and manually restore it to the desired configuration. Record the manual change in Nexplane as a note on the change request.

---

## Symptom 4: Rollback fails with a permission error

```
ERROR: rollback failed: AccessDenied: User: arn:aws:iam::123456789012:user/nexplane 
  is not authorized to perform: iam:UpdateAccessKey on resource: ...
```

The connector credentials do not have permission to perform the rollback operation. This can happen if:
- The IAM policy was changed after the connector was created
- The connector credentials were rotated and the new credentials have fewer permissions

**Resolution:** Update the connector credentials to a key with the required permissions. See [AWS Connector Permissions](../connectors/aws.md#required-permissions).

---

## Symptom 5: Partial rollback -- some steps succeeded, some failed

CIS hardening profiles and other multi-step changes report per-step rollback results. If some steps fail during rollback:

1. On the change detail page, expand **Rollback Details** to see which steps succeeded and which failed
2. For each failed step, the error message explains why
3. Failed rollback steps must be resolved manually
4. Click **Mark as Resolved** on each failed step after manually restoring the state

---

## Symptom 6: Rollback succeeded but system behavior is wrong

Rollback restored the configuration as recorded by Nexplane, but the system is not behaving as expected before the change.

Possible causes:

- A dependent system was updated in the window between the change and the rollback
- The pre-change snapshot did not capture all relevant state (for example, a service that was already failing before the change)
- The rollback restored the configuration but did not restart a dependent service

**Resolution:** Review the pre-change state recorded in the change detail page and compare it against the current system state. If there is a gap, use additional change requests to bridge it.
