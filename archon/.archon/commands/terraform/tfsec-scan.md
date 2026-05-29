# TFSec Scan

Run tfsec (or checkov) against Terraform code to identify security misconfigurations. Produces a structured findings artifact conforming to the unified security scan schema.

## PRECONDITIONS CHECK

```bash
# Check for tfsec or checkov
if which tfsec > /dev/null 2>&1; then
  SCAN_TOOL="tfsec"
  SCAN_VERSION=$(tfsec --version 2>/dev/null | head -1)
elif which checkov > /dev/null 2>&1; then
  SCAN_TOOL="checkov"
  SCAN_VERSION=$(checkov --version 2>/dev/null | head -1)
else
  echo "ERROR: Neither tfsec nor checkov found in PATH"
  exit 1
fi

echo "Using $SCAN_TOOL $SCAN_VERSION"
echo "Using $SCAN_TOOL $SCAN_VERSION" | tee -a $ARTIFACTS_DIR/tool-versions.txt
```

**PRECONDITION_CHECKPOINT:**
- [ ] Security scanner available

## PHASE 1: RUN SCAN

```bash
cd "$TF_ROOT"

if [ "$SCAN_TOOL" = "tfsec" ]; then
  tfsec . \
    --format json \
    --no-color \
    --out "$ARTIFACTS_DIR/tfsec-raw.json" \
    2>&1 | tee "$ARTIFACTS_DIR/tfsec-scan.log"
  SCAN_EXIT=$?
elif [ "$SCAN_TOOL" = "checkov" ]; then
  checkov \
    -d . \
    --output json \
    --quiet \
    > "$ARTIFACTS_DIR/tfsec-raw.json" 2>&1
  SCAN_EXIT=$?
fi
```

**CHECKPOINT:** Scan executed.

## PHASE 2: NORMALIZE FINDINGS

Transform tool-specific output into the unified security scan artifact schema:

```bash
# Parse findings based on tool
if [ "$SCAN_TOOL" = "tfsec" ]; then
  TOTAL=$(jq '.results | length' "$ARTIFACTS_DIR/tfsec-raw.json" 2>/dev/null || echo "0")
  CRITICAL=$(jq '[.results[] | select(.severity == "CRITICAL")] | length' "$ARTIFACTS_DIR/tfsec-raw.json" 2>/dev/null || echo "0")
  HIGH=$(jq '[.results[] | select(.severity == "HIGH")] | length' "$ARTIFACTS_DIR/tfsec-raw.json" 2>/dev/null || echo "0")
  MEDIUM=$(jq '[.results[] | select(.severity == "MEDIUM")] | length' "$ARTIFACTS_DIR/tfsec-raw.json" 2>/dev/null || echo "0")
  LOW=$(jq '[.results[] | select(.severity == "LOW")] | length' "$ARTIFACTS_DIR/tfsec-raw.json" 2>/dev/null || echo "0")
fi

# Determine gate result
BLOCK_SEVERITY="${SECURITY_BLOCK_SEVERITY:-HIGH}"
BLOCK_COUNT=0
case "$BLOCK_SEVERITY" in
  CRITICAL) BLOCK_COUNT=$CRITICAL ;;
  HIGH) BLOCK_COUNT=$((CRITICAL + HIGH)) ;;
  MEDIUM) BLOCK_COUNT=$((CRITICAL + HIGH + MEDIUM)) ;;
  LOW) BLOCK_COUNT=$((CRITICAL + HIGH + MEDIUM + LOW)) ;;
esac

GATE_RESULT=$([ "$BLOCK_COUNT" -gt "0" ] && echo "fail" || echo "pass")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/tfsec-findings.json" << EOF
{
  "tool": "$SCAN_TOOL",
  "version": "$SCAN_VERSION",
  "scan_target": "$TF_ROOT",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "summary": {
    "total": $TOTAL,
    "critical": $CRITICAL,
    "high": $HIGH,
    "medium": $MEDIUM,
    "low": $LOW,
    "info": 0
  },
  "findings": [],
  "gate_result": "$GATE_RESULT",
  "block_threshold": "$BLOCK_SEVERITY",
  "block_count": $BLOCK_COUNT
}
EOF
```

**CHECKPOINT:** Security scan complete. Gate result: $GATE_RESULT

**EMIT:** `<promise>TFSEC_$GATE_RESULT</promise>`
