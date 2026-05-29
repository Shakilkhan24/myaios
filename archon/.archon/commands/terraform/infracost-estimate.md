# Infracost Estimate

Run Infracost against a Terraform plan to estimate the monthly cost delta. Produces a structured cost artifact for the approval gate.

## PRECONDITIONS CHECK

```bash
which infracost > /dev/null 2>&1 || { echo "WARNING: infracost not found — cost estimation will be skipped"; }
```

## PHASE 1: RUN COST ESTIMATION

```bash
cd "$TF_ROOT"

# Check if infracost is available
if which infracost > /dev/null 2>&1; then
  INFRACOST_VERSION=$(infracost --version 2>/dev/null | head -1)
  echo "Using infracost $INFRACOST_VERSION"

  # Run infracost against the plan
  if [ -f "$ARTIFACTS_DIR/tfplan.json" ]; then
    infracost diff \
      --path "$ARTIFACTS_DIR/tfplan.json" \
      --format json \
      --no-color \
      > "$ARTIFACTS_DIR/infracost-raw.json" 2>&1
    COST_EXIT=$?
  else
    infracost breakdown \
      --path . \
      --format json \
      --no-color \
      > "$ARTIFACTS_DIR/infracost-raw.json" 2>&1
    COST_EXIT=$?
  fi

  if [ $COST_EXIT -ne 0 ]; then
    echo "WARNING: Infracost analysis failed — writing zero-cost estimate"
    COST_FAILED="true"
  fi
else
  COST_FAILED="true"
fi
```

**CHECKPOINT:** Cost estimation executed.

## PHASE 2: PARSE RESULTS

```bash
if [ "$COST_FAILED" != "true" ]; then
  CURRENT_TOTAL=$(jq -r '.totalMonthlyCost // "0"' "$ARTIFACTS_DIR/infracost-raw.json" 2>/dev/null || echo "0")
  PROPOSED_TOTAL=$(jq -r '.projects[0].breakdown.totalMonthlyCost // "0"' "$ARTIFACTS_DIR/infracost-raw.json" 2>/dev/null || echo "0")
  DELTA=$(jq -r '.diffTotalMonthlyCost // "0"' "$ARTIFACTS_DIR/infracost-raw.json" 2>/dev/null || echo "0")
else
  CURRENT_TOTAL="0"
  PROPOSED_TOTAL="0"
  DELTA="0"
fi

# Calculate annual projection
ANNUAL=$(echo "$PROPOSED_TOTAL * 12" | bc -l 2>/dev/null || echo "0")

# Check budget
BUDGET_EXCEEDED="false"
if [ -n "$MONTHLY_BUDGET_LIMIT" ] && [ "$MONTHLY_BUDGET_LIMIT" != "null" ]; then
  if [ "$(echo "$PROPOSED_TOTAL > $MONTHLY_BUDGET_LIMIT" | bc -l 2>/dev/null)" = "1" ]; then
    BUDGET_EXCEEDED="true"
  fi
fi
```

**CHECKPOINT:** Cost data parsed.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/cost-estimate.json" << EOF
{
  "tool": "infracost",
  "currency": "${COST_CURRENCY:-USD}",
  "period": "monthly",
  "current_total": $CURRENT_TOTAL,
  "proposed_total": $PROPOSED_TOTAL,
  "delta": $DELTA,
  "delta_percent": 0.0,
  "annual_projection": ${ANNUAL:-0},
  "top_cost_drivers": [],
  "budget_limit": ${MONTHLY_BUDGET_LIMIT:-null},
  "budget_exceeded": $BUDGET_EXCEEDED,
  "estimation_available": $([ "$COST_FAILED" != "true" ] && echo "true" || echo "false"),
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Cost estimate artifact written.

**EMIT:** `<promise>COST_ESTIMATED</promise>`
