# SOC2 Control Check

Automated SOC2 Type II evidence gathering for Trust Service Criteria. Maps to CC (Common Criteria) control IDs.

## PHASE 1: GATHER EVIDENCE

Collect evidence for key SOC2 controls:

1. **CC6.1 — Logical Access**: Verify RBAC, MFA, password policies
2. **CC6.2 — Credentials**: Check credential rotation policies
3. **CC6.3 — Least Privilege**: Audit IAM for overly permissive roles
4. **CC7.1 — System Monitoring**: Verify logging/alerting is active
5. **CC7.2 — Anomaly Detection**: Check for security monitoring
6. **CC8.1 — Change Management**: Verify approval workflows exist

```bash
# Check logging is enabled
LOGGING_ENABLED="false"
if kubectl get daemonset -n kube-system -l app=fluentd 2>/dev/null | grep -q fluentd; then LOGGING_ENABLED="true"; fi
if kubectl get daemonset -n monitoring -l app=promtail 2>/dev/null | grep -q promtail; then LOGGING_ENABLED="true"; fi

# Check RBAC is enabled
RBAC_ENABLED=$(kubectl api-versions 2>/dev/null | grep -c "rbac.authorization.k8s.io" || echo "0")

cat > "$ARTIFACTS_DIR/soc2-controls.json" << EOF
{"framework":"soc2-type2","scan_date":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","scope":"cluster","summary":{"total_controls":6,"passed":0,"failed":0,"manual":6,"not_applicable":0,"pass_rate_percent":0},"findings":[{"control_id":"CC6.1","title":"Logical Access Controls","status":"manual","severity":"high","automatable":false},{"control_id":"CC7.1","title":"System Monitoring","status":"$([ "$LOGGING_ENABLED" = "true" ] && echo "pass" || echo "fail")","severity":"high","automatable":true}]}
EOF
```

**EMIT:** `<promise>SOC2_CHECK_COMPLETE</promise>`
