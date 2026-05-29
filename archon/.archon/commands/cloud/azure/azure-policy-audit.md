# Azure Policy Audit

Review Azure Policy posture and prepare an audit artifact for governance follow-up.

## PRECONDITIONS CHECK

```bash
az account show --output json > "$ARTIFACTS_DIR/azure-account.json"
```

## PHASE 1: COLLECT POLICY STATE

```bash
az policy assignment list --output json > "$ARTIFACTS_DIR/azure-policy-assignments.json"
```

## PHASE 2: WRITE REPORT

Write:
- `$ARTIFACTS_DIR/azure-policy-audit.md`
- `$ARTIFACTS_DIR/azure-policy-audit.json`

Call out missing policy coverage and likely governance gaps.

**EMIT:** `<promise>AZURE_POLICY_AUDIT_READY</promise>`
