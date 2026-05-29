# GitLab CI Lint

Validate `.gitlab-ci.yml` syntax and flag risky patterns before pipeline changes land.
Writes a machine-readable report suitable for validation workflows and review comments.

## PRECONDITIONS CHECK

### A.1 Verify Pipeline File

```bash
if [ ! -f ".gitlab-ci.yml" ]; then
  echo "ERROR: .gitlab-ci.yml not found"
  exit 1
fi
```

### A.2 Verify Tool Availability

```bash
which yq || { echo "ERROR: yq not found"; exit 1; }
yq --version | head -1
```

**PRECONDITION_CHECKPOINT:**
- [ ] GitLab pipeline file exists
- [ ] yq available

## PHASE 1: VALIDATE YAML SHAPE

```bash
yq eval '.' .gitlab-ci.yml > "$ARTIFACTS_DIR/gitlab-ci-normalized.yml"
LINT_EXIT=$?
if [ "$LINT_EXIT" -ne 0 ]; then
  echo "ERROR: invalid GitLab CI YAML"
  exit 1
fi
```

**CHECKPOINT:** YAML parsed successfully.

## PHASE 2: SECURITY HEURISTICS

```bash
PRIVILEGED_COUNT=$(rg -n "privileged:\\s*true" .gitlab-ci.yml 2>/dev/null | wc -l | tr -d ' ')
CURL_PIPE_COUNT=$(rg -n "curl .+\\|\\s*(sh|bash)" .gitlab-ci.yml 2>/dev/null | wc -l | tr -d ' ')
TOKEN_ECHO_COUNT=$(rg -n "echo.+\\$[A-Z0-9_]*(TOKEN|SECRET|PASSWORD|KEY)" .gitlab-ci.yml 2>/dev/null | wc -l | tr -d ' ')
GATE_RESULT=$([ "$PRIVILEGED_COUNT" -eq 0 ] && [ "$CURL_PIPE_COUNT" -eq 0 ] && [ "$TOKEN_ECHO_COUNT" -eq 0 ] && echo "pass" || echo "fail")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/gitlab-ci-findings.json" << EOF
{
  "tool": "gitlab-ci-lint",
  "scan_target": ".gitlab-ci.yml",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "summary": {
    "privileged_jobs": $PRIVILEGED_COUNT,
    "curl_pipe_shell": $CURL_PIPE_COUNT,
    "token_echoes": $TOKEN_ECHO_COUNT
  },
  "gate_result": "$GATE_RESULT"
}
EOF
```

**CHECKPOINT:** GitLab CI artifact written.

**EMIT:** `<promise>GITLAB_CI_$GATE_RESULT</promise>`
