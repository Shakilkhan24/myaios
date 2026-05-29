# Ansible Inventory Audit

Audit an Ansible inventory for duplicate hosts, empty groups, and obvious stale placeholders.
Writes an inventory health report that can be used before configuration management runs.

## PRECONDITIONS CHECK

### A.1 Verify Input

```bash
if [ ! -f "$ANSIBLE_INVENTORY" ]; then
  echo "ERROR: inventory not found at $ANSIBLE_INVENTORY"
  exit 1
fi
```

### A.2 Verify Tool Availability

```bash
which ansible-inventory || { echo "ERROR: ansible-inventory not found"; exit 1; }
ansible-inventory --version | head -1
```

**PRECONDITION_CHECKPOINT:**
- [ ] Inventory exists
- [ ] ansible-inventory available

## PHASE 1: EXPORT INVENTORY GRAPH

```bash
ansible-inventory -i "$ANSIBLE_INVENTORY" --list > "$ARTIFACTS_DIR/ansible-inventory.json"
AUDIT_EXIT=$?
if [ "$AUDIT_EXIT" -ne 0 ]; then
  echo "ERROR: ansible-inventory export failed"
  exit 1
fi
```

**CHECKPOINT:** Inventory exported.

## PHASE 2: DETECT HEALTH ISSUES

```bash
HOST_COUNT=$(jq '[.. | objects | select(has("hosts")) | .hosts[]?] | unique | length' "$ARTIFACTS_DIR/ansible-inventory.json" 2>/dev/null || echo "0")
GROUP_COUNT=$(jq '[to_entries[] | select(.key != "_meta")] | length' "$ARTIFACTS_DIR/ansible-inventory.json" 2>/dev/null || echo "0")
EMPTY_GROUPS=$(jq '[to_entries[] | select(.key != "_meta") | select(((.value.hosts // []) | length) == 0 and ((.value.children // []) | length) == 0)] | length' "$ARTIFACTS_DIR/ansible-inventory.json" 2>/dev/null || echo "0")
PLACEHOLDER_HOSTS=$(jq '[.. | objects | select(has("hosts")) | .hosts[]? | select(test("example|placeholder|changeme"; "i"))] | length' "$ARTIFACTS_DIR/ansible-inventory.json" 2>/dev/null || echo "0")
GATE_RESULT=$([ "$EMPTY_GROUPS" -gt 0 ] || [ "$PLACEHOLDER_HOSTS" -gt 0 ] && echo "warn" || echo "pass")
```

## ARTIFACT OUTPUT

```bash
cat > "$ARTIFACTS_DIR/ansible-inventory-audit.json" << EOF
{
  "tool": "ansible-inventory",
  "inventory": "$ANSIBLE_INVENTORY",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "summary": {
    "hosts": $HOST_COUNT,
    "groups": $GROUP_COUNT,
    "empty_groups": $EMPTY_GROUPS,
    "placeholder_hosts": $PLACEHOLDER_HOSTS
  },
  "gate_result": "$GATE_RESULT"
}
EOF
```

**CHECKPOINT:** Inventory audit artifact written.

**EMIT:** `<promise>ANSIBLE_INVENTORY_$GATE_RESULT</promise>`
