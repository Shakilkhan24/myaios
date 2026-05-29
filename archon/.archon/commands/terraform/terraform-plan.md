# Terraform Plan

Execute `terraform init` and `terraform plan` against the configured root module. Produces both a binary plan file and a human-readable plan summary artifact.

## PRECONDITIONS CHECK

### A.1 Verify Tool Availability

```bash
which terraform || { echo "ERROR: terraform not found"; exit 1; }
terraform version | head -1
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

### A.3 Verify Environment Lock

```bash
if [ "$ENVIRONMENT" = "production" ]; then
  echo "WARNING: Planning against PRODUCTION environment"
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] terraform binary exists and correct version
- [ ] Cloud auth valid
- [ ] Environment acknowledged

## PHASE 1: TERRAFORM INIT

```bash
cd "$TF_ROOT"

# Initialize terraform with backend config
terraform init \
  -input=false \
  -no-color \
  2>&1 | tee "$ARTIFACTS_DIR/tf-init.log"

if [ $? -ne 0 ]; then
  echo "ERROR: terraform init failed"
  cat > "$ARTIFACTS_DIR/terraform-plan-result.json" << EOF
{
  "status": "failed",
  "phase": "init",
  "error": "terraform init failed — see tf-init.log",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
  exit 1
fi
```

**CHECKPOINT:** Terraform initialized successfully.

## PHASE 2: SELECT WORKSPACE

```bash
# Select or create workspace
terraform workspace select "$TF_WORKSPACE" 2>/dev/null || \
  terraform workspace new "$TF_WORKSPACE"
echo "Active workspace: $(terraform workspace show)"
```

**CHECKPOINT:** Workspace selected.

## PHASE 3: TERRAFORM PLAN

```bash
# Build var-file flag if configured
VAR_FILE_FLAG=""
if [ -n "$TF_VAR_FILE" ] && [ -f "$TF_VAR_FILE" ]; then
  VAR_FILE_FLAG="-var-file=$TF_VAR_FILE"
fi

# Execute plan
terraform plan \
  -input=false \
  -no-color \
  -out="$TF_PLAN_FILE" \
  $VAR_FILE_FLAG \
  2>&1 | tee "$ARTIFACTS_DIR/tfplan.txt"

PLAN_EXIT=$?

if [ $PLAN_EXIT -ne 0 ]; then
  echo "ERROR: terraform plan failed"
  exit 1
fi

# Generate JSON plan for downstream analysis
terraform show -json "$TF_PLAN_FILE" > "$ARTIFACTS_DIR/tfplan.json" 2>&1
```

**CHECKPOINT:** Plan generated successfully.

## PHASE 4: PLAN SUMMARY

```bash
# Extract plan summary
RESOURCE_COUNT=$(jq '.resource_changes | length' "$ARTIFACTS_DIR/tfplan.json" 2>/dev/null || echo "0")
CREATE_COUNT=$(jq '[.resource_changes[] | select(.change.actions | index("create"))] | length' "$ARTIFACTS_DIR/tfplan.json" 2>/dev/null || echo "0")
UPDATE_COUNT=$(jq '[.resource_changes[] | select(.change.actions | index("update"))] | length' "$ARTIFACTS_DIR/tfplan.json" 2>/dev/null || echo "0")
DELETE_COUNT=$(jq '[.resource_changes[] | select(.change.actions | index("delete"))] | length' "$ARTIFACTS_DIR/tfplan.json" 2>/dev/null || echo "0")
REPLACE_COUNT=$(jq '[.resource_changes[] | select(.change.actions | index("delete") and index("create"))] | length' "$ARTIFACTS_DIR/tfplan.json" 2>/dev/null || echo "0")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/terraform-plan-result.json" << EOF
{
  "status": "success",
  "workspace": "$TF_WORKSPACE",
  "environment": "$ENVIRONMENT",
  "plan_file": "$TF_PLAN_FILE",
  "summary": {
    "total_changes": $RESOURCE_COUNT,
    "to_create": $CREATE_COUNT,
    "to_update": $UPDATE_COUNT,
    "to_delete": $DELETE_COUNT,
    "to_replace": $REPLACE_COUNT
  },
  "has_destructive_changes": $([ "$DELETE_COUNT" -gt "0" ] && echo "true" || echo "false"),
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Plan artifacts written. Ready for security scan and impact classification.

**EMIT:** `<promise>PLAN_COMPLETE</promise>`
