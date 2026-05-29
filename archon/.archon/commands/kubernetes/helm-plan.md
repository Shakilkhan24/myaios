# Helm Plan (Diff)

Run `helm diff upgrade` to preview changes a Helm upgrade would make. Produces a structured diff artifact for review before the actual upgrade.

## PRECONDITIONS CHECK

```bash
which helm || { echo "ERROR: helm not found"; exit 1; }
which kubectl || { echo "ERROR: kubectl not found"; exit 1; }
helm version | head -1

# Verify cluster connectivity
kubectl cluster-info > /dev/null 2>&1 || { echo "ERROR: Cannot connect to Kubernetes cluster"; exit 1; }
```

**PRECONDITION_CHECKPOINT:**
- [ ] helm binary exists
- [ ] kubectl binary exists
- [ ] Cluster is reachable

## PHASE 1: VERIFY HELM DIFF PLUGIN

```bash
# Check if helm-diff plugin is installed
helm plugin list | grep -q diff || {
  echo "WARNING: helm-diff plugin not installed. Installing..."
  helm plugin install https://github.com/databus23/helm-diff 2>/dev/null || {
    echo "WARNING: Could not install helm-diff. Will use template-based diff."
  }
}
```

## PHASE 2: GENERATE DIFF

```bash
# Run helm diff
helm diff upgrade "$HELM_RELEASE" "$HELM_CHART" \
  --namespace "$HELM_NAMESPACE" \
  --no-color \
  2>&1 | tee "$ARTIFACTS_DIR/helm-diff.txt"

DIFF_EXIT=$?
DIFF_LINES=$(wc -l < "$ARTIFACTS_DIR/helm-diff.txt" 2>/dev/null || echo "0")

if [ $DIFF_EXIT -ne 0 ]; then
  echo "WARNING: helm diff failed — this may be a new release"
fi
```

**CHECKPOINT:** Diff generated.

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/helm-plan.json" << EOF
{
  "release": "$HELM_RELEASE",
  "chart": "$HELM_CHART",
  "namespace": "$HELM_NAMESPACE",
  "diff_lines": $DIFF_LINES,
  "has_changes": $([ "$DIFF_LINES" -gt "0" ] && echo "true" || echo "false"),
  "is_new_release": $(helm list -n "$HELM_NAMESPACE" --filter "$HELM_RELEASE" -q 2>/dev/null | grep -q "$HELM_RELEASE" && echo "false" || echo "true"),
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**EMIT:** `<promise>HELM_PLAN_COMPLETE</promise>`
