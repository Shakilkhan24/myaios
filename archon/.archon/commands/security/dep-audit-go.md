# Dependency Audit — Go

Run `govulncheck` for Go projects. Produces unified security scan artifact.

## PHASE 1: RUN AUDIT

```bash
if which govulncheck > /dev/null 2>&1; then
  govulncheck -json ./... > "$ARTIFACTS_DIR/govulncheck-raw.json" 2>&1 || true
  TOTAL=$(jq '[.vulns[]?] | length' "$ARTIFACTS_DIR/govulncheck-raw.json" 2>/dev/null || echo "0")
else
  echo "WARNING: govulncheck not found"; TOTAL=0
fi

cat > "$ARTIFACTS_DIR/dep-audit-go.json" << EOF
{"tool":"govulncheck","scan_target":"go.mod","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","summary":{"total":$TOTAL,"critical":0,"high":$TOTAL,"medium":0,"low":0,"info":0},"findings":[],"gate_result":"$([ "$TOTAL" -gt "0" ] && echo "fail" || echo "pass")"}
EOF
```

**EMIT:** `<promise>GO_AUDIT_COMPLETE</promise>`
