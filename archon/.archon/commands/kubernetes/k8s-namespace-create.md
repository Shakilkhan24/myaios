# Kubernetes Namespace Create

Create and configure a new Kubernetes namespace with RBAC, NetworkPolicy, LimitRange, ResourceQuota, and service accounts. Idempotent — handles "already exists" case.

## PHASE 0: VALIDATE INPUT

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: Must provide namespace name"
  exit 1
fi
if ! echo "$ARGUMENTS" | grep -qE '^[a-z0-9][a-z0-9-]{0,62}$'; then
  echo "ERROR: Invalid namespace name. Must be lowercase alphanumeric with hyphens."
  exit 1
fi
NAMESPACE="$ARGUMENTS"
```

## PHASE 1: CREATE NAMESPACE

```bash
# Idempotent creation
kubectl get namespace "$NAMESPACE" > /dev/null 2>&1 && {
  echo "Namespace $NAMESPACE already exists — updating configuration"
} || {
  kubectl create namespace "$NAMESPACE"
  echo "Namespace $NAMESPACE created"
}

# Apply labels
kubectl label namespace "$NAMESPACE" \
  environment="$ENVIRONMENT" \
  team="$TEAM" \
  managed-by=archon \
  --overwrite
```

**CHECKPOINT:** Namespace exists.

## PHASE 2: APPLY RESOURCE QUOTA

```bash
cat <<EOF | kubectl apply -f - -n "$NAMESPACE"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: default-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "50"
    services: "10"
    persistentvolumeclaims: "10"
EOF
```

## PHASE 3: APPLY LIMIT RANGE

```bash
cat <<EOF | kubectl apply -f - -n "$NAMESPACE"
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
EOF
```

## PHASE 4: APPLY NETWORK POLICY

```bash
cat <<EOF | kubectl apply -f - -n "$NAMESPACE"
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/namespace-create.json" << EOF
{
  "namespace": "$NAMESPACE",
  "environment": "$ENVIRONMENT",
  "resources_applied": ["ResourceQuota", "LimitRange", "NetworkPolicy"],
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>NAMESPACE_CREATED</promise>`
