# SBOM Generate

Generate Software Bill of Materials using Syft in CycloneDX format.

## PHASE 1: GENERATE SBOM

```bash
if which syft > /dev/null 2>&1; then
  SYFT_VERSION=$(syft version 2>/dev/null | grep -oP '\d+\.\d+\.\d+' | head -1)
  
  # SBOM for container image if available
  if [ -n "$IMAGE_REPO" ] && [ -n "$IMAGE_TAG" ]; then
    syft "$IMAGE_REPO:$IMAGE_TAG" -o cyclonedx-json > "$ARTIFACTS_DIR/sbom.json" 2>&1
  else
    # SBOM for source directory
    syft dir:. -o cyclonedx-json > "$ARTIFACTS_DIR/sbom.json" 2>&1
  fi
  
  COMPONENT_COUNT=$(jq '.components | length' "$ARTIFACTS_DIR/sbom.json" 2>/dev/null || echo "0")
else
  echo "WARNING: syft not found — SBOM generation skipped"
  COMPONENT_COUNT=0
fi

cat > "$ARTIFACTS_DIR/sbom-metadata.json" << EOF
{"tool":"syft","format":"cyclonedx","components":$COMPONENT_COUNT,"timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)"}
EOF
```

**EMIT:** `<promise>SBOM_GENERATED</promise>`
