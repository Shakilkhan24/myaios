# Semver Bump

Determine the next semantic version from recent git history. Produces both a version artifact
and a rationale for release workflows.

## PRECONDITIONS CHECK

### A.1 Verify Git Context

```bash
git rev-parse --is-inside-work-tree >/dev/null 2>&1 || {
  echo "ERROR: not inside a git repository"
  exit 1
}
```

**PRECONDITION_CHECKPOINT:**
- [ ] Running inside git repository

## PHASE 1: READ CURRENT VERSION

```bash
CURRENT_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
COMMITS=$(git log --pretty=%s "${CURRENT_TAG}..HEAD" 2>/dev/null || git log --pretty=%s)
```

**CHECKPOINT:** Version baseline resolved.

## PHASE 2: DETERMINE BUMP LEVEL

```bash
if echo "$COMMITS" | grep -qE "BREAKING CHANGE|!:"; then
  BUMP_LEVEL="major"
elif echo "$COMMITS" | grep -qE "^feat(\(|:)"; then
  BUMP_LEVEL="minor"
else
  BUMP_LEVEL="patch"
fi

VERSION=${CURRENT_TAG#v}
MAJOR=$(echo "$VERSION" | cut -d. -f1)
MINOR=$(echo "$VERSION" | cut -d. -f2)
PATCH=$(echo "$VERSION" | cut -d. -f3)

case "$BUMP_LEVEL" in
  major) NEXT_VERSION="v$((MAJOR + 1)).0.0" ;;
  minor) NEXT_VERSION="v${MAJOR}.$((MINOR + 1)).0" ;;
  patch) NEXT_VERSION="v${MAJOR}.${MINOR}.$((PATCH + 1))" ;;
esac
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/semver-bump.json" << EOF
{
  "current_version": "$CURRENT_TAG",
  "next_version": "$NEXT_VERSION",
  "bump_level": "$BUMP_LEVEL",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Version artifact written.

**EMIT:** `<promise>SEMVER_$BUMP_LEVEL</promise>`
