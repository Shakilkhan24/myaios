# SLO Status Report

Generate an SLO status report from available OpenSLO or Sloth inputs and summarize current
error budget health for the target service or environment.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
TARGET=${ARGUMENTS:-$STACK}
if [ -z "$TARGET" ]; then
  echo "ERROR: service or SLO target required"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Target service resolved

## PHASE 1: COLLECT SLO MATERIAL

```bash
find . -type f \( -name "*openslo*.yaml" -o -name "*sloth*.yaml" -o -name "*slo*.yaml" \) > "$ARTIFACTS_DIR/slo-source-files.txt"
SOURCE_COUNT=$(Get-Content "$ARTIFACTS_DIR/slo-source-files.txt" | Measure-Object | Select-Object -ExpandProperty Count)
echo "{\"target\":\"$TARGET\",\"source_count\":$SOURCE_COUNT}" > "$ARTIFACTS_DIR/slo-source-summary.json"
```

**CHECKPOINT:** SLO source inventory captured.

## PHASE 2: GENERATE STATUS REPORT

Read the available source inventory and create a status summary.

Write the following artifacts:
- `$ARTIFACTS_DIR/slo-status-report.md`
- `$ARTIFACTS_DIR/slo-status-report.json`

The report must include:
1. Target service or environment
2. Known SLO source files found in the repo
3. Current risk level: healthy, watch, or exhausted
4. Recommended next action

If no SLO source files are found, state that explicitly and mark the status as `unknown`.

**CHECKPOINT:** SLO report artifacts written.

**EMIT:** `<promise>SLO_STATUS_READY</promise>`
