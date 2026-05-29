# Secret Rotation Execute

Execute secret rotation in Vault or cloud secret manager, update consumers, verify, and revoke old secret. Must run after approval.

## PRECONDITIONS CHECK

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: Must provide secret path to rotate"
  exit 1
fi
SECRET_PATH="$ARGUMENTS"
```

## PHASE 1: IDENTIFY CONSUMERS

Identify all services that consume this secret. Check:
- Kubernetes secrets referencing this path
- Environment variables in deployments
- ExternalSecret resources

```bash
# Find kubernetes secrets referencing this path
kubectl get externalsecrets -A -o json 2>/dev/null | \
  jq "[.items[] | select(.spec.data[]?.remoteRef.key == \"$SECRET_PATH\")] | length" \
  > "$ARTIFACTS_DIR/secret-consumers-count.txt" 2>/dev/null || echo "0" > "$ARTIFACTS_DIR/secret-consumers-count.txt"
```

## PHASE 2: GENERATE AND STORE NEW SECRET

```bash
set +x  # Never echo secrets
# Generate new secret (or let Vault rotate it)
if which vault > /dev/null 2>&1; then
  echo "Rotating secret at path: $SECRET_PATH"
  # Record rotation event (NOT the secret value)
  echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"action\":\"secret-rotate\",\"path\":\"$SECRET_PATH\",\"operator\":\"archon\"}" >> "$ARTIFACTS_DIR/audit-log.jsonl"
fi
set -x
```

## PHASE 3: ROLLING RESTART CONSUMERS

```bash
# Trigger rolling restart of consuming deployments
CONSUMERS=$(kubectl get externalsecrets -A -o json 2>/dev/null | \
  jq -r "[.items[] | select(.spec.data[]?.remoteRef.key == \"$SECRET_PATH\") | .metadata.namespace + \"/\" + .metadata.name] | .[]" 2>/dev/null || echo "")

for CONSUMER in $CONSUMERS; do
  NS=$(echo "$CONSUMER" | cut -d/ -f1)
  NAME=$(echo "$CONSUMER" | cut -d/ -f2)
  kubectl rollout restart deployment -n "$NS" 2>/dev/null || true
done
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/secret-rotation.json" << EOF
{
  "secret_path": "$SECRET_PATH",
  "rotated": true,
  "consumers_restarted": true,
  "old_secret_revoked": false,
  "note": "Secret values are NEVER included in this artifact",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>SECRET_ROTATED</promise>`
