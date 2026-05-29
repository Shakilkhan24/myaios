# kubectl Diff

Run `kubectl diff` to show what would change if a manifest were applied. Produces a structured diff artifact.

## PHASE 1: GENERATE DIFF

```bash
which kubectl || { echo "ERROR: kubectl not found"; exit 1; }

kubectl diff \
  -f "$ARGUMENTS" \
  -n "$K8S_NAMESPACE" \
  2>&1 | tee "$ARTIFACTS_DIR/kubectl-diff.txt"

DIFF_EXIT=$?
# kubectl diff exit codes: 0=no diff, 1=diff exists, >1=error
DIFF_LINES=$(wc -l < "$ARTIFACTS_DIR/kubectl-diff.txt" 2>/dev/null || echo "0")

if [ $DIFF_EXIT -gt 1 ]; then
  echo "ERROR: kubectl diff failed"
fi
```

**CHECKPOINT:** Diff generated.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/kubectl-diff-result.json" << EOF
{
  "source": "$ARGUMENTS",
  "namespace": "$K8S_NAMESPACE",
  "has_changes": $([ "$DIFF_EXIT" -eq "1" ] && echo "true" || echo "false"),
  "diff_lines": $DIFF_LINES,
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>KUBECTL_DIFF_COMPLETE</promise>`
