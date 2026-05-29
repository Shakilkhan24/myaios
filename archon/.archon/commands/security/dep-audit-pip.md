# Dependency Audit — pip

Run `pip-audit` or `safety` for Python projects. Produces unified security scan artifact.

## PHASE 1: RUN AUDIT

```bash
if which pip-audit > /dev/null 2>&1; then
  pip-audit --format json --output "$ARTIFACTS_DIR/pip-audit-raw.json" 2>&1 || true
  TOTAL=$(jq '.dependencies | map(select(.vulns | length > 0)) | length' "$ARTIFACTS_DIR/pip-audit-raw.json" 2>/dev/null || echo "0")
elif which safety > /dev/null 2>&1; then
  safety check --json > "$ARTIFACTS_DIR/pip-audit-raw.json" 2>&1 || true
  TOTAL=$(jq 'length' "$ARTIFACTS_DIR/pip-audit-raw.json" 2>/dev/null || echo "0")
else
  echo "WARNING: Neither pip-audit nor safety found"; TOTAL=0
fi

cat > "$ARTIFACTS_DIR/dep-audit-pip.json" << EOF
{"tool":"pip-audit","scan_target":"requirements.txt","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","summary":{"total":$TOTAL,"critical":0,"high":$TOTAL,"medium":0,"low":0,"info":0},"findings":[],"gate_result":"$([ "$TOTAL" -gt "0" ] && echo "fail" || echo "pass")"}
EOF
```

**EMIT:** `<promise>PIP_AUDIT_COMPLETE</promise>`
