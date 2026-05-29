# Jenkins Lint

Validate a Jenkinsfile and flag insecure or brittle pipeline constructs before merge.
Produces an audit artifact that review workflows can consume.

## PRECONDITIONS CHECK

### A.1 Verify Jenkinsfile Exists

```bash
if [ ! -f "Jenkinsfile" ]; then
  echo "ERROR: Jenkinsfile not found"
  exit 1
fi
```

### A.2 Verify Java Availability

```bash
which java || { echo "ERROR: java not found"; exit 1; }
java -version 2>&1 | head -1
```

**PRECONDITION_CHECKPOINT:**
- [ ] Jenkinsfile exists
- [ ] Java available

## PHASE 1: RUN HEURISTIC CHECKS

```bash
SCRIPT_APPROVAL_CALLS=$(rg -n "evaluate\\(|GroovyShell|new File\\(" Jenkinsfile 2>/dev/null | wc -l | tr -d ' ')
SHELL_CURL_PIPE=$(rg -n "sh ['\\\"].*curl .+\\|\\s*(sh|bash)" Jenkinsfile 2>/dev/null | wc -l | tr -d ' ')
PLAINTEXT_SECRET=$(rg -n "(password|token|secret)\\s*=\\s*['\\\"][^$]" Jenkinsfile 2>/dev/null | wc -l | tr -d ' ')
GATE_RESULT=$([ "$SCRIPT_APPROVAL_CALLS" -eq 0 ] && [ "$SHELL_CURL_PIPE" -eq 0 ] && [ "$PLAINTEXT_SECRET" -eq 0 ] && echo "pass" || echo "fail")
```

**CHECKPOINT:** Jenkins security heuristics complete.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/jenkins-findings.json" << EOF
{
  "tool": "jenkins-lint",
  "scan_target": "Jenkinsfile",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "summary": {
    "script_approval_calls": $SCRIPT_APPROVAL_CALLS,
    "curl_pipe_shell": $SHELL_CURL_PIPE,
    "plaintext_secret_assignments": $PLAINTEXT_SECRET
  },
  "gate_result": "$GATE_RESULT"
}
EOF
```

**CHECKPOINT:** Jenkins artifact written.

**EMIT:** `<promise>JENKINS_$GATE_RESULT</promise>`
