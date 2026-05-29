# Incident Classify

Classify an incident by severity, service impact, and likely runbook so downstream workflow
steps know whether to escalate immediately or continue with standard handling.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: incident description required in ARGUMENTS"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Incident description provided

## PHASE 1: CLASSIFY INCIDENT

Read the incident description from `$ARGUMENTS`.

Write:
- `$ARTIFACTS_DIR/incident-classification.md`
- `$ARTIFACTS_DIR/incident-classification.json`

The JSON artifact must include:
```json
{
  "severity": "P1|P2|P3|P4",
  "category": "availability|latency|security|data|deployment|unknown",
  "runbook": "suggested-runbook-name",
  "page_oncall": true,
  "timestamp": "ISO-8601"
}
```

Base the classification on user impact, blast radius, and urgency to mitigate.

**CHECKPOINT:** Incident classification artifacts written.

**EMIT:** `<promise>INCIDENT_CLASSIFIED</promise>`
