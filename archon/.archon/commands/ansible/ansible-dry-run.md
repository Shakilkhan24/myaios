# Ansible Dry Run

Run an Ansible playbook in check and diff mode. Produces a dry-run artifact that downstream
approval and impact-analysis steps can review before any write operation.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: playbook path is required in ARGUMENTS"
  exit 1
fi

PLAYBOOK_PATH="$ANSIBLE_PLAYBOOKS_DIR/$ARGUMENTS"
if [ ! -f "$PLAYBOOK_PATH" ]; then
  echo "ERROR: playbook not found at $PLAYBOOK_PATH"
  exit 1
fi

if [ ! -f "$ANSIBLE_INVENTORY" ]; then
  echo "ERROR: inventory not found at $ANSIBLE_INVENTORY"
  exit 1
fi
```

### A.2 Verify Tool Availability

```bash
which ansible-playbook || { echo "ERROR: ansible-playbook not found"; exit 1; }
ansible-playbook --version | head -1
```

**PRECONDITION_CHECKPOINT:**
- [ ] Playbook path resolved
- [ ] Inventory file exists
- [ ] ansible-playbook available

## PHASE 1: EXECUTE CHECK MODE

```bash
PLAYBOOK_PATH="$ANSIBLE_PLAYBOOKS_DIR/$ARGUMENTS"
START_TIME=$(date +%s)

ansible-playbook \
  --check \
  --diff \
  -i "$ANSIBLE_INVENTORY" \
  "$PLAYBOOK_PATH" \
  $([ "$ANSIBLE_BECOME" = "true" ] && echo "--become") \
  2>&1 | tee "$ARTIFACTS_DIR/ansible-dry-run.txt"

RUN_EXIT=${PIPESTATUS[0]}
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
```

**CHECKPOINT:** Dry run completed.

## PHASE 2: SUMMARIZE DIFF

```bash
CHANGED_LINES=$(grep -c "^changed:" "$ARTIFACTS_DIR/ansible-dry-run.txt" 2>/dev/null || echo "0")
FAILED_LINES=$(grep -c "^fatal:" "$ARTIFACTS_DIR/ansible-dry-run.txt" 2>/dev/null || echo "0")
OUTCOME=$([ "$RUN_EXIT" -eq 0 ] && echo "success" || echo "failed")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/ansible-dry-run.json" << EOF
{
  "tool": "ansible-playbook",
  "operation": "dry-run",
  "playbook": "$ARGUMENTS",
  "inventory": "$ANSIBLE_INVENTORY",
  "environment": "$ENVIRONMENT",
  "status": "$OUTCOME",
  "summary": {
    "changed_lines": $CHANGED_LINES,
    "fatal_lines": $FAILED_LINES
  },
  "duration_seconds": $DURATION,
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Dry-run artifacts written.

**EMIT:** `<promise>ANSIBLE_DRY_RUN_$OUTCOME</promise>`
