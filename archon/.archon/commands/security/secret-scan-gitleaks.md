# Secret Scan — Gitleaks

Scan repository for secrets using Gitleaks. Alternative to TruffleHog.

## PHASE 1: RUN SCAN

```bash
which gitleaks || { echo "ERROR: gitleaks not found"; exit 1; }
GITLEAKS_VERSION=$(gitleaks version 2>/dev/null)

gitleaks detect --source . --report-format json --report-path "$ARTIFACTS_DIR/gitleaks-raw.json" 2>&1 | tee "$ARTIFACTS_DIR/gitleaks-scan.log"
SCAN_EXIT=$?
TOTAL=$(jq 'length' "$ARTIFACTS_DIR/gitleaks-raw.json" 2>/dev/null || echo "0")

cat > "$ARTIFACTS_DIR/gitleaks-findings.json" << EOF
{"tool":"gitleaks","version":"$GITLEAKS_VERSION","scan_target":".","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","summary":{"total":$TOTAL,"critical":$TOTAL,"high":0,"medium":0,"low":0,"info":0},"findings":[],"gate_result":"$([ "$TOTAL" -gt "0" ] && echo "fail" || echo "pass")"}
EOF
```

**EMIT:** `<promise>GITLEAKS_COMPLETE</promise>`
