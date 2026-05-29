# ArgoCD Sync

Trigger an ArgoCD application sync, wait for health, and record the sync result. Uses ArgoCD CLI or API.

## PRECONDITIONS CHECK

```bash
if which argocd > /dev/null 2>&1; then
  ARGOCD_CLI="true"
  argocd version --client 2>/dev/null | head -1
elif [ -n "$ARGOCD_URL" ]; then
  ARGOCD_CLI="false"
  echo "Using ArgoCD API at $ARGOCD_URL"
else
  echo "ERROR: Neither argocd CLI nor ARGOCD_URL configured"
  exit 1
fi
```

## PHASE 1: CHECK CURRENT STATE

```bash
APP_NAME="${ARGUMENTS:-$HELM_RELEASE}"

if [ "$ARGOCD_CLI" = "true" ]; then
  argocd app get "$APP_NAME" -o json > "$ARTIFACTS_DIR/argocd-pre-sync.json" 2>&1
  CURRENT_SYNC=$(jq -r '.status.sync.status' "$ARTIFACTS_DIR/argocd-pre-sync.json" 2>/dev/null)
  CURRENT_HEALTH=$(jq -r '.status.health.status' "$ARTIFACTS_DIR/argocd-pre-sync.json" 2>/dev/null)
  echo "Current state: sync=$CURRENT_SYNC health=$CURRENT_HEALTH"
fi
```

**CHECKPOINT:** Current app state captured.

## PHASE 2: TRIGGER SYNC

```bash
START_TIME=$(date +%s)

if [ "$ARGOCD_CLI" = "true" ]; then
  argocd app sync "$APP_NAME" \
    --prune \
    --timeout 300 \
    2>&1 | tee "$ARTIFACTS_DIR/argocd-sync.log"
  SYNC_EXIT=$?
fi

# Wait for health
if [ "$ARGOCD_CLI" = "true" ]; then
  argocd app wait "$APP_NAME" \
    --health \
    --timeout 300 \
    2>&1 | tee -a "$ARTIFACTS_DIR/argocd-sync.log"
  WAIT_EXIT=$?
fi

END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
```

## ARTIFACT OUTPUT

```bash
if [ "$ARGOCD_CLI" = "true" ]; then
  argocd app get "$APP_NAME" -o json > "$ARTIFACTS_DIR/argocd-post-sync.json" 2>&1
  SYNC_STATUS=$(jq -r '.status.sync.status' "$ARTIFACTS_DIR/argocd-post-sync.json" 2>/dev/null)
  HEALTH_STATUS=$(jq -r '.status.health.status' "$ARTIFACTS_DIR/argocd-post-sync.json" 2>/dev/null)
fi

cat > "$ARTIFACTS_DIR/gitops-sync.json" << EOF
{
  "app_name": "$APP_NAME",
  "sync_status": "${SYNC_STATUS:-unknown}",
  "health_status": "${HEALTH_STATUS:-unknown}",
  "duration_seconds": $DURATION,
  "environment": "$ENVIRONMENT",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>ARGOCD_${HEALTH_STATUS:-UNKNOWN}</promise>`
