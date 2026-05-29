# SAST CodeQL Scan

Run CodeQL static analysis. Produces unified security scan artifact.

## PHASE 1: RUN SCAN

```bash
which codeql || { echo "WARNING: codeql not found — skipping"; exit 0; }

codeql database create "$ARTIFACTS_DIR/codeql-db" --language=javascript --source-root=. 2>&1 | tee "$ARTIFACTS_DIR/codeql-create.log"
codeql database analyze "$ARTIFACTS_DIR/codeql-db" --format=sarif-latest --output="$ARTIFACTS_DIR/codeql-raw.sarif" 2>&1 | tee "$ARTIFACTS_DIR/codeql-analyze.log"
```

## PHASE 2: NORMALIZE

```bash
TOTAL=$(jq '.runs[0].results | length' "$ARTIFACTS_DIR/codeql-raw.sarif" 2>/dev/null || echo "0")
CRITICAL=$(jq '[.runs[0].results[] | select(.level == "error")] | length' "$ARTIFACTS_DIR/codeql-raw.sarif" 2>/dev/null || echo "0")
HIGH=$(jq '[.runs[0].results[] | select(.level == "warning")] | length' "$ARTIFACTS_DIR/codeql-raw.sarif" 2>/dev/null || echo "0")
GATE_RESULT=$([ "$CRITICAL" -gt "0" ] && echo "fail" || echo "pass")

cat > "$ARTIFACTS_DIR/codeql-findings.json" << EOF
{"tool":"codeql","scan_target":".","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","summary":{"total":$TOTAL,"critical":$CRITICAL,"high":$HIGH,"medium":0,"low":0,"info":0},"findings":[],"gate_result":"$GATE_RESULT"}
EOF
```

**EMIT:** `<promise>CODEQL_$GATE_RESULT</promise>`
