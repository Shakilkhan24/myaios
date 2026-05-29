# Loki Search

Run a LogQL query against Loki and write the resulting log excerpts to machine-readable and
human-readable artifacts for downstream investigation.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: LogQL query required in ARGUMENTS"
  exit 1
fi
```

### A.2 Verify Loki Access

```bash
if [ -z "$LOKI_URL" ]; then
  echo "ERROR: LOKI_URL not configured"
  exit 1
fi

curl -sf "$LOKI_URL/ready" > "$ARTIFACTS_DIR/loki-ready.txt"
```

**PRECONDITION_CHECKPOINT:**
- [ ] LogQL query provided
- [ ] Loki reachable

## PHASE 1: EXECUTE SEARCH

```bash
QUERY=$(python -c "import urllib.parse,sys; print(urllib.parse.quote(sys.argv[1]))" "$ARGUMENTS")
curl -sf "$LOKI_URL/loki/api/v1/query_range?query=$QUERY&limit=200" > "$ARTIFACTS_DIR/loki-search.json"
SEARCH_EXIT=$?
if [ "$SEARCH_EXIT" -ne 0 ]; then
  echo "ERROR: Loki query failed"
  exit 1
fi
```

**CHECKPOINT:** Loki response captured.

## PHASE 2: NORMALIZE OUTPUT

```bash
MATCH_COUNT=$(jq '.data.result | length' "$ARTIFACTS_DIR/loki-search.json" 2>/dev/null || echo "0")
jq -r '.data.result[]?.values[]?[1]' "$ARTIFACTS_DIR/loki-search.json" > "$ARTIFACTS_DIR/loki-excerpts.txt" 2>/dev/null || echo "" > "$ARTIFACTS_DIR/loki-excerpts.txt"

cat > "$ARTIFACTS_DIR/loki-search-result.json" << EOF
{
  "query": "$ARGUMENTS",
  "streams": $MATCH_COUNT,
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Loki artifacts written.

**EMIT:** `<promise>LOKI_SEARCH_COMPLETE</promise>`
