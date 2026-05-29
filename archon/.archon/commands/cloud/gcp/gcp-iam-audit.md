# GCP IAM Audit

Audit Google Cloud IAM policy state and write a least-privilege review artifact.

## PRECONDITIONS CHECK

```bash
gcloud auth list --filter=status:ACTIVE --format="value(account)" > "$ARTIFACTS_DIR/gcp-account.txt"
```

## PHASE 1: COLLECT POLICY DATA

```bash
gcloud projects get-iam-policy "$CLOUD_ACCOUNT_ID" --format=json > "$ARTIFACTS_DIR/gcp-iam-policy.json"
```

## PHASE 2: WRITE REPORT

Write:
- `$ARTIFACTS_DIR/gcp-iam-audit.md`
- `$ARTIFACTS_DIR/gcp-iam-audit.json`

Highlight broad principals, inherited risk, and role tightening opportunities.

**EMIT:** `<promise>GCP_IAM_AUDIT_READY</promise>`
