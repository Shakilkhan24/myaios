# Runbook Executor

Execute a named runbook step by step with explicit state tracking and verification at each
stage so incident response remains auditable and resumable.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: runbook name or runbook path required in ARGUMENTS"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Runbook target provided

## PHASE 1: LOAD RUNBOOK

Locate the runbook referenced by `$ARGUMENTS`. Search these locations in order:
1. `.archon/runbooks/`
2. `docs/runbooks/`
3. `runbooks/`

Write the selected path and any fallback notes to `$ARTIFACTS_DIR/runbook-selection.json`.

**CHECKPOINT:** Runbook located or missing-state recorded.

## PHASE 2: EXECUTE STEP BY STEP

Read the selected runbook and execute it one step at a time.

For each step:
1. Record the step number and action
2. Execute only the minimum required action
3. Verify the result before moving on
4. Append the status to `$ARTIFACTS_DIR/runbook-execution.jsonl`

Write a human-readable summary to `$ARTIFACTS_DIR/runbook-execution.md`.

If you encounter an irreversible action, pause and require explicit approval before continuing.

**CHECKPOINT:** Runbook execution state recorded.

**EMIT:** `<promise>RUNBOOK_EXECUTED</promise>`
