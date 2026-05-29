# Terraform Drift Analyze

Detect and analyze infrastructure drift between actual cloud state and Terraform code. Classifies drift as intentional or accidental and outputs a structured analysis.

## PRECONDITIONS CHECK

```bash
which terraform || { echo "ERROR: terraform not found"; exit 1; }
if [ "$CLOUD_PROVIDER" = "aws" ]; then
  aws sts get-caller-identity --output json || { echo "ERROR: AWS auth failed"; exit 1; }
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] terraform exists
- [ ] Cloud auth valid

## PHASE 1: REFRESH STATE

```bash
cd "$TF_ROOT"
terraform init -input=false -no-color > /dev/null 2>&1
terraform workspace select "$TF_WORKSPACE" 2>/dev/null || true

# Refresh state to detect drift
terraform refresh -input=false -no-color 2>&1 | tee "$ARTIFACTS_DIR/tf-refresh.log"
```

**CHECKPOINT:** State refreshed.

## PHASE 2: DETECT DRIFT

```bash
# Plan to detect differences
terraform plan -detailed-exitcode -input=false -no-color -out="$ARTIFACTS_DIR/tf-drift.plan" 2>&1 | tee "$ARTIFACTS_DIR/tf-drift-plan.txt"
PLAN_EXIT=$?

# Exit codes: 0 = no drift, 2 = drift detected
if [ $PLAN_EXIT -eq 0 ]; then
  DRIFT_DETECTED="false"
  DRIFT_COUNT=0
  echo "No drift detected"
elif [ $PLAN_EXIT -eq 2 ]; then
  DRIFT_DETECTED="true"
  terraform show -json "$ARTIFACTS_DIR/tf-drift.plan" > "$ARTIFACTS_DIR/tf-drift.json" 2>&1
  DRIFT_COUNT=$(jq '.resource_changes | length' "$ARTIFACTS_DIR/tf-drift.json" 2>/dev/null || echo "0")
  echo "Drift detected: $DRIFT_COUNT resources differ"
else
  echo "ERROR: terraform plan failed during drift detection"
  exit 1
fi
```

**CHECKPOINT:** Drift detection complete.

## PHASE 3: CLASSIFY DRIFT

For each drifted resource, classify as:

1. **Intentional** — manual change made outside Terraform (e.g., emergency hotfix, console change)
2. **Accidental** — unexpected change from auto-scaling, provider defaults, or external system
3. **Security-relevant** — drift in security groups, IAM roles, encryption settings

Analyze the drift plan JSON and identify:
- Which resources drifted
- What attributes changed
- Whether changes affect security posture
- Whether changes should be imported into code or reverted

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/drift-analysis.json" << EOF
{
  "drift_detected": $DRIFT_DETECTED,
  "drift_count": $DRIFT_COUNT,
  "environment": "$ENVIRONMENT",
  "workspace": "$TF_WORKSPACE",
  "classification": {
    "intentional": [],
    "accidental": [],
    "security_relevant": []
  },
  "recommendation": "$([ "$DRIFT_DETECTED" = "true" ] && echo "review_and_reconcile" || echo "no_action")",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>DRIFT_ANALYZED</promise>`
