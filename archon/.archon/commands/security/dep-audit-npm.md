# Dependency Audit — npm

Run `npm audit` for JavaScript/Node.js projects. Produces unified security scan artifact.

## PHASE 1: RUN AUDIT

```bash
which npm || { echo "ERROR: npm not found"; exit 1; }
npm audit --json > "$ARTIFACTS_DIR/npm-audit-raw.json" 2>&1 || true

CRITICAL=$(jq '.metadata.vulnerabilities.critical // 0' "$ARTIFACTS_DIR/npm-audit-raw.json" 2>/dev/null || echo "0")
HIGH=$(jq '.metadata.vulnerabilities.high // 0' "$ARTIFACTS_DIR/npm-audit-raw.json" 2>/dev/null || echo "0")
MEDIUM=$(jq '.metadata.vulnerabilities.moderate // 0' "$ARTIFACTS_DIR/npm-audit-raw.json" 2>/dev/null || echo "0")
LOW=$(jq '.metadata.vulnerabilities.low // 0' "$ARTIFACTS_DIR/npm-audit-raw.json" 2>/dev/null || echo "0")
TOTAL=$((CRITICAL + HIGH + MEDIUM + LOW))

cat > "$ARTIFACTS_DIR/dep-audit-npm.json" << EOF
{"tool":"npm-audit","scan_target":"package.json","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","summary":{"total":$TOTAL,"critical":$CRITICAL,"high":$HIGH,"medium":$MEDIUM,"low":$LOW,"info":0},"findings":[],"gate_result":"$([ "$CRITICAL" -gt "0" ] && echo "fail" || echo "pass")"}
EOF
```

**EMIT:** `<promise>NPM_AUDIT_COMPLETE</promise>`
