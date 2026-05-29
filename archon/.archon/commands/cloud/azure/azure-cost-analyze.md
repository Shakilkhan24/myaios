# Azure Cost Analyze

Prepare an Azure cost analysis summary for the active subscription.

## PRECONDITIONS CHECK

```bash
az account show --output json > "$ARTIFACTS_DIR/azure-account.json"
```

## PHASE 1: WRITE ANALYSIS

Write:
- `$ARTIFACTS_DIR/azure-cost-analysis.md`
- `$ARTIFACTS_DIR/azure-cost-analysis.json`

Summarize top spend categories, budget concerns, and savings candidates.

**EMIT:** `<promise>AZURE_COST_ANALYSIS_READY</promise>`
