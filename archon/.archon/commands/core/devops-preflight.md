# DevOps Pre-Flight Check

Universal pre-flight verification for all DevOps workflows. Runs tool availability checks, authentication verification, and environment lock confirmation before any infrastructure operation.

## PRECONDITIONS CHECK

### A.1 Verify Tool Availability

```bash
# Check Terraform
which terraform || { echo "ERROR: terraform not found"; exit 1; }
terraform version | head -1

# Check kubectl
which kubectl || { echo "ERROR: kubectl not found"; exit 1; }
kubectl version --client | head -1

# Check Helm
which helm || { echo "ERROR: helm not found"; exit 1; }
helm version | head -1

# Check Docker
which docker || { echo "ERROR: docker not found"; exit 1; }
docker --version
```

### A.2 Verify Cloud Authentication

```bash
# Verify cloud provider authentication based on config
if [ "$CLOUD_PROVIDER" = "aws" ]; then
  aws sts get-caller-identity --output json || { echo "ERROR: AWS auth failed"; exit 1; }
elif [ "$CLOUD_PROVIDER" = "gcp" ]; then
  gcloud auth list --filter=status:ACTIVE --format="value(account)" || { echo "ERROR: GCP auth failed"; exit 1; }
elif [ "$CLOUD_PROVIDER" = "azure" ]; then
  az account show --output json || { echo "ERROR: Azure auth failed"; exit 1; }
fi
```

### A.3 Verify Environment Lock

```bash
# Confirm environment matches intent
if [ "$ENVIRONMENT" = "production" ]; then
  echo "WARNING: Targeting PRODUCTION environment"
  if [ -z "$CONFIRMATION_TOKEN" ]; then
    echo "ERROR: Production operations require CONFIRMATION_TOKEN"
    exit 1
  fi
fi
```

### A.4 Verify Concurrent Operations

```bash
# Check for concurrent operations using a lock file
LOCK_FILE="$ARTIFACTS_DIR/.archon-lock"
if [ -f "$LOCK_FILE" ]; then
  echo "ERROR: Concurrent operation detected at $(cat $LOCK_FILE)"
  exit 1
fi
echo "$$" > "$LOCK_FILE"
trap "rm -f $LOCK_FILE" EXIT
```

**PRECONDITION_CHECKPOINT:**

- [ ] Tools present and correct version
- [ ] Cloud auth valid and for correct account
- [ ] Environment matches intent
- [ ] No concurrent operations in progress

## PHASE 1: TOOL VERSION CHECK

Capture all tool versions for the artifact:

```bash
cat > "$ARTIFACTS_DIR/tool-versions.json" << 'EOF'
{
  "terraform": "$(terraform version | head -1)",
  "kubectl": "$(kubectl version --client 2>/dev/null | head -1)",
  "helm": "$(helm version 2>/dev/null | head -1)",
  "docker": "$(docker --version 2>/dev/null || echo 'not installed')",
  "cloud_provider": "'$CLOUD_PROVIDER'",
  "cloud_region": "'$CLOUD_REGION'",
  "environment": "'$ENVIRONMENT'",
  "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
}
EOF
```

## PHASE 2: AUTHENTICATION VERIFICATION

```bash
# Verify and record authentication state
if [ "$CLOUD_PROVIDER" = "aws" ]; then
  aws sts get-caller-identity --output json > "$ARTIFACTS_DIR/auth-check.json" 2>&1
  IDENTITY=$(aws sts get-caller-identity --query 'Arn' --output text)
  echo "Authenticated as: $IDENTITY"
elif [ "$CLOUD_PROVIDER" = "gcp" ]; then
  gcloud auth list --filter=status:ACTIVE --format="value(account)" > "$ARTIFACTS_DIR/auth-check.json" 2>&1
elif [ "$CLOUD_PROVIDER" = "azure" ]; then
  az account show --output json > "$ARTIFACTS_DIR/auth-check.json" 2>&1
fi
```

## PHASE 3: ENVIRONMENT VALIDATION

```bash
# Validate environment variable
case "$ENVIRONMENT" in
  dev|staging|production)
    echo "Environment validated: $ENVIRONMENT"
    ;;
  *)
    echo "ERROR: Invalid ENVIRONMENT '$ENVIRONMENT'. Must be dev, staging, or production"
    exit 1
    ;;
esac
```

## ARTIFACT OUTPUT

Write the preflight result:

```bash
cat > "$ARTIFACTS_DIR/preflight.json" << 'EOF'
{
  "status": "pass",
  "environment": "'$ENVIRONMENT'",
  "cloud_provider": "'$CLOUD_PROVIDER'",
  "cloud_region": "'$CLOUD_REGION'",
  "k8s_context": "'$K8S_CONTEXT'",
  "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
}
EOF
```

**CHECKPOINT:** All preconditions satisfied. Proceed with workflow.

**EMIT:** `<promise>PREFLIGHT_PASS</promise>`
