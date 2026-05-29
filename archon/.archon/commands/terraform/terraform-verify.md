# Terraform Verify

Post-apply verification of Terraform-managed infrastructure. Confirms that resources created/modified by the most recent apply are in the expected state. Runs state checks and optional health probes.

## PHASE 1: READ DEPLOY ARTIFACT

```bash
# Find the most recent deploy artifact
DEPLOY_ARTIFACT=$(ls -t "$ARTIFACTS_DIR/deploys/"tf-*.json 2>/dev/null | head -1)
if [ -z "$DEPLOY_ARTIFACT" ]; then
  echo "WARNING: No Terraform deploy artifact found"
fi

DEPLOY_OUTCOME=$(jq -r '.outcome // "unknown"' "$DEPLOY_ARTIFACT" 2>/dev/null || echo "unknown")
if [ "$DEPLOY_OUTCOME" = "failed" ]; then
  echo "ERROR: Previous apply failed — skipping verification"
  cat > "$ARTIFACTS_DIR/terraform-verify.json" << EOF
{"status": "skipped", "reason": "apply_failed", "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"}
EOF
  exit 1
fi
```

**CHECKPOINT:** Deploy artifact read.

## PHASE 2: STATE VERIFICATION

```bash
cd "$TF_ROOT"

# Verify terraform state is consistent
terraform plan -detailed-exitcode -input=false -no-color 2>&1 | tee "$ARTIFACTS_DIR/tf-verify-plan.txt"
VERIFY_EXIT=$?

# Exit codes: 0 = no changes (clean), 1 = error, 2 = changes detected (drift)
case $VERIFY_EXIT in
  0)
    STATE_STATUS="clean"
    echo "State verification: CLEAN — no drift detected"
    ;;
  2)
    STATE_STATUS="drift_detected"
    echo "WARNING: Drift detected after apply — investigate immediately"
    ;;
  *)
    STATE_STATUS="error"
    echo "ERROR: State verification failed"
    ;;
esac
```

**CHECKPOINT:** State verified.

## PHASE 3: RESOURCE HEALTH CHECK

```bash
# Check key resource outputs
terraform output -json > "$ARTIFACTS_DIR/tf-outputs.json" 2>/dev/null || echo "{}" > "$ARTIFACTS_DIR/tf-outputs.json"

# If outputs include URLs, check connectivity
HEALTH_URL=$(jq -r '.health_url.value // empty' "$ARTIFACTS_DIR/tf-outputs.json" 2>/dev/null)
if [ -n "$HEALTH_URL" ]; then
  HTTP_STATUS=$(curl -sf -o /dev/null -w "%{http_code}" "$HEALTH_URL" 2>/dev/null || echo "000")
  if [ "$HTTP_STATUS" = "200" ]; then
    HEALTH_STATUS="healthy"
  else
    HEALTH_STATUS="degraded"
  fi
else
  HEALTH_STATUS="no_endpoint"
fi
```

## ARTIFACT OUTPUT

```bash
OVERALL_STATUS="healthy"
if [ "$STATE_STATUS" != "clean" ] || [ "$HEALTH_STATUS" = "degraded" ]; then
  OVERALL_STATUS="degraded"
fi

cat > "$ARTIFACTS_DIR/terraform-verify.json" << EOF
{
  "status": "$OVERALL_STATUS",
  "state_check": "$STATE_STATUS",
  "health_check": "$HEALTH_STATUS",
  "drift_detected": $([ "$STATE_STATUS" = "drift_detected" ] && echo "true" || echo "false"),
  "environment": "$ENVIRONMENT",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Verification complete.

**EMIT:** `<promise>$OVERALL_STATUS</promise>`
