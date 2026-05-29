# AWS IAM Audit

Audit AWS IAM roles and policies for overly broad access and write a remediation-oriented
artifact for security review.

## PRECONDITIONS CHECK

```bash
aws sts get-caller-identity --output json > "$ARTIFACTS_DIR/aws-identity.json"
```

**PRECONDITION_CHECKPOINT:**
- [ ] AWS credentials valid

## PHASE 1: COLLECT IAM DATA

```bash
aws iam list-roles --output json > "$ARTIFACTS_DIR/aws-iam-roles.json"
aws iam list-policies --scope Local --output json > "$ARTIFACTS_DIR/aws-iam-policies.json"
```

## PHASE 2: WRITE REPORT

Write:
- `$ARTIFACTS_DIR/aws-iam-audit.md`
- `$ARTIFACTS_DIR/aws-iam-audit.json`

Focus on wildcard permissions, stale identities, and least-privilege gaps.

**EMIT:** `<promise>AWS_IAM_AUDIT_READY</promise>`
