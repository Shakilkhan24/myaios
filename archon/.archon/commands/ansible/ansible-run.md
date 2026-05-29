# Ansible Run

Execute an Ansible playbook against the configured inventory and write a deployment artifact
for downstream verification and audit trails.

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

## PHASE 1: EXECUTE PLAYBOOK

```bash
PLAYBOOK_PATH="$ANSIBLE_PLAYBOOKS_DIR/$ARGUMENTS"
START_TIME=$(date +%s)

ansible-playbook \
  -i "$ANSIBLE_INVENTORY" \
  "$PLAYBOOK_PATH" \
  $([ "$ANSIBLE_BECOME" = "true" ] && echo "--become") \
  2>&1 | tee "$ARTIFACTS_DIR/ansible-run.log"

RUN_EXIT=${PIPESTATUS[0]}
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
OUTCOME=$([ "$RUN_EXIT" -eq 0 ] && echo "success" || echo "failed")
DEPLOYMENT_ID="ansible-$(date +%Y%m%d%H%M%S)"
```

**CHECKPOINT:** Playbook execution complete.

## PHASE 2: AUDIT OUTPUT

```bash
CHANGED_LINES=$(grep -c "^changed:" "$ARTIFACTS_DIR/ansible-run.log" 2>/dev/null || echo "0")
FAILED_LINES=$(grep -c "^fatal:" "$ARTIFACTS_DIR/ansible-run.log" 2>/dev/null || echo "0")
UNREACHABLE_LINES=$(grep -c "UNREACHABLE!" "$ARTIFACTS_DIR/ansible-run.log" 2>/dev/null || echo "0")
```

## ARTIFACT OUTPUT

```bash
mkdir -p "$ARTIFACTS_DIR/deploys"

cat > "$ARTIFACTS_DIR/deploys/$DEPLOYMENT_ID.json" << EOF
{
  "deployment_id": "$DEPLOYMENT_ID",
  "workflow_id": "$WORKFLOW_ID",
  "environment": "$ENVIRONMENT",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "tool": "ansible",
  "resource_identifier": "$ARGUMENTS",
  "version": "$BASE_BRANCH",
  "outcome": "$OUTCOME",
  "duration_seconds": $DURATION,
  "change_summary": "Ansible playbook execution for $ARGUMENTS",
  "rollback_command": "ansible-playbook -i $ANSIBLE_INVENTORY $PLAYBOOK_PATH --tags rollback",
  "rollback_version": "manual",
  "change_url": "",
  "deployed_by": "archon/$WORKFLOW_ID",
  "health_check_url": ""
}
EOF

cat > "$ARTIFACTS_DIR/ansible-run-result.json" << EOF
{
  "tool": "ansible-playbook",
  "operation": "apply",
  "playbook": "$ARGUMENTS",
  "inventory": "$ANSIBLE_INVENTORY",
  "environment": "$ENVIRONMENT",
  "status": "$OUTCOME",
  "summary": {
    "changed_lines": $CHANGED_LINES,
    "fatal_lines": $FAILED_LINES,
    "unreachable_lines": $UNREACHABLE_LINES
  },
  "deployment_id": "$DEPLOYMENT_ID",
  "duration_seconds": $DURATION,
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

**CHECKPOINT:** Execution artifacts written.

**EMIT:** `<promise>ANSIBLE_RUN_$OUTCOME</promise>`
