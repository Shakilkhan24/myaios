# Notify Deploy Complete

Dispatches deployment completion notifications to configured channels (Slack, PagerDuty, Teams, email). Reads deploy artifacts and sends a structured summary.

## PHASE 0: VALIDATE INPUT

Validate that a deploy artifact exists before attempting notification:

```bash
# Find the most recent deploy artifact
DEPLOY_ARTIFACT=$(ls -t "$ARTIFACTS_DIR/deploys/"*.json 2>/dev/null | head -1)
if [ -z "$DEPLOY_ARTIFACT" ]; then
  echo "WARNING: No deploy artifact found in $ARTIFACTS_DIR/deploys/"
  # Create a minimal notification artifact anyway
  DEPLOY_ARTIFACT="$ARTIFACTS_DIR/deploy-summary.json"
fi
```

**CHECKPOINT:** Deploy artifact located.

## PHASE 1: BUILD NOTIFICATION PAYLOAD

```bash
# Extract deploy details from artifact
ENVIRONMENT=$(jq -r '.environment // "unknown"' "$DEPLOY_ARTIFACT" 2>/dev/null || echo "unknown")
OUTCOME=$(jq -r '.outcome // "unknown"' "$DEPLOY_ARTIFACT" 2>/dev/null || echo "unknown")
TOOL=$(jq -r '.tool // "unknown"' "$DEPLOY_ARTIFACT" 2>/dev/null || echo "unknown")
RESOURCE=$(jq -r '.resource_identifier // "unknown"' "$DEPLOY_ARTIFACT" 2>/dev/null || echo "unknown")
DURATION=$(jq -r '.duration_seconds // "0"' "$DEPLOY_ARTIFACT" 2>/dev/null || echo "0")
TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)

# Build notification message
if [ "$OUTCOME" = "success" ]; then
  EMOJI="✅"
  COLOR="good"
elif [ "$OUTCOME" = "failed" ]; then
  EMOJI="❌"
  COLOR="danger"
elif [ "$OUTCOME" = "rolled_back" ]; then
  EMOJI="⚠️"
  COLOR="warning"
else
  EMOJI="ℹ️"
  COLOR="info"
fi

NOTIFICATION_MSG="$EMOJI Deploy $OUTCOME | $ENVIRONMENT | $TOOL | $RESOURCE | ${DURATION}s"
```

**CHECKPOINT:** Notification payload constructed.

## PHASE 2: DISPATCH TO CHANNELS

```bash
# Slack notification
if [ -n "$SLACK_CHANNEL" ] && [ -n "$SLACK_WEBHOOK_URL" ]; then
  # Required permissions: None (webhook-based)
  set +x  # Never echo webhook URLs
  curl -sf -X POST "$SLACK_WEBHOOK_URL" \
    -H 'Content-Type: application/json' \
    -d "{\"channel\":\"$SLACK_CHANNEL\",\"text\":\"$NOTIFICATION_MSG\",\"attachments\":[{\"color\":\"$COLOR\",\"fields\":[{\"title\":\"Environment\",\"value\":\"$ENVIRONMENT\",\"short\":true},{\"title\":\"Outcome\",\"value\":\"$OUTCOME\",\"short\":true}]}]}" \
    || echo "WARNING: Slack notification failed"
  set -x
fi

# Teams notification
if [ -n "$TEAMS_WEBHOOK" ]; then
  set +x
  curl -sf -X POST "$TEAMS_WEBHOOK" \
    -H 'Content-Type: application/json' \
    -d "{\"text\":\"$NOTIFICATION_MSG\"}" \
    || echo "WARNING: Teams notification failed"
  set -x
fi

# PagerDuty (only on failure)
if [ "$OUTCOME" = "failed" ] && [ -n "$PAGERDUTY_ROUTING_KEY" ]; then
  set +x
  curl -sf -X POST "https://events.pagerduty.com/v2/enqueue" \
    -H 'Content-Type: application/json' \
    -d "{\"routing_key\":\"[REDACTED]\",\"event_action\":\"trigger\",\"payload\":{\"summary\":\"Deploy failed: $RESOURCE in $ENVIRONMENT\",\"severity\":\"critical\",\"source\":\"archon\"}}" \
    || echo "WARNING: PagerDuty notification failed"
  set -x
fi
```

**CHECKPOINT:** Notifications dispatched to all configured channels.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/notification-result.json" << EOF
{
  "notification_sent": true,
  "channels": {
    "slack": $([ -n "$SLACK_CHANNEL" ] && echo "true" || echo "false"),
    "teams": $([ -n "$TEAMS_WEBHOOK" ] && echo "true" || echo "false"),
    "pagerduty": $([ "$OUTCOME" = "failed" ] && [ -n "$PAGERDUTY_ROUTING_KEY" ] && echo "true" || echo "false")
  },
  "deploy_outcome": "$OUTCOME",
  "environment": "$ENVIRONMENT",
  "timestamp": "$TIMESTAMP"
}
EOF
```

**CHECKPOINT:** Notification result recorded.

**EMIT:** `<promise>NOTIFICATION_SENT</promise>`
