# Cosign Sign

Sign a container image using Cosign for supply chain security. Writes attestation artifact.

## PRECONDITIONS CHECK

```bash
which cosign || { echo "ERROR: cosign not found"; exit 1; }
cosign version 2>/dev/null | head -1
```

## PHASE 1: SIGN IMAGE

```bash
IMAGE_FULL="${IMAGE_REPO}:${IMAGE_TAG}"
# Get digest for signing (Rule S7 — sign by digest, not tag)
IMAGE_DIGEST=$(docker inspect --format='{{index .RepoDigests 0}}' "$IMAGE_FULL" 2>/dev/null)
if [ -z "$IMAGE_DIGEST" ]; then
  echo "ERROR: Cannot get image digest. Push image first."
  exit 1
fi

set +x  # Never echo signing keys
cosign sign --yes "$IMAGE_DIGEST" 2>&1 | tee "$ARTIFACTS_DIR/cosign-sign.log"
SIGN_EXIT=$?
set -x

OUTCOME=$([ $SIGN_EXIT -eq 0 ] && echo "success" || echo "failed")
```

## PHASE 2: VERIFY SIGNATURE

```bash
cosign verify "$IMAGE_DIGEST" 2>&1 | tee "$ARTIFACTS_DIR/cosign-verify.log"
VERIFY_EXIT=$?
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/image-signature.json" << EOF
{
  "image": "$IMAGE_FULL",
  "digest": "$IMAGE_DIGEST",
  "signed": $([ "$OUTCOME" = "success" ] && echo "true" || echo "false"),
  "verified": $([ "$VERIFY_EXIT" -eq "0" ] && echo "true" || echo "false"),
  "tool": "cosign",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>SIGN_$OUTCOME</promise>`
