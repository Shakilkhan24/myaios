# Ansible Lint

Run ansible-lint against the playbooks directory and normalize the result into a machine-readable
finding report.

## PRECONDITIONS CHECK

### A.1 Verify Tool Availability

```bash
which ansible-lint || { echo "ERROR: ansible-lint not found"; exit 1; }
ansible-lint --version | head -1
```

### A.2 Verify Playbooks Directory

```bash
if [ ! -d "$ANSIBLE_PLAYBOOKS_DIR" ]; then
  echo "ERROR: playbooks directory not found at $ANSIBLE_PLAYBOOKS_DIR"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] ansible-lint available
- [ ] Playbooks directory exists

## PHASE 1: RUN LINTER

```bash
ansible-lint \
  --format json \
  "$ANSIBLE_PLAYBOOKS_DIR" \
  > "$ARTIFACTS_DIR/ansible-lint-raw.json" \
  2> "$ARTIFACTS_DIR/ansible-lint.log"

LINT_EXIT=$?
```

**CHECKPOINT:** Linter execution complete.

## PHASE 2: NORMALIZE FINDINGS

```bash
TOTAL=$(jq 'length' "$ARTIFACTS_DIR/ansible-lint-raw.json" 2>/dev/null || echo "0")
CRITICAL=$(jq '[.[] | select((.severity // "high") == "critical")] | length' "$ARTIFACTS_DIR/ansible-lint-raw.json" 2>/dev/null || echo "0")
HIGH=$(jq '[.[] | select((.severity // "high") == "high")] | length' "$ARTIFACTS_DIR/ansible-lint-raw.json" 2>/dev/null || echo "0")
MEDIUM=$(jq '[.[] | select((.severity // "medium") == "medium")] | length' "$ARTIFACTS_DIR/ansible-lint-raw.json" 2>/dev/null || echo "0")
GATE_RESULT=$([ "$LINT_EXIT" -eq 0 ] && echo "pass" || echo "fail")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/ansible-lint-findings.json" << EOF
{
  "tool": "ansible-lint",
  "version": "$(ansible-lint --version | head -1)",
  "scan_target": "$ANSIBLE_PLAYBOOKS_DIR",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "summary": {
    "total": $TOTAL,
    "critical": $CRITICAL,
    "high": $HIGH,
    "medium": $MEDIUM,
    "low": 0,
    "info": 0
  },
  "findings": [],
  "gate_result": "$GATE_RESULT",
  "block_threshold": "HIGH"
}
EOF
```

**CHECKPOINT:** Lint artifact written.

**EMIT:** `<promise>ANSIBLE_LINT_$GATE_RESULT</promise>`
