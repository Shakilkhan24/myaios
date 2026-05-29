# Alert Correlate

Correlate an alert with nearby metrics and logs so downstream incident commands can work from
one consolidated evidence set instead of separate tool outputs.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: alert identifier or alert payload summary required in ARGUMENTS"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Alert context supplied

## PHASE 1: COLLECT AVAILABLE EVIDENCE

```bash
cat > "$ARTIFACTS_DIR/alert-context.json" << EOF
{
  "alert_context": "$ARGUMENTS",
  "metrics_artifact_present": $([ -f "$ARTIFACTS_DIR/metrics-query.json" ] && echo "true" || echo "false"),
  "logs_artifact_present": $([ -f "$ARTIFACTS_DIR/loki-search.json" ] && echo "true" || echo "false"),
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Alert context captured.

## PHASE 2: SYNTHESIZE CORRELATION

Read any available artifacts:
- `$ARTIFACTS_DIR/alert-context.json`
- `$ARTIFACTS_DIR/metrics-query.json`
- `$ARTIFACTS_DIR/loki-search.json`

Write:
- `$ARTIFACTS_DIR/alert-correlation.md`
- `$ARTIFACTS_DIR/alert-correlation.json`

Your synthesis must identify:
1. What signal fired
2. Which metric changes and log patterns line up with it
3. The most plausible failure window
4. The next best investigation step

**CHECKPOINT:** Correlation artifacts written.

**EMIT:** `<promise>ALERT_CORRELATION_READY</promise>`
