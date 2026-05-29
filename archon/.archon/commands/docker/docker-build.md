# Docker Build

Build a container image using Docker with multi-stage build support. Records image metadata artifact.

## PRECONDITIONS CHECK

```bash
which docker || { echo "ERROR: docker not found"; exit 1; }
docker --version
```

## PHASE 1: BUILD IMAGE

```bash
START_TIME=$(date +%s)
IMAGE_TAG="${IMAGE_TAG:-$(git rev-parse --short HEAD 2>/dev/null || echo 'latest')}"
IMAGE_FULL="$IMAGE_REPO:$IMAGE_TAG"

docker build \
  -t "$IMAGE_FULL" \
  --label "org.opencontainers.image.revision=$(git rev-parse HEAD 2>/dev/null || echo 'unknown')" \
  --label "org.opencontainers.image.created=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  -f "${DOCKERFILE:-Dockerfile}" \
  . \
  2>&1 | tee "$ARTIFACTS_DIR/docker-build.log"

BUILD_EXIT=$?
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
OUTCOME=$([ $BUILD_EXIT -eq 0 ] && echo "success" || echo "failed")

IMAGE_SIZE=$(docker image inspect "$IMAGE_FULL" --format='{{.Size}}' 2>/dev/null || echo "0")
IMAGE_ID=$(docker image inspect "$IMAGE_FULL" --format='{{.Id}}' 2>/dev/null || echo "unknown")
```

**CHECKPOINT:** Build complete.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/docker-build.json" << EOF
{
  "image": "$IMAGE_FULL",
  "image_id": "$IMAGE_ID",
  "size_bytes": $IMAGE_SIZE,
  "dockerfile": "${DOCKERFILE:-Dockerfile}",
  "outcome": "$OUTCOME",
  "duration_seconds": $DURATION,
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>BUILD_$OUTCOME</promise>`
