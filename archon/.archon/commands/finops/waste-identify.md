# Waste Identify

Identify obvious infrastructure waste such as idle compute, orphaned storage, or oversized
allocations so cost optimization workflows can focus on the highest-confidence savings.

## PRECONDITIONS CHECK

### A.1 Validate Scope

```bash
SCOPE=${ARGUMENTS:-$ENVIRONMENT}
if [ -z "$SCOPE" ]; then
  echo "ERROR: waste analysis scope required"
  exit 1
fi
```

**PRECONDITION_CHECKPOINT:**
- [ ] Waste analysis scope resolved

## PHASE 1: INVENTORY WASTE CANDIDATES

Write:
- `$ARTIFACTS_DIR/waste-candidates.md`
- `$ARTIFACTS_DIR/waste-candidates.json`

The output must classify likely waste into:
1. idle resources
2. unattached or orphaned resources
3. obviously oversized resources
4. items that need human validation before action

**CHECKPOINT:** Waste candidate artifacts written.

**EMIT:** `<promise>WASTE_CANDIDATES_READY</promise>`
