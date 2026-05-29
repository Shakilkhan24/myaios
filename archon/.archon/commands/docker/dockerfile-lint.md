# Dockerfile Lint

Lint a Dockerfile using hadolint for best practices. Produces structured findings.

## PHASE 1: RUN LINT

```bash
DOCKERFILE="${DOCKERFILE:-Dockerfile}"
if which hadolint > /dev/null 2>&1; then
  hadolint "$DOCKERFILE" --format json > "$ARTIFACTS_DIR/dockerfile-lint-raw.json" 2>&1
  LINT_EXIT=$?
  FINDINGS=$(jq 'length' "$ARTIFACTS_DIR/dockerfile-lint-raw.json" 2>/dev/null || echo "0")
  ERRORS=$(jq '[.[] | select(.level == "error")] | length' "$ARTIFACTS_DIR/dockerfile-lint-raw.json" 2>/dev/null || echo "0")
  WARNINGS=$(jq '[.[] | select(.level == "warning")] | length' "$ARTIFACTS_DIR/dockerfile-lint-raw.json" 2>/dev/null || echo "0")
else
  echo "WARNING: hadolint not found — skipping Dockerfile lint"
  FINDINGS=0; ERRORS=0; WARNINGS=0
fi

cat > "$ARTIFACTS_DIR/dockerfile-lint.json" << EOF
{"tool":"hadolint","dockerfile":"$DOCKERFILE","findings":$FINDINGS,"errors":$ERRORS,"warnings":$WARNINGS,"gate_result":"$([ "$ERRORS" -gt "0" ] && echo "fail" || echo "pass")","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)"}
EOF
```

**EMIT:** `<promise>LINT_COMPLETE</promise>`
