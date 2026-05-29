# Prometheus Query

Run a PromQL query against the configured Prometheus endpoint and write the result to a
time-series artifact for downstream alert analysis or reporting.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: PromQL query required in ARGUMENTS"
  exit 1
fi
```

### A.2 Verify Prometheus Access

```bash
if [ -z "$PROMETHEUS_URL" ]; then
  echo "ERROR: PROMETHEUS_URL not configured"
  exit 1
fi

curl -sf "$PROMETHEUS_URL/api/v1/status/buildinfo" > "$ARTIFACTS_DIR/prometheus-buildinfo.json"
```

**PRECONDITION_CHECKPOINT:**
- [ ] PromQL query provided
- [ ] Prometheus reachable

## PHASE 1: EXECUTE QUERY

```bash
QUERY=$(python -c "import urllib.parse,sys; print(urllib.parse.quote(sys.argv[1]))" "$ARGUMENTS")
curl -sf "$PROMETHEUS_URL/api/v1/query?query=$QUERY" > "$ARTIFACTS_DIR/metrics-query.json"
QUERY_EXIT=$?
if [ "$QUERY_EXIT" -ne 0 ]; then
  echo "ERROR: Prometheus query failed"
  exit 1
fi
```

**CHECKPOINT:** Query response captured.

## PHASE 2: SUMMARIZE RESULTS

```bash
RESULT_COUNT=$(jq '.data.result | length' "$ARTIFACTS_DIR/metrics-query.json" 2>/dev/null || echo "0")
cat > "$ARTIFACTS_DIR/prometheus-query-result.json" << EOF
{
  "query": "$ARGUMENTS",
  "result_series": $RESULT_COUNT,
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Query summary written.

**EMIT:** `<promise>PROMETHEUS_QUERY_COMPLETE</promise>`
