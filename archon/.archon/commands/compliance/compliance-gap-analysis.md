# Compliance Gap Analysis

Generate a compliance gap analysis from scan results. Identifies automatable vs manual controls, produces executive summary and remediation priorities.

## PHASE 1: AGGREGATE COMPLIANCE SCANS

```bash
# Collect all compliance scan artifacts
COMPLIANCE_SCANS=$(ls "$ARTIFACTS_DIR"/{cis-benchmark,soc2-controls,pci-dss-scan}.json 2>/dev/null)

TOTAL_CONTROLS=0; TOTAL_PASS=0; TOTAL_FAIL=0; TOTAL_MANUAL=0
for SCAN in $COMPLIANCE_SCANS; do
  TC=$(jq '.summary.total_controls // 0' "$SCAN" 2>/dev/null || echo "0")
  TP=$(jq '.summary.passed // 0' "$SCAN" 2>/dev/null || echo "0")
  TF=$(jq '.summary.failed // 0' "$SCAN" 2>/dev/null || echo "0")
  TM=$(jq '.summary.manual // 0' "$SCAN" 2>/dev/null || echo "0")
  TOTAL_CONTROLS=$((TOTAL_CONTROLS + TC))
  TOTAL_PASS=$((TOTAL_PASS + TP))
  TOTAL_FAIL=$((TOTAL_FAIL + TF))
  TOTAL_MANUAL=$((TOTAL_MANUAL + TM))
done
```

## PHASE 2: GENERATE GAP ANALYSIS

Produce both executive summary and detailed findings:

1. **Executive Summary**: 3-5 sentences on overall compliance posture
2. **Gap List**: Failed controls with remediation steps
3. **Automation Opportunities**: Controls that can be auto-remediated
4. **Exemptions**: Load from `$COMPLIANCE_EXEMPTIONS_FILE` and mark exempt controls

```bash
cat > "$ARTIFACTS_DIR/compliance-gap-analysis.json" << EOF
{
  "frameworks_scanned": [],
  "aggregate_summary": {
    "total_controls": $TOTAL_CONTROLS,
    "passed": $TOTAL_PASS,
    "failed": $TOTAL_FAIL,
    "manual": $TOTAL_MANUAL,
    "pass_rate_percent": $([ $TOTAL_CONTROLS -gt 0 ] && echo "$((TOTAL_PASS * 100 / TOTAL_CONTROLS))" || echo "0")
  },
  "gaps": [],
  "automatable_remediations": [],
  "exemptions_applied": [],
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>GAP_ANALYSIS_COMPLETE</promise>`
