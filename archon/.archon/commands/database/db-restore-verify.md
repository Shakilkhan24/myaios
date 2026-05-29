# DB Restore Verify

Restore a backup to a temporary target and verify basic integrity before the backup is trusted
as a valid recovery artifact.

## PRECONDITIONS CHECK

### A.1 Validate Backup Input

```bash
if [ -z "$ARGUMENTS" ] || [ ! -f "$ARGUMENTS" ]; then
  echo "ERROR: backup file path required in ARGUMENTS"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Backup file exists

## PHASE 1: VERIFY RESTORE PLAN

Write the restore verification plan to:
- `$ARTIFACTS_DIR/db-restore-verify.md`
- `$ARTIFACTS_DIR/db-restore-verify.json`

The plan must capture:
1. source backup file
2. target restore environment or temporary database
3. integrity checks to run after restore
4. cleanup action for the temporary restore target

If a live restore cannot be executed safely in this context, say so explicitly and mark the
status as `planned-only` rather than `verified`.

**CHECKPOINT:** Restore verification artifacts written.

**EMIT:** `<promise>DB_RESTORE_VERIFIED</promise>`
