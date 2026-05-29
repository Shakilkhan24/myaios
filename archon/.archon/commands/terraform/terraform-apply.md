# Terraform Apply

Apply a previously generated Terraform plan. Reads the binary plan file from `$TF_PLAN_FILE`, applies it, and records the deployment artifact. This command must only run after approval.

## PRECONDITIONS CHECK

### A.1 Verify Plan File Exists

```bash
if [ ! -f "$TF_PLAN_FILE" ]; then
  echo "ERROR: Plan file not found at $TF_PLAN_FILE"
  echo "Run terraform-plan first to generate a plan."
  exit 1
fi
```

### A.2 Verify Authentication

```bash
if [ "$CLOUD_PROVIDER" = "aws" ]; then
  aws sts get-caller-identity --output json || { echo "ERROR: AWS auth failed"; exit 1; }
elif [ "$CLOUD_PROVIDER" = "gcp" ]; then
  gcloud config get-value account || { echo "ERROR: GCP auth failed"; exit 1; }
elif [ "$CLOUD_PROVIDER" = "azure" ]; then
  az account show --output json || { echo "ERROR: Azure auth failed"; exit 1; }
fi
```

### A.3 Production Confirmation

```bash
if [ "$ENVIRONMENT" = "production" ]; then
  if [ -z "$CONFIRMATION_TOKEN" ]; then
    echo "ERROR: Production apply requires CONFIRMATION_TOKEN"
    exit 1
  fi
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Plan file exists
- [ ] Cloud auth valid
- [ ] Production confirmation received (if applicable)

## PHASE 1: RECORD PRE-APPLY STATE

```bash
cd "$TF_ROOT"

# Generate a deployment ID
DEPLOYMENT_ID="tf-$(date +%Y%m%d%H%M%S)-$(head -c 4 /dev/urandom | xxd -p)"
START_TIME=$(date +%s)

# Record current state version for rollback reference
terraform state pull > "$ARTIFACTS_DIR/pre-apply-state.json" 2>/dev/null || true
PRE_APPLY_SERIAL=$(jq '.serial // 0' "$ARTIFACTS_DIR/pre-apply-state.json" 2>/dev/null || echo "0")

# Audit log entry
echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"action\":\"terraform-apply-start\",\"environment\":\"$ENVIRONMENT\",\"operator\":\"archon/$WORKFLOW_ID\",\"deployment_id\":\"$DEPLOYMENT_ID\"}" >> "$ARTIFACTS_DIR/audit-log.jsonl"
```

**CHECKPOINT:** Pre-apply state recorded.

## PHASE 2: APPLY PLAN

```bash
# Apply the saved plan
terraform apply \
  -input=false \
  -no-color \
  "$TF_PLAN_FILE" \
  2>&1 | tee "$ARTIFACTS_DIR/tf-apply.log"

APPLY_EXIT=$?
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

if [ $APPLY_EXIT -eq 0 ]; then
  OUTCOME="success"
  echo "Terraform apply succeeded in ${DURATION}s"
else
  OUTCOME="failed"
  echo "ERROR: Terraform apply failed after ${DURATION}s"
fi
```

**CHECKPOINT:** Apply executed.

## PHASE 3: RECORD DEPLOYMENT ARTIFACT

```bash
# Get post-apply state serial for rollback
POST_APPLY_SERIAL=$(terraform state pull | jq '.serial // 0' 2>/dev/null || echo "0")

cat > "$ARTIFACTS_DIR/deploys/$DEPLOYMENT_ID.json" << EOF
{
  "deployment_id": "$DEPLOYMENT_ID",
  "workflow_id": "$WORKFLOW_ID",
  "environment": "$ENVIRONMENT",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "tool": "terraform",
  "resource_identifier": "$TF_ROOT",
  "version": "state-serial-$POST_APPLY_SERIAL",
  "outcome": "$OUTCOME",
  "duration_seconds": $DURATION,
  "change_summary": "Applied plan from $TF_PLAN_FILE",
  "rollback_command": "terraform state push $ARTIFACTS_DIR/pre-apply-state.json",
  "rollback_version": "state-serial-$PRE_APPLY_SERIAL",
  "deployed_by": "archon/$WORKFLOW_ID"
}
EOF

# Audit log entry
echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"action\":\"terraform-apply-complete\",\"outcome\":\"$OUTCOME\",\"environment\":\"$ENVIRONMENT\",\"operator\":\"archon/$WORKFLOW_ID\",\"deployment_id\":\"$DEPLOYMENT_ID\",\"duration\":$DURATION}" >> "$ARTIFACTS_DIR/audit-log.jsonl"
```

**CHECKPOINT:** Deployment artifact recorded.

**EMIT:** `<promise>APPLY_$OUTCOME</promise>`
