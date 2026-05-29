# Right-Sizing Analyze

Analyze resource sizing pressure and recommend safe downsize or rebalance actions based on
available capacity and utilization evidence.

## PRECONDITIONS CHECK

### A.1 Validate Scope

```bash
TARGET=${ARGUMENTS:-$STACK}
if [ -z "$TARGET" ]; then
  echo "ERROR: right-sizing target required"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Right-sizing target resolved

## PHASE 1: CAPTURE ANALYSIS CONTEXT

Write a context artifact to `$ARTIFACTS_DIR/right-sizing-context.json` describing:
1. target service or environment
2. known traffic pattern assumptions
3. whether utilization metrics are already available

**CHECKPOINT:** Right-sizing context written.

## PHASE 2: PRODUCE RIGHT-SIZING REPORT

Write:
- `$ARTIFACTS_DIR/right-sizing-report.md`
- `$ARTIFACTS_DIR/right-sizing-report.json`

Include:
1. candidates safe to downsize now
2. candidates needing more metrics before change
3. the expected savings confidence level
4. risk notes for each recommendation

**CHECKPOINT:** Right-sizing report artifacts written.

**EMIT:** `<promise>RIGHT_SIZING_READY</promise>`
