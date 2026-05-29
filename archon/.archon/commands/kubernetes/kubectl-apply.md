# kubectl Apply

Execute `kubectl apply` with dry-run validation before actual apply. Records deployment artifact with rollback command.

## PRECONDITIONS CHECK

```bash
which kubectl || { echo "ERROR: kubectl not found"; exit 1; }
kubectl cluster-info > /dev/null 2>&1 || { echo "ERROR: Cannot connect to cluster"; exit 1; }
```

## PHASE 1: DRY-RUN VALIDATION

```bash
# Server-side dry-run for accurate validation
kubectl apply \
  --dry-run=server \
  -f "$ARGUMENTS" \
  -n "$K8S_NAMESPACE" \
  -o yaml \
  2>&1 | tee "$ARTIFACTS_DIR/kubectl-dry-run.yaml"

DRY_RUN_EXIT=$?
if [ $DRY_RUN_EXIT -ne 0 ]; then
  echo "ERROR: Dry-run validation failed"
  exit 1
fi
```

**CHECKPOINT:** Dry-run passed.

## PHASE 2: APPLY

```bash
START_TIME=$(date +%s)
DEPLOYMENT_ID="kubectl-$(date +%Y%m%d%H%M%S)-$(head -c 4 /dev/urandom | xxd -p)"

kubectl apply \
  -f "$ARGUMENTS" \
  -n "$K8S_NAMESPACE" \
  2>&1 | tee "$ARTIFACTS_DIR/kubectl-apply.log"

APPLY_EXIT=$?
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
OUTCOME=$([ $APPLY_EXIT -eq 0 ] && echo "success" || echo "failed")

echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"action\":\"kubectl-apply\",\"outcome\":\"$OUTCOME\",\"resource\":\"$ARGUMENTS\",\"namespace\":\"$K8S_NAMESPACE\"}" >> "$ARTIFACTS_DIR/audit-log.jsonl"
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/deploys/$DEPLOYMENT_ID.json" << EOF
{
  "deployment_id": "$DEPLOYMENT_ID",
  "environment": "$ENVIRONMENT",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "tool": "kubectl",
  "resource_identifier": "$ARGUMENTS",
  "outcome": "$OUTCOME",
  "duration_seconds": $DURATION,
  "rollback_command": "kubectl delete -f $ARGUMENTS -n $K8S_NAMESPACE",
  "namespace": "$K8S_NAMESPACE",
  "deployed_by": "archon/$WORKFLOW_ID"
}
EOF
```

**EMIT:** `<promise>KUBECTL_$OUTCOME</promise>`
