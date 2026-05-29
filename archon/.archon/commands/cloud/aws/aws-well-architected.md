# AWS Well-Architected

Prepare a lightweight Well-Architected review summary for the current workload.

## PRECONDITIONS CHECK

```bash
aws sts get-caller-identity --output json > "$ARTIFACTS_DIR/aws-identity.json"
```

## PHASE 1: WRITE ASSESSMENT

Write:
- `$ARTIFACTS_DIR/aws-well-architected.md`
- `$ARTIFACTS_DIR/aws-well-architected.json`

Cover the six pillars briefly and identify the highest-risk review area.

**EMIT:** `<promise>AWS_WELL_ARCHITECTED_READY</promise>`
