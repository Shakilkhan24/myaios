# Helm Upgrade

Execute `helm upgrade --install` for a release. Records the deployment artifact with revision info and rollback command. Must run after approval for production.

## PRECONDITIONS CHECK

```bash
which helm || { echo "ERROR: helm not found"; exit 1; }
kubectl cluster-info > /dev/null 2>&1 || { echo "ERROR: Cannot connect to cluster"; exit 1; }

if [ "$ENVIRONMENT" = "production" ]; then
  if [ -z "$CONFIRMATION_TOKEN" ]; then
    echo "ERROR: Production Helm upgrade requires CONFIRMATION_TOKEN"
    exit 1
  fi
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] helm exists
- [ ] Cluster reachable
- [ ] Production confirmed (if applicable)

## PHASE 1: PRE-UPGRADE STATE

```bash
DEPLOYMENT_ID="helm-$(date +%Y%m%d%H%M%S)-$(head -c 4 /dev/urandom | xxd -p)"
START_TIME=$(date +%s)

# Capture current revision for rollback
CURRENT_REVISION=$(helm list -n "$HELM_NAMESPACE" --filter "^${HELM_RELEASE}$" -o json 2>/dev/null | jq -r '.[0].revision // "0"')
echo "Current revision: $CURRENT_REVISION"

# Audit log
echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"action\":\"helm-upgrade-start\",\"release\":\"$HELM_RELEASE\",\"namespace\":\"$HELM_NAMESPACE\",\"environment\":\"$ENVIRONMENT\",\"deployment_id\":\"$DEPLOYMENT_ID\"}" >> "$ARTIFACTS_DIR/audit-log.jsonl"
```

**CHECKPOINT:** Pre-upgrade state captured.

## PHASE 2: EXECUTE UPGRADE

```bash
# Build values flags
VALUES_FLAGS=""
if [ -f "values.yaml" ]; then VALUES_FLAGS="$VALUES_FLAGS -f values.yaml"; fi
if [ -f "values.$ENVIRONMENT.yaml" ]; then VALUES_FLAGS="$VALUES_FLAGS -f values.$ENVIRONMENT.yaml"; fi

helm upgrade --install "$HELM_RELEASE" "$HELM_CHART" \
  --namespace "$HELM_NAMESPACE" \
  --create-namespace \
  --wait \
  --timeout 300s \
  $VALUES_FLAGS \
  2>&1 | tee "$ARTIFACTS_DIR/helm-upgrade.log"

UPGRADE_EXIT=$?
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
NEW_REVISION=$(helm list -n "$HELM_NAMESPACE" --filter "^${HELM_RELEASE}$" -o json 2>/dev/null | jq -r '.[0].revision // "0"')
OUTCOME=$([ $UPGRADE_EXIT -eq 0 ] && echo "success" || echo "failed")
```

**CHECKPOINT:** Helm upgrade executed.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/deploys/$DEPLOYMENT_ID.json" << EOF
{
  "deployment_id": "$DEPLOYMENT_ID",
  "workflow_id": "$WORKFLOW_ID",
  "environment": "$ENVIRONMENT",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "tool": "helm",
  "resource_identifier": "$HELM_RELEASE",
  "version": "revision-$NEW_REVISION",
  "outcome": "$OUTCOME",
  "duration_seconds": $DURATION,
  "change_summary": "Helm upgrade $HELM_RELEASE to revision $NEW_REVISION",
  "rollback_command": "helm rollback $HELM_RELEASE $CURRENT_REVISION -n $HELM_NAMESPACE",
  "rollback_version": "revision-$CURRENT_REVISION",
  "deployed_by": "archon/$WORKFLOW_ID",
  "namespace": "$HELM_NAMESPACE",
  "chart": "$HELM_CHART"
}
EOF

echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"action\":\"helm-upgrade-complete\",\"outcome\":\"$OUTCOME\",\"release\":\"$HELM_RELEASE\",\"revision\":\"$NEW_REVISION\",\"duration\":$DURATION}" >> "$ARTIFACTS_DIR/audit-log.jsonl"
```

**EMIT:** `<promise>HELM_$OUTCOME</promise>`
