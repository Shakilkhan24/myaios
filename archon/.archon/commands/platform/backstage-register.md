# Backstage Register

Prepare the registration data needed to add a new service or system to Backstage.

## PRECONDITIONS CHECK

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: service identifier required in ARGUMENTS"
  exit 1
fi
```

## PHASE 1: WRITE REGISTRATION ARTIFACT

Write:
- `$ARTIFACTS_DIR/backstage-registration.md`
- `$ARTIFACTS_DIR/backstage-registration.json`

Include service name, owner, lifecycle, and required catalog metadata.

**EMIT:** `<promise>BACKSTAGE_REGISTRATION_READY</promise>`
