# Kubernetes Health Check

Comprehensive health check of pods, deployments, services, and endpoints in a namespace. Used as post-deploy verification and in health-poll loops.

## PHASE 1: POD STATUS CHECK

```bash
which kubectl || { echo "ERROR: kubectl not found"; exit 1; }

# Get pod status
kubectl get pods -n "$K8S_NAMESPACE" -o wide 2>&1 | tee "$ARTIFACTS_DIR/k8s-pods.txt"

TOTAL_PODS=$(kubectl get pods -n "$K8S_NAMESPACE" --no-headers 2>/dev/null | wc -l)
RUNNING_PODS=$(kubectl get pods -n "$K8S_NAMESPACE" --no-headers 2>/dev/null | grep -c "Running")
FAILED_PODS=$(kubectl get pods -n "$K8S_NAMESPACE" --no-headers 2>/dev/null | grep -cE "CrashLoopBackOff|Error|ImagePullBackOff|OOMKilled")
PENDING_PODS=$(kubectl get pods -n "$K8S_NAMESPACE" --no-headers 2>/dev/null | grep -c "Pending")
```

**CHECKPOINT:** Pod status collected.

## PHASE 2: DEPLOYMENT ROLLOUT STATUS

```bash
# Check deployment rollout status
kubectl get deployments -n "$K8S_NAMESPACE" -o wide 2>&1 | tee "$ARTIFACTS_DIR/k8s-deployments.txt"

DEPLOYMENTS=$(kubectl get deployments -n "$K8S_NAMESPACE" --no-headers 2>/dev/null | wc -l)
READY_DEPLOYMENTS=$(kubectl get deployments -n "$K8S_NAMESPACE" --no-headers 2>/dev/null | awk '{split($2,a,"/"); if(a[1]==a[2]) print}' | wc -l)
```

## PHASE 3: EVENTS CHECK

```bash
# Get recent events — look for warnings
kubectl get events -n "$K8S_NAMESPACE" --sort-by='.lastTimestamp' --field-selector type=Warning 2>/dev/null | tail -20 > "$ARTIFACTS_DIR/k8s-warning-events.txt"
WARNING_EVENTS=$(wc -l < "$ARTIFACTS_DIR/k8s-warning-events.txt" 2>/dev/null || echo "0")
```

## PHASE 4: ENDPOINT HEALTH

```bash
# Check service endpoints
kubectl get endpoints -n "$K8S_NAMESPACE" 2>&1 | tee "$ARTIFACTS_DIR/k8s-endpoints.txt"
EMPTY_ENDPOINTS=$(kubectl get endpoints -n "$K8S_NAMESPACE" --no-headers 2>/dev/null | awk '{if($2=="<none>") print}' | wc -l)
```

## PHASE 5: DETERMINE OVERALL STATUS

```bash
STATUS="healthy"
if [ "$FAILED_PODS" -gt "0" ]; then STATUS="degraded"; fi
if [ "$READY_DEPLOYMENTS" -lt "$DEPLOYMENTS" ]; then STATUS="degraded"; fi
if [ "$TOTAL_PODS" -eq "0" ]; then STATUS="degraded"; fi
if [ "$EMPTY_ENDPOINTS" -gt "0" ]; then STATUS="degraded"; fi
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/health-check.json" << EOF
{
  "status": "$STATUS",
  "namespace": "$K8S_NAMESPACE",
  "pods": {
    "total": $TOTAL_PODS,
    "running": $RUNNING_PODS,
    "failed": $FAILED_PODS,
    "pending": $PENDING_PODS
  },
  "deployments": {
    "total": $DEPLOYMENTS,
    "ready": $READY_DEPLOYMENTS
  },
  "warning_events": $WARNING_EVENTS,
  "empty_endpoints": $EMPTY_ENDPOINTS,
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>$STATUS</promise>`
