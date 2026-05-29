# Helm Rollback

Roll back a Helm release to a previous revision. Identifies the target revision, executes rollback, and verifies health. Follows Standard D (rollback commands).

## PHASE 1: IDENTIFY ROLLBACK TARGET

```bash
which helm || { echo "ERROR: helm not found"; exit 1; }

# List revision history
helm history "$HELM_RELEASE" -n "$HELM_NAMESPACE" --max 10 2>&1 | tee "$ARTIFACTS_DIR/helm-history.txt"

CURRENT_REVISION=$(helm list -n "$HELM_NAMESPACE" --filter "^${HELM_RELEASE}$" -o json | jq -r '.[0].revision // "0"')
TARGET_REVISION=$((CURRENT_REVISION - 1))

if [ "$TARGET_REVISION" -lt "1" ]; then
  echo "ERROR: No previous revision to roll back to"
  exit 1
fi

echo "Rolling back from revision $CURRENT_REVISION to $TARGET_REVISION"
```

**CHECKPOINT:** Rollback target identified.

## PHASE 2: RECORD PRE-ROLLBACK STATE

```bash
# Capture current state before rolling back
kubectl get pods -n "$HELM_NAMESPACE" -l "app.kubernetes.io/instance=$HELM_RELEASE" -o wide > "$ARTIFACTS_DIR/pre-rollback-pods.txt"
kubectl get events -n "$HELM_NAMESPACE" --sort-by='.lastTimestamp' 2>/dev/null | tail -20 > "$ARTIFACTS_DIR/pre-rollback-events.txt"
```

**CHECKPOINT:** Pre-rollback state captured.

## PHASE 3: EXECUTE ROLLBACK

```bash
START_TIME=$(date +%s)
DEPLOYMENT_ID="helm-rollback-$(date +%Y%m%d%H%M%S)"

helm rollback "$HELM_RELEASE" "$TARGET_REVISION" \
  --namespace "$HELM_NAMESPACE" \
  --wait \
  --timeout 300s \
  2>&1 | tee "$ARTIFACTS_DIR/helm-rollback.log"

ROLLBACK_EXIT=$?
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
OUTCOME=$([ $ROLLBACK_EXIT -eq 0 ] && echo "success" || echo "failed")
```

**CHECKPOINT:** Rollback executed.

## PHASE 4: VERIFY HEALTH POST-ROLLBACK

```bash
# Same health check as deploy
kubectl get pods -n "$HELM_NAMESPACE" -l "app.kubernetes.io/instance=$HELM_RELEASE" -o wide > "$ARTIFACTS_DIR/post-rollback-pods.txt"

READY_PODS=$(kubectl get pods -n "$HELM_NAMESPACE" -l "app.kubernetes.io/instance=$HELM_RELEASE" --no-headers 2>/dev/null | grep -c "Running")
TOTAL_PODS=$(kubectl get pods -n "$HELM_NAMESPACE" -l "app.kubernetes.io/instance=$HELM_RELEASE" --no-headers 2>/dev/null | wc -l)

HEALTH_STATUS="healthy"
if [ "$READY_PODS" -lt "$TOTAL_PODS" ]; then
  HEALTH_STATUS="degraded"
fi

if [ "$OUTCOME" = "failed" ]; then
  # Rollback also failed — escalate immediately
  echo "CRITICAL: Rollback failed. Immediate human intervention required."
fi
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/deploys/$DEPLOYMENT_ID.json" << EOF
{
  "deployment_id": "$DEPLOYMENT_ID",
  "environment": "$ENVIRONMENT",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "tool": "helm",
  "action": "rollback",
  "resource_identifier": "$HELM_RELEASE",
  "from_revision": "$CURRENT_REVISION",
  "to_revision": "$TARGET_REVISION",
  "outcome": "$OUTCOME",
  "health_status": "$HEALTH_STATUS",
  "duration_seconds": $DURATION,
  "deployed_by": "archon/$WORKFLOW_ID"
}
EOF
```

**EMIT:** `<promise>ROLLBACK_$OUTCOME</promise>`
