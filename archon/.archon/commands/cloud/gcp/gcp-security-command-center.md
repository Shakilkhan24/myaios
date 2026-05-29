# GCP Security Command Center

Summarize available Security Command Center findings for the configured GCP scope.

## PRECONDITIONS CHECK

```bash
gcloud auth list --filter=status:ACTIVE --format="value(account)" > "$ARTIFACTS_DIR/gcp-account.txt"
```

## PHASE 1: WRITE SCC REPORT

Write:
- `$ARTIFACTS_DIR/gcp-scc-report.md`
- `$ARTIFACTS_DIR/gcp-scc-report.json`

Focus on highest-severity findings and recommended next actions.

**EMIT:** `<promise>GCP_SCC_READY</promise>`
