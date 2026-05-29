# Post-Mortem Draft

Draft a blameless post-mortem from incident artifacts so teams can review causes, impact,
and action items without recreating the narrative from scratch.

## PRECONDITIONS CHECK

### A.1 Verify Supporting Artifacts

```bash
if [ ! -f "$ARTIFACTS_DIR/incident-timeline.md" ]; then
  echo "ERROR: incident-timeline.md is required"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Incident timeline exists

## PHASE 1: DRAFT POST-MORTEM

Use:
- `$ARTIFACTS_DIR/incident-timeline.md`
- `$ARTIFACTS_DIR/incident-classification.json`
- `$ARTIFACTS_DIR/alert-correlation.md`

Write:
- `$ARTIFACTS_DIR/post-mortem.md`
- `$ARTIFACTS_DIR/post-mortem.json`

The draft must include:
1. incident summary
2. impact and affected users
3. timeline
4. contributing factors
5. remediation performed
6. follow-up action items with owners or placeholders

Keep the tone blameless and factual.

**CHECKPOINT:** Post-mortem artifacts written.

**EMIT:** `<promise>POST_MORTEM_READY</promise>`
