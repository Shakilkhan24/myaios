# Generate Runbook

Generates an operational runbook from incident patterns, service architecture, and historical incident data. Produces a structured, step-by-step runbook that can be executed by `runbook-executor`.

## PHASE 0: VALIDATE INPUT

```bash
# Validate arguments
if [ -z "$ARGUMENTS" ]; then
  echo "ERROR: Must provide service name or incident category"
  echo "Usage: generate-runbook <service-name|incident-category>"
  cat > "$ARTIFACTS_DIR/error.json" << EOF
{
  "error": "missing_arguments",
  "message": "Must provide service name or incident category",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
  exit 1
fi

# Validate argument format (alphanumeric, hyphens, underscores)
if ! echo "$ARGUMENTS" | grep -qE '^[a-zA-Z0-9_-]+$'; then
  echo "ERROR: Invalid argument format. Use alphanumeric characters, hyphens, or underscores."
  exit 1
fi
```

**CHECKPOINT:** Input validated.

## PHASE 1: GATHER SERVICE CONTEXT

Analyze the target service or incident category to understand:

1. **Service architecture**: What components does this service depend on? (databases, caches, message queues, APIs)
2. **Common failure modes**: What typically goes wrong? (OOM, connection pool exhaustion, certificate expiry, disk full)
3. **Health endpoints**: Where can we check health? (HTTP endpoints, TCP ports, process checks)
4. **Metrics to check**: What Prometheus/Grafana queries reveal the state?
5. **Log sources**: Where do logs live? (Loki queries, CloudWatch log groups, file paths)
6. **Escalation path**: Who owns this service? (on-call rotation, team Slack channel)

**CHECKPOINT:** Service context gathered.

## PHASE 2: GENERATE RUNBOOK STRUCTURE

Create a runbook with the following mandatory sections:

### Runbook Template Structure

```markdown
# Runbook: [Service/Category] — [Failure Mode]

## Metadata
- **Service:** $ARGUMENTS
- **Category:** [availability|performance|security|data]
- **Severity Trigger:** [P1|P2|P3|P4]
- **Last Updated:** [timestamp]
- **Owner:** [team]

## Symptoms
- [Observable symptom 1]
- [Observable symptom 2]

## Diagnostic Steps (execute in order)
### Step 1: [Check description]
```bash
[diagnostic command]
```
**Expected:** [what healthy looks like]
**If abnormal:** Go to Remediation Step X

### Step 2: [Check description]
...

## Remediation Steps
### Remediation 1: [Fix description]
```bash
[fix command]
```
**Verify:** [verification command]
**Rollback:** [rollback command if fix fails]

## Escalation
- **If unresolved after 15 min:** Page [team]
- **If customer-facing:** Notify [status page]
```

**CHECKPOINT:** Runbook structure generated.

## PHASE 3: WRITE ARTIFACTS

```bash
# Write the runbook as a markdown file
cat > "$ARTIFACTS_DIR/runbook-${ARGUMENTS}.md" << 'RUNBOOK_EOF'
[Generated runbook content goes here]
RUNBOOK_EOF

# Write machine-readable metadata
cat > "$ARTIFACTS_DIR/runbook-metadata.json" << EOF
{
  "runbook_id": "rb-${ARGUMENTS}-$(date +%Y%m%d%H%M%S)",
  "service": "$ARGUMENTS",
  "generated_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "sections": ["symptoms", "diagnostic", "remediation", "escalation"],
  "executable": true,
  "version": "1.0.0"
}
EOF
```

**CHECKPOINT:** Runbook and metadata artifacts written.

**EMIT:** `<promise>RUNBOOK_GENERATED</promise>`
