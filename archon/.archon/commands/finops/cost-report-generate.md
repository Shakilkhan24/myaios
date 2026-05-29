# Cost Report Generate

Generate a cloud cost report with current totals, major drivers, and team or environment
breakdowns so FinOps reviews start from a structured baseline.

## PRECONDITIONS CHECK

### A.1 Validate Cost Context

```bash
TARGET=${ARGUMENTS:-$ENVIRONMENT}
if [ -z "$TARGET" ]; then
  echo "ERROR: cost target could not be resolved"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Cost target resolved

## PHASE 1: BUILD REPORT INPUT

Write a report request artifact to `$ARTIFACTS_DIR/cost-report-request.json` including:
1. target environment or scope
2. cloud provider
3. reporting currency
4. monthly budget limit if configured

**CHECKPOINT:** Report request artifact written.

## PHASE 2: GENERATE COST REPORT

Write:
- `$ARTIFACTS_DIR/cost-report.md`
- `$ARTIFACTS_DIR/cost-report.json`

Include:
1. current monthly total
2. notable deltas or anomalies
3. top cost drivers
4. budget risk and next action

**CHECKPOINT:** Cost report artifacts written.

**EMIT:** `<promise>COST_REPORT_READY</promise>`
