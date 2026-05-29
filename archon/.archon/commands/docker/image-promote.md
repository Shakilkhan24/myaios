# Image Promote

Promote a container image tag across registries (e.g., dev registry → prod registry). Retags and pushes with verification.

## PHASE 1: PULL SOURCE IMAGE

```bash
SOURCE_IMAGE="${SOURCE_REGISTRY}/${IMAGE_REPO}:${IMAGE_TAG}"
TARGET_IMAGE="${TARGET_REGISTRY}/${IMAGE_REPO}:${IMAGE_TAG}"

docker pull "$SOURCE_IMAGE" 2>&1 | tee "$ARTIFACTS_DIR/image-pull.log"
if [ $? -ne 0 ]; then echo "ERROR: Failed to pull $SOURCE_IMAGE"; exit 1; fi
```

## PHASE 2: RETAG AND PUSH

```bash
docker tag "$SOURCE_IMAGE" "$TARGET_IMAGE"
docker push "$TARGET_IMAGE" 2>&1 | tee "$ARTIFACTS_DIR/image-push.log"
PUSH_EXIT=$?

TARGET_DIGEST=$(docker inspect --format='{{index .RepoDigests 0}}' "$TARGET_IMAGE" 2>/dev/null || echo "unknown")

cat > "$ARTIFACTS_DIR/image-promote.json" << EOF
{
  "source": "$SOURCE_IMAGE",
  "target": "$TARGET_IMAGE",
  "target_digest": "$TARGET_DIGEST",
  "outcome": "$([ $PUSH_EXIT -eq 0 ] && echo "success" || echo "failed")",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>PROMOTE_COMPLETE</promise>`
