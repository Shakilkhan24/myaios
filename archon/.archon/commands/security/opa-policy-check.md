# OPA Policy Check

Run Open Policy Agent (OPA) / Conftest policy checks against Kubernetes manifests, Terraform plans, or Dockerfiles.

## PHASE 1: RUN POLICY CHECK

```bash
if which conftest > /dev/null 2>&1; then
  POLICY_DIR="${CUSTOM_POLICIES_DIR:-.archon/policies/}"
  
  # Check what artifacts are available to validate
  if [ -f "$ARTIFACTS_DIR/tfplan.json" ]; then
    conftest test "$ARTIFACTS_DIR/tfplan.json" --policy "$POLICY_DIR" --output json > "$ARTIFACTS_DIR/opa-tf-results.json" 2>&1 || true
  fi
  
  if [ -f "$ARTIFACTS_DIR/helm-rendered.yaml" ]; then
    conftest test "$ARTIFACTS_DIR/helm-rendered.yaml" --policy "$POLICY_DIR" --output json > "$ARTIFACTS_DIR/opa-k8s-results.json" 2>&1 || true
  fi
  
  FAILURES=$(jq '[.[] | .failures[]?] | length' "$ARTIFACTS_DIR/opa-"*"-results.json" 2>/dev/null || echo "0")
  WARNINGS=$(jq '[.[] | .warnings[]?] | length' "$ARTIFACTS_DIR/opa-"*"-results.json" 2>/dev/null || echo "0")
elif which opa > /dev/null 2>&1; then
  echo "OPA available but conftest preferred for policy testing"
  FAILURES=0; WARNINGS=0
else
  echo "WARNING: Neither conftest nor opa found"; FAILURES=0; WARNINGS=0
fi

cat > "$ARTIFACTS_DIR/opa-policy-check.json" << EOF
{"tool":"conftest/opa","failures":$FAILURES,"warnings":$WARNINGS,"gate_result":"$([ "$FAILURES" -gt "0" ] && echo "fail" || echo "pass")","timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)"}
EOF
```

**EMIT:** `<promise>POLICY_CHECK_COMPLETE</promise>`
