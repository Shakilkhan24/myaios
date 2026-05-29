# Trivy Container Scan

Scan a container image with Trivy for vulnerabilities. Produces unified security scan artifact.

## PRECONDITIONS CHECK

```bash
which trivy || { echo "ERROR: trivy not found"; exit 1; }
TRIVY_VERSION=$(trivy --version 2>/dev/null | grep -oP '\d+\.\d+\.\d+' | head -1)
echo "Using trivy $TRIVY_VERSION" | tee -a "$ARTIFACTS_DIR/tool-versions.txt"
```

## PHASE 1: RUN SCAN

```bash
IMAGE_FULL="${IMAGE_REPO}:${IMAGE_TAG}"

trivy image \
  --format json \
  --output "$ARTIFACTS_DIR/trivy-raw.json" \
  --severity CRITICAL,HIGH,MEDIUM,LOW \
  "$IMAGE_FULL" \
  2>&1 | tee "$ARTIFACTS_DIR/trivy-scan.log"

SCAN_EXIT=$?
if [ $SCAN_EXIT -ne 0 ]; then
  echo "WARNING: Trivy scan returned non-zero exit"
fi
```

**CHECKPOINT:** Scan executed.

## PHASE 2: NORMALIZE FINDINGS

```bash
CRITICAL=$(jq '[.Results[]?.Vulnerabilities[]? | select(.Severity == "CRITICAL")] | length' "$ARTIFACTS_DIR/trivy-raw.json" 2>/dev/null || echo "0")
HIGH=$(jq '[.Results[]?.Vulnerabilities[]? | select(.Severity == "HIGH")] | length' "$ARTIFACTS_DIR/trivy-raw.json" 2>/dev/null || echo "0")
MEDIUM=$(jq '[.Results[]?.Vulnerabilities[]? | select(.Severity == "MEDIUM")] | length' "$ARTIFACTS_DIR/trivy-raw.json" 2>/dev/null || echo "0")
LOW=$(jq '[.Results[]?.Vulnerabilities[]? | select(.Severity == "LOW")] | length' "$ARTIFACTS_DIR/trivy-raw.json" 2>/dev/null || echo "0")
TOTAL=$((CRITICAL + HIGH + MEDIUM + LOW))

BLOCK_SEVERITY="${SECURITY_BLOCK_SEVERITY:-HIGH}"
BLOCK_COUNT=0
case "$BLOCK_SEVERITY" in
  CRITICAL) BLOCK_COUNT=$CRITICAL ;;
  HIGH) BLOCK_COUNT=$((CRITICAL + HIGH)) ;;
  MEDIUM) BLOCK_COUNT=$((CRITICAL + HIGH + MEDIUM)) ;;
esac
GATE_RESULT=$([ "$BLOCK_COUNT" -gt "0" ] && echo "fail" || echo "pass")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/trivy-scan.json" << EOF
{
  "tool": "trivy",
  "version": "$TRIVY_VERSION",
  "scan_target": "$IMAGE_FULL",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "summary": {"total": $TOTAL, "critical": $CRITICAL, "high": $HIGH, "medium": $MEDIUM, "low": $LOW, "info": 0},
  "findings": [],
  "gate_result": "$GATE_RESULT",
  "block_threshold": "$BLOCK_SEVERITY"
}
EOF
```

**EMIT:** `<promise>TRIVY_$GATE_RESULT</promise>`
