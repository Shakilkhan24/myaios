# Notify Security Findings

Dispatches security scan findings to configured notification channels. Sends critical/high finding alerts to security team channels and PagerDuty for critical findings.

## PHASE 0: VALIDATE INPUT

```bash
# Locate security scan artifacts
SCAN_ARTIFACTS=$(ls "$ARTIFACTS_DIR"/*-scan.json "$ARTIFACTS_DIR"/*-findings.json 2>/dev/null)
if [ -z "$SCAN_ARTIFACTS" ]; then
  echo "WARNING: No security scan artifacts found"
  exit 0
fi
```

**CHECKPOINT:** Security scan artifacts located.

## PHASE 1: AGGREGATE FINDINGS

```bash
# Initialize counters
TOTAL_CRITICAL=0
TOTAL_HIGH=0
TOTAL_MEDIUM=0
TOTAL_LOW=0
TOOLS_SCANNED=""

# Aggregate across all scan artifacts
for SCAN_FILE in $SCAN_ARTIFACTS; do
  TOOL=$(jq -r '.tool // "unknown"' "$SCAN_FILE" 2>/dev/null || echo "unknown")
  CRITICAL=$(jq -r '.summary.critical // 0' "$SCAN_FILE" 2>/dev/null || echo "0")
  HIGH=$(jq -r '.summary.high // 0' "$SCAN_FILE" 2>/dev/null || echo "0")
  MEDIUM=$(jq -r '.summary.medium // 0' "$SCAN_FILE" 2>/dev/null || echo "0")
  LOW=$(jq -r '.summary.low // 0' "$SCAN_FILE" 2>/dev/null || echo "0")

  TOTAL_CRITICAL=$((TOTAL_CRITICAL + CRITICAL))
  TOTAL_HIGH=$((TOTAL_HIGH + HIGH))
  TOTAL_MEDIUM=$((TOTAL_MEDIUM + MEDIUM))
  TOTAL_LOW=$((TOTAL_LOW + LOW))
  TOOLS_SCANNED="$TOOLS_SCANNED $TOOL"
done

TOTAL_FINDINGS=$((TOTAL_CRITICAL + TOTAL_HIGH + TOTAL_MEDIUM + TOTAL_LOW))
```

**CHECKPOINT:** Findings aggregated across all scan tools.

## PHASE 2: DETERMINE SEVERITY AND DISPATCH

```bash
# Determine alert severity
if [ "$TOTAL_CRITICAL" -gt "0" ]; then
  ALERT_SEVERITY="critical"
  EMOJI="🚨"
  COLOR="danger"
elif [ "$TOTAL_HIGH" -gt "0" ]; then
  ALERT_SEVERITY="high"
  EMOJI="⚠️"
  COLOR="warning"
else
  ALERT_SEVERITY="info"
  EMOJI="ℹ️"
  COLOR="good"
fi

NOTIFICATION_MSG="$EMOJI Security Scan: $TOTAL_FINDINGS findings ($TOTAL_CRITICAL critical, $TOTAL_HIGH high, $TOTAL_MEDIUM medium, $TOTAL_LOW low) | Tools: $TOOLS_SCANNED"

# Slack notification
if [ -n "$SLACK_CHANNEL" ] && [ -n "$SLACK_WEBHOOK_URL" ]; then
  set +x
  curl -sf -X POST "$SLACK_WEBHOOK_URL" \
    -H 'Content-Type: application/json' \
    -d "{\"channel\":\"$SLACK_CHANNEL\",\"text\":\"$NOTIFICATION_MSG\"}" \
    || echo "WARNING: Slack notification failed"
  set -x
fi

# PagerDuty alert for critical findings
if [ "$TOTAL_CRITICAL" -gt "0" ] && [ -n "$PAGERDUTY_ROUTING_KEY" ]; then
  set +x
  curl -sf -X POST "https://events.pagerduty.com/v2/enqueue" \
    -H 'Content-Type: application/json' \
    -d "{\"routing_key\":\"[REDACTED]\",\"event_action\":\"trigger\",\"payload\":{\"summary\":\"$TOTAL_CRITICAL critical security findings detected\",\"severity\":\"critical\",\"source\":\"archon-security-scan\"}}" \
    || echo "WARNING: PagerDuty notification failed"
  set -x
fi
```

**CHECKPOINT:** Security notifications dispatched.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/security-notification.json" << EOF
{
  "alert_severity": "$ALERT_SEVERITY",
  "total_findings": $TOTAL_FINDINGS,
  "summary": {
    "critical": $TOTAL_CRITICAL,
    "high": $TOTAL_HIGH,
    "medium": $TOTAL_MEDIUM,
    "low": $TOTAL_LOW
  },
  "tools_scanned": "$(echo $TOOLS_SCANNED | xargs)",
  "notification_channels": {
    "slack": $([ -n "$SLACK_CHANNEL" ] && echo "true" || echo "false"),
    "pagerduty": $([ "$TOTAL_CRITICAL" -gt "0" ] && [ -n "$PAGERDUTY_ROUTING_KEY" ] && echo "true" || echo "false")
  },
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>SECURITY_NOTIFIED</promise>`
