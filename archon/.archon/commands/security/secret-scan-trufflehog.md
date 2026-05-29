# Secret Scan — TruffleHog

Scan repository for exposed secrets using TruffleHog. Detects API keys, tokens, passwords in code and git history.

## PHASE 1: RUN SCAN

```bash
which trufflehog || { echo "ERROR: trufflehog not found"; exit 1; }

trufflehog filesystem . --json > "$ARTIFACTS_DIR/trufflehog-raw.json" 2>&1
SCAN_EXIT=$?
TOTAL=$(wc -l < "$ARTIFACTS_DIR/trufflehog-raw.json" 2>/dev/null || echo "0")
# Each line is a finding in JSONL format
VERIFIED=$(grep -c '"Verified":true' "$ARTIFACTS_DIR/trufflehog-raw.json" 2>/dev/null || echo "0")

cat > "$ARTIFACTS_DIR/secret-scan.json" << EOF
{"tool":"trufflehog","scan_target":".","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","summary":{"total":$TOTAL,"critical":$VERIFIED,"high":$((TOTAL - VERIFIED)),"medium":0,"low":0,"info":0},"findings":[],"gate_result":"$([ "$VERIFIED" -gt "0" ] && echo "fail" || echo "pass")","block_threshold":"CRITICAL"}
EOF
```

**EMIT:** `<promise>SECRET_SCAN_COMPLETE</promise>`
