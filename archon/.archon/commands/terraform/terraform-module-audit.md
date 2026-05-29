# Terraform Module Audit

Review Terraform modules for version currency, security best practices, code quality, and provider compatibility. Produces a structured audit report.

## PHASE 1: DISCOVER MODULES

```bash
cd "$TF_ROOT"

# Find all module references in root and modules directory
MODULES_DIR="${TF_MODULES_DIR:-infra/modules/}"

# List all .tf files with module blocks
grep -rl 'module "' . --include="*.tf" 2>/dev/null | sort > "$ARTIFACTS_DIR/tf-module-files.txt"

# Extract module sources
grep -h 'source' $(cat "$ARTIFACTS_DIR/tf-module-files.txt" 2>/dev/null) 2>/dev/null | \
  sed 's/.*source.*=.*"\(.*\)"/\1/' | sort -u > "$ARTIFACTS_DIR/tf-module-sources.txt"

MODULE_COUNT=$(wc -l < "$ARTIFACTS_DIR/tf-module-sources.txt" 2>/dev/null || echo "0")
echo "Found $MODULE_COUNT unique module sources"
```

**CHECKPOINT:** Modules discovered.

## PHASE 2: VERSION AUDIT

```bash
# Check for pinned versions
UNPINNED=$(grep -c '\.git$\|\.git"$' "$ARTIFACTS_DIR/tf-module-sources.txt" 2>/dev/null || echo "0")
REGISTRY_MODULES=$(grep -c 'registry.terraform.io\|^[a-z]' "$ARTIFACTS_DIR/tf-module-sources.txt" 2>/dev/null || echo "0")
LOCAL_MODULES=$(grep -c '^\.\./\|^\./\|^\.\\' "$ARTIFACTS_DIR/tf-module-sources.txt" 2>/dev/null || echo "0")

# Check .terraform.lock.hcl for provider versions
if [ -f ".terraform.lock.hcl" ]; then
  PROVIDER_COUNT=$(grep -c 'provider "' .terraform.lock.hcl 2>/dev/null || echo "0")
else
  PROVIDER_COUNT=0
fi
```

**CHECKPOINT:** Version audit complete.

## PHASE 3: SECURITY REVIEW

Analyze modules for common security anti-patterns:

1. **Hardcoded credentials** — any `access_key`, `secret_key`, `password` in .tf files
2. **Public access** — `0.0.0.0/0` ingress rules, public S3 buckets, public RDS instances
3. **Missing encryption** — unencrypted EBS, S3, RDS, or Secrets Manager resources
4. **Overly permissive IAM** — `*` actions or resources in IAM policies
5. **Missing logging** — CloudTrail, VPC flow logs, access logging not enabled

```bash
# Check for hardcoded secrets
SECRET_FINDINGS=$(grep -rnl 'access_key\|secret_key\|password\s*=' . --include="*.tf" 2>/dev/null | wc -l || echo "0")

# Check for public access patterns
PUBLIC_ACCESS=$(grep -rnl '0\.0\.0\.0/0\|public.*=.*true' . --include="*.tf" 2>/dev/null | wc -l || echo "0")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/terraform-module-audit.json" << EOF
{
  "total_modules": $MODULE_COUNT,
  "module_sources": {
    "registry": $REGISTRY_MODULES,
    "local": $LOCAL_MODULES,
    "unpinned_git": $UNPINNED
  },
  "providers_locked": $PROVIDER_COUNT,
  "security_findings": {
    "hardcoded_secrets": $SECRET_FINDINGS,
    "public_access_patterns": $PUBLIC_ACCESS
  },
  "recommendations": [],
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>MODULE_AUDIT_COMPLETE</promise>`
