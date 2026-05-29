# DB Backup

Trigger a database backup and record the resulting backup metadata so migration and recovery
workflows have a verifiable restore point.

## PRECONDITIONS CHECK

### A.1 Validate Database Access

```bash
if [ -z "$DATABASE_URL" ]; then
  echo "ERROR: DATABASE_URL is required"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Database URL configured

## PHASE 1: TRIGGER BACKUP

```bash
BACKUP_FILE="$ARTIFACTS_DIR/db-backup-$(date +%Y%m%d%H%M%S).sql"
pg_dump "$DATABASE_URL" > "$BACKUP_FILE"
BACKUP_EXIT=$?
if [ "$BACKUP_EXIT" -ne 0 ]; then
  echo "ERROR: database backup failed"
  exit 1
fi
```

**CHECKPOINT:** Backup file created.

## PHASE 2: WRITE METADATA

```bash
cat > "$ARTIFACTS_DIR/db-backup.json" << EOF
{
  "backup_file": "$BACKUP_FILE",
  "status": "success",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Backup metadata written.

**EMIT:** `<promise>DB_BACKUP_READY</promise>`
