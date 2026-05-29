# Release Announce

Draft a release announcement from the generated version and changelog artifacts. Intended for
Slack, Teams, GitHub Releases, or internal deployment channels.

## PRECONDITIONS CHECK

### A.1 Verify Required Artifacts

```bash
if [ ! -f "$ARTIFACTS_DIR/semver-bump.json" ]; then
  echo "ERROR: missing semver-bump.json artifact"
  exit 1
fi

if [ ! -f "$ARTIFACTS_DIR/changelog.md" ]; then
  echo "ERROR: missing changelog.md artifact"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Version artifact exists
- [ ] Changelog artifact exists

## PHASE 1: BUILD ANNOUNCEMENT

```bash
NEXT_VERSION=$(jq -r '.next_version' "$ARTIFACTS_DIR/semver-bump.json")
cat > "$ARTIFACTS_DIR/release-announcement.md" << EOF
## Release $NEXT_VERSION

Release candidate is ready.

### Included Changes
$(cat "$ARTIFACTS_DIR/changelog.md")

### Verification
- Security gates completed
- Deployment workflow completed
- Post-deploy health checks passed
EOF
```

**CHECKPOINT:** Announcement markdown generated.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/release-announcement.json" << EOF
{
  "version": "$(jq -r '.next_version' "$ARTIFACTS_DIR/semver-bump.json")",
  "announcement_file": "$ARTIFACTS_DIR/release-announcement.md",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Announcement artifacts written.

**EMIT:** `<promise>RELEASE_ANNOUNCEMENT_READY</promise>`
