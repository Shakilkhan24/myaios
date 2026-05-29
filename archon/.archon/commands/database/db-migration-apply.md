# DB Migration Apply

Apply the validated database migration set and write an audit artifact describing the
execution outcome for change tracking and rollback planning.

## PRECONDITIONS CHECK

### A.1 Validate Tooling

```bash
if [ -z "$DB_MIGRATION_TOOL" ]; then
  echo "ERROR: DB_MIGRATION_TOOL is required"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Migration tool configured

## PHASE 1: APPLY MIGRATIONS

```bash
case "$DB_MIGRATION_TOOL" in
  flyway) flyway migrate > "$ARTIFACTS_DIR/db-migration-apply.log" 2>&1 ;;
  liquibase) liquibase update > "$ARTIFACTS_DIR/db-migration-apply.log" 2>&1 ;;
  atlas) atlas migrate apply --dir file://migrations > "$ARTIFACTS_DIR/db-migration-apply.log" 2>&1 ;;
  dbmate) dbmate up > "$ARTIFACTS_DIR/db-migration-apply.log" 2>&1 ;;
  *) echo "ERROR: unsupported migration tool $DB_MIGRATION_TOOL" ; exit 1 ;;
esac
APPLY_EXIT=$?
STATUS=$([ "$APPLY_EXIT" -eq 0 ] && echo "success" || echo "failed")
if [ "$APPLY_EXIT" -ne 0 ]; then
  exit 1
fi
```

**CHECKPOINT:** Migration execution completed.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/db-migration-apply.json" << EOF
{
  "tool": "$DB_MIGRATION_TOOL",
  "status": "$STATUS",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Apply artifact written.

**EMIT:** `<promise>DB_MIGRATION_APPLIED</promise>`
