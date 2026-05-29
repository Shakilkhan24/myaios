# Azure Security Center

Summarize Defender for Cloud / Security Center posture for the active subscription.

## PRECONDITIONS CHECK

```bash
az account show --output json > "$ARTIFACTS_DIR/azure-account.json"
```

## PHASE 1: WRITE REPORT

Write:
- `$ARTIFACTS_DIR/azure-security-center.md`
- `$ARTIFACTS_DIR/azure-security-center.json`

Highlight critical recommendations and immediate remediation areas.

**EMIT:** `<promise>AZURE_SECURITY_CENTER_READY</promise>`
