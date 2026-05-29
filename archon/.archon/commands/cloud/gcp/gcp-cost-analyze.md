# GCP Cost Analyze

Prepare a GCP cost analysis summary for the configured project or billing scope.

## PRECONDITIONS CHECK

```bash
gcloud auth list --filter=status:ACTIVE --format="value(account)" > "$ARTIFACTS_DIR/gcp-account.txt"
```

## PHASE 1: WRITE ANALYSIS

Write:
- `$ARTIFACTS_DIR/gcp-cost-analysis.md`
- `$ARTIFACTS_DIR/gcp-cost-analysis.json`

Summarize major cost drivers, anomaly risk, and likely optimization areas.

**EMIT:** `<promise>GCP_COST_ANALYSIS_READY</promise>`
