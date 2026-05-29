# Change Impact Classification

Analyzes the blast radius and approval requirements for a proposed infrastructure change. Classifies impact as low, medium, high, or critical and determines whether human approval is required.

## INPUT

This command reads the following artifacts to classify impact:

- `$ARTIFACTS_DIR/tfplan.json` or `$ARTIFACTS_DIR/helm-diff.txt` (if available)
- `$ARTIFACTS_DIR/cost-estimate.json` (if available)
- `$ARTIFACTS_DIR/security-scan.json` (if available)

## PHASE 1: GATHER CHANGE METADATA

```bash
# Analyze Terraform plan for resource changes
if [ -f "$ARTIFACTS_DIR/tfplan.json" ]; then
  RESOURCE_COUNT=$(jq '.resource_changes | length' "$ARTIFACTS_DIR/tfplan.json" 2>/dev/null || echo "0")
  DESTROY_COUNT=$(jq '[.resource_changes[] | select(.change.actions | index("delete"))] | length' "$ARTIFACTS_DIR/tfplan.json" 2>/dev/null || echo "0")
  CREATE_COUNT=$(jq '[.resource_changes[] | select(.change.actions | index("create"))] | length' "$ARTIFACTS_DIR/tfplan.json" 2>/dev/null || echo "0")
fi

# Analyze Helm diff for application changes
if [ -f "$ARTIFACTS_DIR/helm-diff.txt" ]; then
  DIFF_LINES=$(wc -l < "$ARTIFACTS_DIR/helm-diff.txt")
fi

# Get cost delta if available
if [ -f "$ARTIFACTS_DIR/cost-estimate.json" ]; then
  COST_DELTA=$(jq '.delta' "$ARTIFACTS_DIR/cost-estimate.json" 2>/dev/null || echo "0")
fi
```

**CHECKPOINT:** Change metadata gathered from available artifacts.

## PHASE 2: CLASSIFY IMPACT

Use the following decision matrix:

**Critical Impact** (requires 2 approvals, CAB review):
- Any `delete` action in production
- Database resource modifications
- Network security group changes
- IAM policy modifications
- Cost delta exceeds $10,000/month

**High Impact** (requires 1 approval):
- Create/modify in production
- Scaling operations
- New resource creation > $1,000/month
- Any change affecting more than 3 services

**Medium Impact** (notification required):
- All staging changes
- Development environment full replacement
- Cost delta $100-$1,000/month

**Low Impact** (automatic pass):
- Development environment additions
- Cost delta < $100/month
- Read-only operations

**CHECKPOINT:** Impact classification determined.

## PHASE 3: DETERMINE APPROVAL REQUIREMENTS

```bash
# Classification logic
if [ "$DESTROY_COUNT" -gt "0" ] && [ "$ENVIRONMENT" = "production" ]; then
  BLAST_RADIUS="critical"
  APPROVAL_REQUIRED="true"
  APPROVAL_COUNT="2"
  REQUIRES_CAB="true"
elif [ "$ENVIRONMENT" = "production" ]; then
  BLAST_RADIUS="high"
  APPROVAL_REQUIRED="true"
  APPROVAL_COUNT="1"
  REQUIRES_CAB="false"
elif [ "$ENVIRONMENT" = "staging" ]; then
  BLAST_RADIUS="medium"
  APPROVAL_REQUIRED="false"
  APPROVAL_COUNT="0"
  REQUIRES_CAB="false"
else
  BLAST_RADIUS="low"
  APPROVAL_REQUIRED="false"
  APPROVAL_COUNT="0"
  REQUIRES_CAB="false"
fi
```

## ARTIFACT OUTPUT

Write classification result:

```bash
cat > "$ARTIFACTS_DIR/impact-classification.json" << EOF
{
  "blast_radius": "$BLAST_RADIUS",
  "requires_approval": $APPROVAL_REQUIRED,
  "approval_count": $APPROVAL_COUNT,
  "requires_cab": $REQUIRES_CAB,
  "affected_services": [],
  "cost_delta": "${COST_DELTA:-0}",
  "resource_changes": {
    "created": ${CREATE_COUNT:-0},
    "destroyed": ${DESTROY_COUNT:-0},
    "total": ${RESOURCE_COUNT:-0}
  },
  "environment": "'$ENVIRONMENT'",
  "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
}
EOF
```

**CHECKPOINT:** Classification complete. Workflow router will determine approval gates based on this output.

**EMIT:** `<promise>IMPACT_CLASSIFIED</promise>`
