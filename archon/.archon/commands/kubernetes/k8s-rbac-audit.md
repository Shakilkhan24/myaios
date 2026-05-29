# Kubernetes RBAC Audit

Audit RBAC roles, role bindings, cluster roles, and service accounts for overly permissive access and security violations.

## PHASE 1: COLLECT RBAC DATA

```bash
which kubectl || { echo "ERROR: kubectl not found"; exit 1; }

# Collect all RBAC resources
kubectl get clusterroles -o json > "$ARTIFACTS_DIR/rbac-clusterroles.json" 2>&1
kubectl get clusterrolebindings -o json > "$ARTIFACTS_DIR/rbac-clusterrolebindings.json" 2>&1
kubectl get roles -A -o json > "$ARTIFACTS_DIR/rbac-roles.json" 2>&1
kubectl get rolebindings -A -o json > "$ARTIFACTS_DIR/rbac-rolebindings.json" 2>&1
kubectl get serviceaccounts -A -o json > "$ARTIFACTS_DIR/rbac-serviceaccounts.json" 2>&1
```

**CHECKPOINT:** RBAC data collected.

## PHASE 2: IDENTIFY VIOLATIONS

Analyze for these RBAC anti-patterns:

1. **Wildcard permissions** — ClusterRoles with `*` verbs or `*` resources
2. **Cluster-admin bindings** — Non-system bindings to `cluster-admin`
3. **Excessive namespace admin** — Too many users with namespace admin access
4. **Unused service accounts** — Service accounts not referenced by any pod
5. **Default service account usage** — Pods using the `default` service account
6. **Secret access** — Roles that can read secrets in non-essential namespaces

```bash
# Check for wildcard cluster roles
WILDCARD_ROLES=$(jq '[.items[] | select(.rules[]? | .verbs[]? == "*" or .resources[]? == "*") | .metadata.name] | unique | length' "$ARTIFACTS_DIR/rbac-clusterroles.json" 2>/dev/null || echo "0")

# Check for cluster-admin bindings
ADMIN_BINDINGS=$(jq '[.items[] | select(.roleRef.name == "cluster-admin") | .metadata.name] | length' "$ARTIFACTS_DIR/rbac-clusterrolebindings.json" 2>/dev/null || echo "0")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/rbac-audit.json" << EOF
{
  "tool": "kubectl-rbac-audit",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "summary": {
    "wildcard_roles": $WILDCARD_ROLES,
    "cluster_admin_bindings": $ADMIN_BINDINGS
  },
  "findings": [],
  "recommendations": [],
  "gate_result": "$([ "$WILDCARD_ROLES" -gt "5" ] && echo "fail" || echo "pass")"
}
EOF
```

**EMIT:** `<promise>RBAC_AUDIT_COMPLETE</promise>`
