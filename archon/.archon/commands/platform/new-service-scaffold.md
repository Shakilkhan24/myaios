# New Service Scaffold

Plan the repository, CI/CD, and runtime scaffolding needed for a new service bootstrap.

## PRECONDITIONS CHECK

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: service name or bootstrap request required in ARGUMENTS"
  exit 1
fi
```

## PHASE 1: WRITE SCAFFOLD PLAN

Write:
- `$ARTIFACTS_DIR/new-service-scaffold.md`
- `$ARTIFACTS_DIR/new-service-scaffold.json`

Cover repo layout, pipeline requirements, deployment target, and ownership metadata.

**EMIT:** `<promise>NEW_SERVICE_SCAFFOLD_READY</promise>`
