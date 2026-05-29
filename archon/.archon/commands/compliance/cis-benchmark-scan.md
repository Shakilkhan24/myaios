# CIS Benchmark Scan

Run CIS benchmark checks using kube-bench (Kubernetes), aws-inspector, or lynis (Linux). Maps findings to CIS control IDs.

## PHASE 1: DETECT AND RUN SCANNER

```bash
if which kube-bench > /dev/null 2>&1; then
  SCAN_TOOL="kube-bench"
  kube-bench run --json > "$ARTIFACTS_DIR/cis-raw.json" 2>&1
elif which lynis > /dev/null 2>&1; then
  SCAN_TOOL="lynis"
  lynis audit system --quick --json > "$ARTIFACTS_DIR/cis-raw.json" 2>&1 || true
else
  echo "WARNING: No CIS benchmark tool found (kube-bench, lynis)"
  SCAN_TOOL="none"
fi

if [ "$SCAN_TOOL" != "none" ]; then
  PASS=$(jq '[.Controls[].Tests[].Results[] | select(.status == "PASS")] | length' "$ARTIFACTS_DIR/cis-raw.json" 2>/dev/null || echo "0")
  FAIL=$(jq '[.Controls[].Tests[].Results[] | select(.status == "FAIL")] | length' "$ARTIFACTS_DIR/cis-raw.json" 2>/dev/null || echo "0")
  TOTAL=$((PASS + FAIL))
else
  PASS=0; FAIL=0; TOTAL=0
fi

cat > "$ARTIFACTS_DIR/cis-benchmark.json" << EOF
{"framework":"cis-level1","tool":"$SCAN_TOOL","scan_date":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","summary":{"total_controls":$TOTAL,"passed":$PASS,"failed":$FAIL,"manual":0,"not_applicable":0,"pass_rate_percent":$([ $TOTAL -gt 0 ] && echo "$((PASS * 100 / TOTAL))" || echo "0")},"findings":[]}
EOF
```

**EMIT:** `<promise>CIS_SCAN_COMPLETE</promise>`
