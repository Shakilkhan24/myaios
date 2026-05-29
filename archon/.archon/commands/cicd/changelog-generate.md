# Changelog Generate

Generate release notes from git history since the latest tag. Produces markdown and JSON artifacts
for release workflows and announcements.

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

## PHASE 1: COLLECT CHANGESET

```bash
CURRENT_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
if [ -n "$CURRENT_TAG" ]; then
  git log --pretty=format:"- %s (%h)" "${CURRENT_TAG}..HEAD" > "$ARTIFACTS_DIR/changelog.md"
else
  git log --pretty=format:"- %s (%h)" -20 > "$ARTIFACTS_DIR/changelog.md"
fi
```

**CHECKPOINT:** Changelog markdown generated.

## PHASE 2: SUMMARIZE COUNTS

```bash
TOTAL=$(wc -l < "$ARTIFACTS_DIR/changelog.md" | tr -d ' ')
FEATURES=$(grep -c "^- feat" "$ARTIFACTS_DIR/changelog.md" 2>/dev/null || echo "0")
FIXES=$(grep -c "^- fix" "$ARTIFACTS_DIR/changelog.md" 2>/dev/null || echo "0")
DOCS=$(grep -c "^- docs" "$ARTIFACTS_DIR/changelog.md" 2>/dev/null || echo "0")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/changelog.json" << EOF
{
  "base_tag": "${CURRENT_TAG:-unreleased}",
  "entries": $TOTAL,
  "summary": {
    "features": $FEATURES,
    "fixes": $FIXES,
    "docs": $DOCS
  },
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Changelog artifacts written.

**EMIT:** `<promise>CHANGELOG_READY</promise>`
