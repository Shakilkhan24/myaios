# Grype Container Scan

Alternative container scanner using Grype. Produces unified security scan artifact compatible with Trivy output.

## PHASE 1: RUN SCAN

```bash
which grype || { echo "ERROR: grype not found"; exit 1; }
GRYPE_VERSION=$(grype version 2>/dev/null | grep -oP '\d+\.\d+\.\d+' | head -1)
IMAGE_FULL="${IMAGE_REPO}:${IMAGE_TAG}"

grype "$IMAGE_FULL" -o json > "$ARTIFACTS_DIR/grype-raw.json" 2>&1
```

## PHASE 2: NORMALIZE

```bash
CRITICAL=$(jq '[.matches[] | select(.vulnerability.severity == "Critical")] | length' "$ARTIFACTS_DIR/grype-raw.json" 2>/dev/null || echo "0")
HIGH=$(jq '[.matches[] | select(.vulnerability.severity == "High")] | length' "$ARTIFACTS_DIR/grype-raw.json" 2>/dev/null || echo "0")
MEDIUM=$(jq '[.matches[] | select(.vulnerability.severity == "Medium")] | length' "$ARTIFACTS_DIR/grype-raw.json" 2>/dev/null || echo "0")
LOW=$(jq '[.matches[] | select(.vulnerability.severity == "Low")] | length' "$ARTIFACTS_DIR/grype-raw.json" 2>/dev/null || echo "0")
TOTAL=$((CRITICAL + HIGH + MEDIUM + LOW))

BLOCK_SEVERITY="${SECURITY_BLOCK_SEVERITY:-HIGH}"
BLOCK_COUNT=0
case "$BLOCK_SEVERITY" in
  CRITICAL) BLOCK_COUNT=$CRITICAL ;;
  HIGH) BLOCK_COUNT=$((CRITICAL + HIGH)) ;;
  MEDIUM) BLOCK_COUNT=$((CRITICAL + HIGH + MEDIUM)) ;;
esac
GATE_RESULT=$([ "$BLOCK_COUNT" -gt "0" ] && echo "fail" || echo "pass")

cat > "$ARTIFACTS_DIR/grype-scan.json" << EOF
{"tool":"grype","version":"$GRYPE_VERSION","scan_target":"$IMAGE_FULL","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","summary":{"total":$TOTAL,"critical":$CRITICAL,"high":$HIGH,"medium":$MEDIUM,"low":$LOW,"info":0},"findings":[],"gate_result":"$GATE_RESULT","block_threshold":"$BLOCK_SEVERITY"}
EOF
```

**EMIT:** `<promise>GRYPE_$GATE_RESULT</promise>`
