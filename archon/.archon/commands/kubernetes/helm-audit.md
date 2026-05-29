# Helm Audit

Audit a Helm chart for security best practices, deprecated APIs, resource limits, and configuration quality. Produces a structured audit report.

## PHASE 1: TEMPLATE AND ANALYZE

```bash
which helm || { echo "ERROR: helm not found"; exit 1; }

# Render templates for analysis
helm template "$HELM_RELEASE" "$HELM_CHART" \
  --namespace "$HELM_NAMESPACE" \
  > "$ARTIFACTS_DIR/helm-rendered.yaml" 2>&1

TEMPLATE_EXIT=$?
if [ $TEMPLATE_EXIT -ne 0 ]; then
  echo "ERROR: Failed to render Helm templates"
  exit 1
fi
```

**CHECKPOINT:** Templates rendered.

## PHASE 2: SECURITY CHECKS

Analyze rendered templates for:

1. **Containers running as root** — `securityContext.runAsNonRoot` should be true
2. **Missing resource limits** — every container should have CPU/memory limits
3. **Privileged containers** — `securityContext.privileged` should be false
4. **Host network/PID/IPC** — should not use host namespaces
5. **Missing readiness/liveness probes** — every deployment should have health probes
6. **Deprecated API versions** — check for deprecated `apiVersion` values
7. **Missing network policies** — namespace should have NetworkPolicy
8. **Hardcoded secrets** — check for inline secret values instead of Secret references

```bash
# Check for running as root
ROOT_CONTAINERS=$(grep -c "runAsNonRoot: false\|runAsUser: 0" "$ARTIFACTS_DIR/helm-rendered.yaml" 2>/dev/null || echo "0")

# Check for missing resource limits
NO_LIMITS=$(grep -c "resources: {}" "$ARTIFACTS_DIR/helm-rendered.yaml" 2>/dev/null || echo "0")

# Check for privileged containers
PRIVILEGED=$(grep -c "privileged: true" "$ARTIFACTS_DIR/helm-rendered.yaml" 2>/dev/null || echo "0")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/helm-audit.json" << EOF
{
  "chart": "$HELM_CHART",
  "release": "$HELM_RELEASE",
  "findings": {
    "root_containers": $ROOT_CONTAINERS,
    "missing_resource_limits": $NO_LIMITS,
    "privileged_containers": $PRIVILEGED
  },
  "recommendations": [],
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>HELM_AUDIT_COMPLETE</promise>`
