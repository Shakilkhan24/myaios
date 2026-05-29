# Vault Audit

Audit HashiCorp Vault policies, auth methods, and secret engines. Never writes secret values to artifacts (Invariant I4).

## PHASE 1: COLLECT VAULT METADATA

```bash
which vault || { echo "ERROR: vault CLI not found"; exit 1; }
# Verify auth — do NOT echo the token
set +x
vault token lookup -format=json 2>/dev/null | jq 'del(.data.id)' > "$ARTIFACTS_DIR/vault-token-info.json" || {
  echo "ERROR: Vault authentication failed"; exit 1
}
set -x
```

## PHASE 2: AUDIT POLICIES

```bash
# List policies (metadata only, never secret values)
vault policy list -format=json > "$ARTIFACTS_DIR/vault-policies-list.json" 2>&1
POLICY_COUNT=$(jq 'length' "$ARTIFACTS_DIR/vault-policies-list.json" 2>/dev/null || echo "0")

# List auth methods
vault auth list -format=json > "$ARTIFACTS_DIR/vault-auth-methods.json" 2>&1
AUTH_COUNT=$(jq 'keys | length' "$ARTIFACTS_DIR/vault-auth-methods.json" 2>/dev/null || echo "0")

# List secret engines
vault secrets list -format=json > "$ARTIFACTS_DIR/vault-secret-engines.json" 2>&1
ENGINE_COUNT=$(jq 'keys | length' "$ARTIFACTS_DIR/vault-secret-engines.json" 2>/dev/null || echo "0")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/vault-audit.json" << EOF
{
  "tool": "vault-audit",
  "policies": $POLICY_COUNT,
  "auth_methods": $AUTH_COUNT,
  "secret_engines": $ENGINE_COUNT,
  "findings": [],
  "note": "Secret values are NEVER included in this artifact",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>VAULT_AUDIT_COMPLETE</promise>`
