# Grafana Dashboard Deploy

Deploy a Grafana dashboard definition through the Grafana HTTP API and record the resulting
UID and version metadata in an artifact for rollback and audit use.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: dashboard JSON path required in ARGUMENTS"
  exit 1
fi

if [ ! -f "$ARGUMENTS" ]; then
  echo "ERROR: dashboard file not found at $ARGUMENTS"
  exit 1
fi
```

### A.2 Verify Grafana Access

```bash
if [ -z "$GRAFANA_URL" ] || [ -z "$GRAFANA_TOKEN" ]; then
  echo "ERROR: GRAFANA_URL and GRAFANA_TOKEN are required"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Dashboard file exists
- [ ] Grafana credentials configured

## PHASE 1: DEPLOY DASHBOARD

```bash
set +x
curl -sf \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  "$GRAFANA_URL/api/dashboards/db" \
  --data @"$ARGUMENTS" \
  > "$ARTIFACTS_DIR/grafana-dashboard-response.json"
set -x
```

**CHECKPOINT:** Dashboard deployment request completed.

## PHASE 2: WRITE DEPLOY ARTIFACT

```bash
UID=$(jq -r '.uid // ""' "$ARTIFACTS_DIR/grafana-dashboard-response.json" 2>/dev/null)
STATUS=$(jq -r '.status // "unknown"' "$ARTIFACTS_DIR/grafana-dashboard-response.json" 2>/dev/null)

cat > "$ARTIFACTS_DIR/grafana-dashboard-deploy.json" << EOF
{
  "dashboard_file": "$ARGUMENTS",
  "uid": "$UID",
  "status": "$STATUS",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Deployment artifact written.

**EMIT:** `<promise>GRAFANA_DASHBOARD_DEPLOYED</promise>`
