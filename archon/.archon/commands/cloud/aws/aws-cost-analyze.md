# AWS Cost Analyze

Collect AWS cost context and prepare a structured cost analysis artifact.

## PRECONDITIONS CHECK

```bash
aws sts get-caller-identity --output json > "$ARTIFACTS_DIR/aws-identity.json"
```

## PHASE 1: REQUEST COST DATA

Write:
- `$ARTIFACTS_DIR/aws-cost-analysis.md`
- `$ARTIFACTS_DIR/aws-cost-analysis.json`

Include current cost drivers, budget risk, and likely optimization opportunities.

**EMIT:** `<promise>AWS_COST_ANALYSIS_READY</promise>`
