# GitHub Actions Lint

Validate GitHub Actions workflow files for syntax, anti-patterns, and obvious secret exposure.
Produces a pipeline validation artifact for CI/CD review workflows.

## PRECONDITIONS CHECK

### A.1 Verify Workflow Directory

```bash
if [ ! -d ".github/workflows" ]; then
  echo "ERROR: .github/workflows directory not found"
  exit 1
fi
```

### A.2 Verify Tool Availability

```bash
which actionlint || { echo "ERROR: actionlint not found"; exit 1; }
actionlint -version
```

**PRECONDITION_CHECKPOINT:**
- [ ] GitHub workflow directory exists
- [ ] actionlint available

## PHASE 1: RUN ACTIONLINT

```bash
actionlint -color > "$ARTIFACTS_DIR/github-actions-lint.txt" 2>&1
LINT_EXIT=$?
```

**CHECKPOINT:** Lint execution complete.

## PHASE 2: CHECK FOR SECRET ANTI-PATTERNS

```bash
SECRET_PATTERNS=$(rg -n "echo\\s+.*\\$\\{\\{\\s*secrets\\.|::add-mask::|password\\s*:" .github/workflows 2>/dev/null | wc -l | tr -d ' ')
TOTAL_ISSUES=$(grep -c ":" "$ARTIFACTS_DIR/github-actions-lint.txt" 2>/dev/null || echo "0")
GATE_RESULT=$([ "$LINT_EXIT" -eq 0 ] && [ "$SECRET_PATTERNS" -eq 0 ] && echo "pass" || echo "fail")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/github-actions-findings.json" << EOF
{
  "tool": "actionlint",
  "scan_target": ".github/workflows",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "summary": {
    "lint_issues": $TOTAL_ISSUES,
    "secret_antipatterns": $SECRET_PATTERNS
  },
  "gate_result": "$GATE_RESULT"
}
EOF
```

**CHECKPOINT:** GitHub Actions artifact written.

**EMIT:** `<promise>GITHUB_ACTIONS_$GATE_RESULT</promise>`
