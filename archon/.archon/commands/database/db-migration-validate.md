# DB Migration Validate

Validate the configured migration set in dry-run or lint mode before any write is applied to
the target database.

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

## PHASE 1: VALIDATE MIGRATIONS

```bash
case "$DB_MIGRATION_TOOL" in
  flyway) flyway validate > "$ARTIFACTS_DIR/db-migration-validate.log" 2>&1 ;;
  liquibase) liquibase validate > "$ARTIFACTS_DIR/db-migration-validate.log" 2>&1 ;;
  atlas) atlas migrate lint --dir file://migrations > "$ARTIFACTS_DIR/db-migration-validate.log" 2>&1 ;;
  dbmate) dbmate status > "$ARTIFACTS_DIR/db-migration-validate.log" 2>&1 ;;
  *) echo "ERROR: unsupported migration tool $DB_MIGRATION_TOOL" ; exit 1 ;;
esac
VALIDATE_EXIT=$?
STATUS=$([ "$VALIDATE_EXIT" -eq 0 ] && echo "success" || echo "failed")
if [ "$VALIDATE_EXIT" -ne 0 ]; then
  exit 1
fi
```

**CHECKPOINT:** Migration validation completed.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/db-migration-validate.json" << EOF
{
  "tool": "$DB_MIGRATION_TOOL",
  "status": "$STATUS",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Validation artifact written.

**EMIT:** `<promise>DB_MIGRATION_VALIDATED</promise>`
