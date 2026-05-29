# Onboard Team

Prepare the namespace, access, monitoring, and operational handoff plan required to onboard
a team into the platform.

## PRECONDITIONS CHECK

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: team or service onboarding request required in ARGUMENTS"
  exit 1
fi
```

## PHASE 1: WRITE ONBOARDING ARTIFACT

Write:
- `$ARTIFACTS_DIR/team-onboarding.md`
- `$ARTIFACTS_DIR/team-onboarding.json`

Include namespace needs, RBAC expectations, alert routing, and owner responsibilities.

**EMIT:** `<promise>TEAM_ONBOARDING_READY</promise>`
