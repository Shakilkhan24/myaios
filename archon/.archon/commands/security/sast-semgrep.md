# SAST Semgrep Scan

Run Semgrep static analysis security testing. Produces unified security scan artifact.

## PHASE 1: RUN SCAN

```bash
which semgrep || { echo "ERROR: semgrep not found"; exit 1; }
SEMGREP_VERSION=$(semgrep --version 2>/dev/null)

semgrep scan --config auto --json --output "$ARTIFACTS_DIR/semgrep-raw.json" . 2>&1 | tee "$ARTIFACTS_DIR/semgrep-scan.log"
```

## PHASE 2: NORMALIZE

```bash
TOTAL=$(jq '.results | length' "$ARTIFACTS_DIR/semgrep-raw.json" 2>/dev/null || echo "0")
CRITICAL=$(jq '[.results[] | select(.extra.severity == "ERROR")] | length' "$ARTIFACTS_DIR/semgrep-raw.json" 2>/dev/null || echo "0")
HIGH=$(jq '[.results[] | select(.extra.severity == "WARNING")] | length' "$ARTIFACTS_DIR/semgrep-raw.json" 2>/dev/null || echo "0")
MEDIUM=$(jq '[.results[] | select(.extra.severity == "INFO")] | length' "$ARTIFACTS_DIR/semgrep-raw.json" 2>/dev/null || echo "0")
BLOCK_SEVERITY="${SAST_BLOCK_SEVERITY:-HIGH}"
BLOCK_COUNT=0
case "$BLOCK_SEVERITY" in
  CRITICAL) BLOCK_COUNT=$CRITICAL ;;
  HIGH) BLOCK_COUNT=$((CRITICAL + HIGH)) ;;
  MEDIUM) BLOCK_COUNT=$((CRITICAL + HIGH + MEDIUM)) ;;
esac
GATE_RESULT=$([ "$BLOCK_COUNT" -gt "0" ] && echo "fail" || echo "pass")

cat > "$ARTIFACTS_DIR/sast-findings.json" << EOF
{"tool":"semgrep","version":"$SEMGREP_VERSION","scan_target":".","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","summary":{"total":$TOTAL,"critical":$CRITICAL,"high":$HIGH,"medium":$MEDIUM,"low":0,"info":0},"findings":[],"gate_result":"$GATE_RESULT","block_threshold":"$BLOCK_SEVERITY"}
EOF
```

**EMIT:** `<promise>SAST_$GATE_RESULT</promise>`
