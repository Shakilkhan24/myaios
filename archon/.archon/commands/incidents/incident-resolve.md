# Incident Resolve

Close an incident after confirming recovery, update the final status artifact, and capture any
remaining follow-up actions for operational tracking.

## PRECONDITIONS CHECK

### A.1 Validate Resolution Context

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: incident resolution summary required in ARGUMENTS"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Resolution context provided

## PHASE 1: CONFIRM RECOVERY

Review any available health, timeline, or post-mortem artifacts and confirm the incident can be
marked resolved.

Write:
- `$ARTIFACTS_DIR/incident-resolution.md`
- `$ARTIFACTS_DIR/incident-resolution.json`

The JSON artifact must include:
```json
{
  "status": "resolved|monitoring|not-resolved",
  "summary": "short resolution summary",
  "follow_up_required": true,
  "timestamp": "ISO-8601"
}
```

**CHECKPOINT:** Resolution artifacts written.

## PHASE 2: RECORD FOLLOW-UP

If follow-up work remains, summarize it in `$ARTIFACTS_DIR/incident-follow-up.md` with:
1. what still needs to change
2. whether it is code, config, or process
3. the urgency level

If no follow-up remains, write a one-line closure note instead.

**CHECKPOINT:** Follow-up note written.

**EMIT:** `<promise>INCIDENT_RESOLVED</promise>`
