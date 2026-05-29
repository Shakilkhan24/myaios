# DB Performance Analyze

Analyze database performance signals such as slow queries, migration impact, and expensive
query shapes to produce a focused tuning report.

## PRECONDITIONS CHECK

### A.1 Validate Input

```bash
TARGET=${ARGUMENTS:-slow-query-log}
echo "$TARGET" > "$ARTIFACTS_DIR/db-performance-target.txt"
```

**PRECONDITION_CHECKPOINT:**
- [ ] Analysis target resolved

## PHASE 1: GATHER PERFORMANCE INPUTS

Write a source inventory to `$ARTIFACTS_DIR/db-performance-inputs.json` describing:
1. target provided in ARGUMENTS
2. whether slow query logs are available
3. whether migration logs are available
4. whether EXPLAIN plans still need to be collected manually

**CHECKPOINT:** Input inventory written.

## PHASE 2: PRODUCE ANALYSIS

Write:
- `$ARTIFACTS_DIR/db-performance-analysis.md`
- `$ARTIFACTS_DIR/db-performance-analysis.json`

Include:
1. likely bottlenecks
2. queries or operations that need EXPLAIN analysis
3. recommended indexes or schema changes
4. whether the issue looks application-side, database-side, or migration-side

**CHECKPOINT:** Performance analysis artifacts written.

**EMIT:** `<promise>DB_PERFORMANCE_ANALYZED</promise>`
