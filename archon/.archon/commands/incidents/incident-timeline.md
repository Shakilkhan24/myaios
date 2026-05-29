# Incident Timeline

Reconstruct an incident timeline from logs, metrics, commits, and operator notes so post-mortem
and incident resolution steps have a single chronological source of truth.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: incident identifier or summary required in ARGUMENTS"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Incident context provided

## PHASE 1: GATHER TIMELINE INPUTS

Collect available inputs from:
- `$ARTIFACTS_DIR/metrics-query.json`
- `$ARTIFACTS_DIR/loki-search.json`
- `$ARTIFACTS_DIR/alert-correlation.json`
- recent git history if deployment activity is relevant

Write an input inventory to `$ARTIFACTS_DIR/incident-timeline-inputs.json`.

**CHECKPOINT:** Timeline evidence inventory written.

## PHASE 2: BUILD TIMELINE

Write:
- `$ARTIFACTS_DIR/incident-timeline.md`
- `$ARTIFACTS_DIR/incident-timeline.json`

The timeline must include:
1. first observable symptom
2. alerting or detection moment
3. mitigation attempts
4. recovery point
5. unresolved follow-up gaps

Each item should include a timestamp or relative ordering note and the supporting evidence source.

**CHECKPOINT:** Timeline artifacts written.

**EMIT:** `<promise>INCIDENT_TIMELINE_READY</promise>`
