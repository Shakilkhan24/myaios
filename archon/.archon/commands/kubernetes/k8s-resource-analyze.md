# Kubernetes Resource Analyze

Analyze Kubernetes resource requests, limits, HPA settings, and actual utilization. Identifies over-provisioned and under-provisioned workloads for right-sizing.

## PHASE 1: COLLECT RESOURCE DATA

```bash
which kubectl || { echo "ERROR: kubectl not found"; exit 1; }

# Get resource requests/limits for all pods
kubectl get pods -n "$K8S_NAMESPACE" -o json > "$ARTIFACTS_DIR/k8s-pods-full.json" 2>&1

# Get HPA configurations
kubectl get hpa -n "$K8S_NAMESPACE" -o json > "$ARTIFACTS_DIR/k8s-hpa.json" 2>&1

# Get VPA recommendations if available
kubectl get vpa -n "$K8S_NAMESPACE" -o json > "$ARTIFACTS_DIR/k8s-vpa.json" 2>/dev/null || echo '{"items":[]}' > "$ARTIFACTS_DIR/k8s-vpa.json"

# Get node resource usage
kubectl top pods -n "$K8S_NAMESPACE" --no-headers 2>/dev/null > "$ARTIFACTS_DIR/k8s-pod-metrics.txt" || true
kubectl top nodes --no-headers 2>/dev/null > "$ARTIFACTS_DIR/k8s-node-metrics.txt" || true
```

**CHECKPOINT:** Resource data collected.

## PHASE 2: ANALYZE UTILIZATION

Analyze for:

1. **No limits set** — containers without CPU/memory limits
2. **Over-provisioned** — containers using <20% of requested resources
3. **Under-provisioned** — containers consistently hitting limits (OOMKilled, CPU throttled)
4. **HPA misconfig** — HPA targets that never scale or are always at max
5. **PDB coverage** — deployments without PodDisruptionBudget

```bash
# Count pods without resource limits
NO_LIMITS=$(jq '[.items[].spec.containers[] | select(.resources.limits == null or .resources.limits == {})] | length' "$ARTIFACTS_DIR/k8s-pods-full.json" 2>/dev/null || echo "0")

# Count HPA objects
HPA_COUNT=$(jq '.items | length' "$ARTIFACTS_DIR/k8s-hpa.json" 2>/dev/null || echo "0")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/k8s-resource-analysis.json" << EOF
{
  "namespace": "$K8S_NAMESPACE",
  "pods_without_limits": $NO_LIMITS,
  "hpa_count": $HPA_COUNT,
  "recommendations": [],
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>RESOURCE_ANALYSIS_COMPLETE</promise>`
