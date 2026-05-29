# AWS Security Hub Report

Retrieve or summarize AWS Security Hub findings for the current account and region.

## PRECONDITIONS CHECK

```bash
aws sts get-caller-identity --output json > "$ARTIFACTS_DIR/aws-identity.json"
```

## PHASE 1: COLLECT FINDINGS

```bash
aws securityhub get-findings --max-results 50 --output json > "$ARTIFACTS_DIR/aws-security-hub-findings.json"
```

## PHASE 2: WRITE REPORT

Write:
- `$ARTIFACTS_DIR/aws-security-hub-report.md`
- `$ARTIFACTS_DIR/aws-security-hub-report.json`

Summarize severity distribution, top controls, and urgent remediation.

**EMIT:** `<promise>AWS_SECURITY_HUB_READY</promise>`
