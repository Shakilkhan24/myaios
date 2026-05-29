# Terraform Destroy

Plan a destroy operation for Terraform-managed infrastructure. Generates a destroy plan, backs up state, and requires explicit human approval before execution. This is a CRITICAL blast-radius operation.

## PRECONDITIONS CHECK

```bash
which terraform || { echo "ERROR: terraform not found"; exit 1; }

# Mandatory production check
if [ "$ENVIRONMENT" = "production" ]; then
  if [ -z "$CONFIRMATION_TOKEN" ]; then
    echo "ERROR: Production destroy requires CONFIRMATION_TOKEN=approve"
    exit 1
  fi
  echo "⚠️  WARNING: Destroying PRODUCTION infrastructure"
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] terraform binary exists
- [ ] Production confirmation received

## PHASE 1: STATE BACKUP

```bash
cd "$TF_ROOT"
terraform init -input=false -no-color > /dev/null 2>&1
terraform workspace select "$TF_WORKSPACE" 2>/dev/null || true

# Full state backup before destroy
terraform state pull > "$ARTIFACTS_DIR/pre-destroy-state-backup.json"
echo "State backup saved to $ARTIFACTS_DIR/pre-destroy-state-backup.json"
echo "State serial: $(jq '.serial' "$ARTIFACTS_DIR/pre-destroy-state-backup.json")"
```

**CHECKPOINT:** State backed up.

## PHASE 2: PLAN DESTROY

```bash
VAR_FILE_FLAG=""
if [ -n "$TF_VAR_FILE" ] && [ -f "$TF_VAR_FILE" ]; then
  VAR_FILE_FLAG="-var-file=$TF_VAR_FILE"
fi

terraform plan \
  -destroy \
  -input=false \
  -no-color \
  -out="$ARTIFACTS_DIR/tf-destroy.plan" \
  $VAR_FILE_FLAG \
  2>&1 | tee "$ARTIFACTS_DIR/tf-destroy-plan.txt"

# Generate JSON for analysis
terraform show -json "$ARTIFACTS_DIR/tf-destroy.plan" > "$ARTIFACTS_DIR/tf-destroy-plan.json" 2>&1

DESTROY_COUNT=$(jq '[.resource_changes[] | select(.change.actions | index("delete"))] | length' "$ARTIFACTS_DIR/tf-destroy-plan.json" 2>/dev/null || echo "0")
```

**CHECKPOINT:** Destroy plan generated. $DESTROY_COUNT resources will be destroyed.

## PHASE 3: EXECUTE DESTROY (post-approval only)

```bash
START_TIME=$(date +%s)
DEPLOYMENT_ID="tf-destroy-$(date +%Y%m%d%H%M%S)-$(head -c 4 /dev/urandom | xxd -p)"

terraform apply \
  -input=false \
  -no-color \
  "$ARTIFACTS_DIR/tf-destroy.plan" \
  2>&1 | tee "$ARTIFACTS_DIR/tf-destroy.log"

DESTROY_EXIT=$?
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

OUTCOME=$([ $DESTROY_EXIT -eq 0 ] && echo "success" || echo "failed")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/deploys/$DEPLOYMENT_ID.json" << EOF
{
  "deployment_id": "$DEPLOYMENT_ID",
  "environment": "$ENVIRONMENT",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "tool": "terraform",
  "action": "destroy",
  "resources_destroyed": $DESTROY_COUNT,
  "outcome": "$OUTCOME",
  "duration_seconds": $DURATION,
  "rollback_command": "terraform state push $ARTIFACTS_DIR/pre-destroy-state-backup.json && terraform apply -auto-approve",
  "deployed_by": "archon/$WORKFLOW_ID"
}
EOF

echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"action\":\"terraform-destroy\",\"outcome\":\"$OUTCOME\",\"environment\":\"$ENVIRONMENT\",\"resources_destroyed\":$DESTROY_COUNT}" >> "$ARTIFACTS_DIR/audit-log.jsonl"
```

**EMIT:** `<promise>DESTROY_$OUTCOME</promise>`
