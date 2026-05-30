# README_SKILLS_TEMPLATES_WORKFLOWS.md
## The Execution Engine — Skills, Templates, Workflows, and the Fusion Layer

> **What this README is for:** The definitive builder's guide to MYAIOS's execution layer.
> After reading this, you should be able to build your first skill within 2 hours without
> asking any questions. The examples are real. The anti-patterns are real. The Fusion Layer
> is resolved — not gestured at.

---

## The Three Primitives and One Meta-Layer

```
SKILL       = atomic capability     "I can do one precise thing"
TEMPLATE    = reusable pattern      "Here is the shape of a recurring output"
WORKFLOW    = composed pipeline     "Here is how capabilities combine into a result"
CRONJOB     = scheduled trigger     "Here is when a workflow runs without being asked"
─────────────────────────────────────────────────────────────
FUSION LAYER = self-improvement     "Here is how the system gets better at all of the above"
```

These are not interchangeable. Using a workflow where a skill would do creates overhead and
fragility. Using a skill where a template would clarify creates inconsistency. The primitives
are distinct and the distinctions matter.

---

## SKILLS — Atomic Capabilities

### What a Skill Is (and Is Not)

A skill is a **self-describing, self-testing, versioned module of capability**. It is the
atomic unit of what the OS can do.

A skill is **NOT** a prompt template, a one-line command, a configuration file, or a
documentation page.

A skill **IS**: a named capability with defined trigger conditions, a structured system prompt,
supporting references, a testing protocol, version tracking, and explicit success metrics.

The critical constraint: **one skill, one goal.** A skill that achieves two goals is two
skills failing to be disciplined. Bigger is not better — a bloated skill burns tokens, widens
the hallucination surface, and produces unpredictable outputs.

### Skill Anatomy

Every skill lives in its own folder:

```
skills/
└── [skill-name]/
    ├── skill.md          ← REQUIRED: goal, trigger, system prompt, I/O spec, metrics
    ├── references/       ← API docs, example outputs, data the skill reads
    │   └── [ref].md
    ├── scripts/          ← helper scripts, if needed
    │   └── [helper].py
    ├── tests/            ← test inputs + expected outputs
    │   └── test-cases.md
    └── CHANGELOG.md      ← version history
```

Not all components are required. A simple query skill may only need `skill.md`. The rule:
**use only what the skill actually needs.**

### skill.md: Required Structure

```yaml
---
name: [skill-name]
version: "1.0.0"
goal: |
  [ONE sentence. What this skill produces. If it can't be said in one sentence,
   it is two skills.]
trigger_conditions:
  - [specific phrase or pattern that activates this skill]
  - [another trigger condition]
argument_hint: "[what input this skill expects]"
last_tested: "[date]"
---

# [Skill Name]

## System Prompt
[The LLM instruction block. Concise, deterministic. Not aspirational.]

## Input Format
[Exact structure of what this skill expects. Be prescriptive.]

## Output Format
[Exact structure of output. The caller depends on this shape.]

## Step-by-Step Protocol
### Step 1: [action]
[Precise instruction]
### Step 2: [action]
...

## Success Metrics
- [metric 1: e.g., "output contains all required fields"]
- [metric 2: e.g., "summary is ≤200 words"]
- [metric 3: e.g., "errors field is present even when empty"]

## Edge Cases
- [case]: [how to handle]
- [case]: [how to handle]

## Dependencies
- [mcp-name or other-skill-name]

## Output Metadata
Append to every output:
---
Skill: [name] | v[version] | [timestamp]
```

---

## The Skill Ontology

<!-- NEW → Full Skill Ontology taxonomy — neither prior model taxonomized skill types into
a structured classification; both listed examples but not a dependency-aware hierarchy -->

Skills are not a flat list. They form a dependency hierarchy where some skill types must
exist before others can function:

```
TIER 0: INFRASTRUCTURE SKILLS (must exist first)
  log-write-skill         ← writes to log.md + DB; called by every other skill
  db-query-skill          ← reads from DB; used by context and audit skills
  context-load-skill      ← assembles the 4 context types at session start
  mcp-test-skill          ← verifies all MCP connections are live

TIER 1: MCP SKILLS (one per MCP; depend on Tier 0)
  github-mcp-skill        ← ingest, query, write via GitHub MCP
  slack-mcp-skill         ← ingest, query, write via Slack MCP
  telegram-mcp-skill      ← send alerts via Telegram MCP
  google-workspace-skill  ← Gmail, Calendar, Drive
  filesystem-mcp-skill    ← local file read/write/search
  browser-mcp-skill       ← Playwright web automation
  [+ one per additional MCP]

TIER 2: COGNITIVE SKILLS (depend on Tier 0 + Tier 1)
  second-brain-ingest-skill   ← compiles raw/ sources into wiki/
  second-brain-query-skill    ← queries wiki/ for context injection
  context-update-skill        ← updates dynamic context mid-session
  context-judge-skill         ← verifies context accuracy and completeness

TIER 3: EXECUTION SKILLS (depend on Tier 0-2)
  [domain-specific skills: pr-review, voice-draft, code-analysis, etc.]
  daily-report-skill
  weekly-briefing-skill
  notifier-skill

TIER 4: META-SKILLS / SUBSTRATE (depend on all prior tiers; built last)
  skill-judge-skill       ← evaluates skill outputs against success metrics
  skill-update-skill      ← proposes changes to skill.md based on judge output
  pattern-learner-skill   ← finds behavioral patterns in log; proposes improvements
  audit-skill             ← compares filesystem against AUDIT.md spec
  gap-detector-skill      ← writes gap.md from accumulated audit findings
```

Build Tier 0 first. Do not build Tier 4 until Tier 1-3 skills have run for weeks and
produced enough log data to evaluate.

---

## Concrete Skill Example: `github-pr-review-skill`

This is the reference-quality skill example. Every skill you build should reach this level
of specification before being deployed.

```yaml
---
name: github-pr-review-skill
version: "1.2.0"
goal: |
  Review a GitHub pull request: summarize changes, assess risk,
  flag missing tests, and draft a review comment in the user's voice.
trigger_conditions:
  - message contains "review PR" or "review this PR"
  - message contains a GitHub PR URL
  - message matches pattern: "owner/repo#[number]"
  - message contains "analyze PR" or "check PR quality"
argument_hint: "<PR URL or owner/repo#number>"
last_tested: "2026-05-29"
---

# GitHub PR Review Skill

## System Prompt
You review GitHub pull requests with the depth and judgment the user would apply.
Read the diff completely. Do not skim. Assess each file for: change nature
(feature/fix/refactor/config), test coverage gaps, security implications, and
breaking change risk. Write in the user's voice — direct, specific, no softening.
Flag real issues. Acknowledge what is genuinely good.

## Input Format
  pr_url: string (GitHub PR URL or owner/repo#number)
  review_mode: "full" | "flags-only" | "draft-comment"

## Output Format
  summary: string (2-3 sentences: what does this PR do?)
  change_breakdown: [{file, change_type, notes}]
  risk_assessment:
    scope_risk: "high" | "medium" | "low"
    complexity_risk: "high" | "medium" | "low"
    test_coverage: "high" | "medium" | "low"
    breaking_change: boolean
  flags:
    critical: [string]    ← blockers
    warning: [string]     ← concerns
    good: [string]        ← what works well
  recommendations: [string]
  verdict: "APPROVE" | "REQUEST_CHANGES" | "COMMENT"
  review_comment_draft: string (formatted for GitHub)

## Step-by-Step Protocol

### Step 1: Fetch PR Metadata
Use github-mcp to retrieve: title, description, author, files changed,
number of comments, CI status, base branch vs head branch.
If CI is failing → lead the output with that. Do not continue the review
as if CI is green.

### Step 2: Fetch and Analyze Diff
For each changed file:
- Identify change nature: new feature, bug fix, refactor, config
- Flag files with behavior changes but no test changes
- Flag new dependencies, changed auth flows, new API routes
- Identify scope: how many other systems does this file touch?

### Step 3: Risk Assessment
Rate on all four risk dimensions. Default to pessimism on scope and
complexity. Default to precision on test coverage (no guessing).

### Step 4: Draft Review Comment
Write in user's voice. Format:
  ## Summary
  [2-3 sentences]
  ## Change Breakdown
  [file-by-file key changes]
  ## Flags
  🔴 [critical], 🟡 [warning], ✅ [good]
  ## Recommendations
  [specific, actionable, line-reference where possible]
  ## Verdict: [APPROVE / REQUEST_CHANGES / COMMENT]

### Step 5: Offer to Post
Always ask: "Want me to post this as a review comment on GitHub?"

## Success Metrics
- output contains all required output_format fields
- risk_assessment has all four dimensions
- review_comment_draft is formatted for direct GitHub use
- verdict is always one of the three valid values
- errors field present even when empty

## Edge Cases
- PR has 50+ changed files → review top 10 by scope; note truncation
- PR has no description → ask for context before reviewing
- CI is failing → lead with that; don't bury it in flags
- PR is a draft → note prominently; adjust verdict to COMMENT only
- PR modifies auth or credentials → escalate scope_risk to HIGH regardless

## Dependencies
- github-mcp

## Output Metadata
---
Skill: github-pr-review-skill | v1.2.0 | [timestamp]
```

---

## TEMPLATES — Reusable Patterns

### What a Template Is

A template is the **structural contract** for a recurring output. Not a prompt, not a skill —
the *shape* that a category of output must conform to. Where a skill is a unit of capability,
a template is a unit of pattern.

**Critical naming clarification:**
- `Fresh_SecondBrain/templates/` — wiki templates (Source, Concept, Entity pages etc.)
  These govern how the Second Brain is structured.
- `skills/templates/` or `.claude/templates/` — execution templates (report formats,
  message formats, artifact shapes). These govern execution outputs.
Both are templates; they live in different domains and serve different masters.

### Template Structure

```markdown
---
name: [template-name]
type: [wiki-page | report | notification | artifact | log-entry]
domain: [second-brain | execution]
generation_skill: [which skill populates this template]
version: "1.0.0"
---

# [Template Name]

## Schema
[Fields, types, constraints. Be prescriptive.]

## Example
[A realistic filled-out example. Not lorem ipsum.]

## Generation Notes
[What the generation skill must do to satisfy this schema]
```

---

## WORKFLOWS — Composed Intelligence Pipelines

### What a Workflow Is

A workflow is a **Workflow Definition** (a YAML artifact) that describes a directed
acyclic graph (DAG) of nodes, each executing a skill, a bash command, a loop, or requesting
human approval. A **Workflow Execution** is a running instance of that definition.

The reference implementation is in `archon/.archon/workflows/`. MYAIOS uses this pattern
directly.

### Workflow State Machine

<!-- NEW → Workflow State Machine — neither prior model addressed what states a workflow
moves through; this was called out in the synthesis prompt as missing -->

Every workflow execution moves through these states:

```
PENDING ──► TRIGGERED ──► CONTEXT_LOADED ──► EXECUTING ──► LOGGING ──► COMPLETE
                │                                  │              │
                │                           AWAITING_APPROVAL     │
                │                                  │              │
                │              ◄────────────────────              │
                │                                                  │
                └──────────────────────────────── FAILED ─────────┘
                                                      │
                                                   NOTIFIED
```

- **PENDING**: CronJob definition exists; not yet triggered
- **TRIGGERED**: Trigger fired (cron, event, or manual); workflow loading
- **CONTEXT_LOADED**: Context Manager assembled the 4 context types
- **EXECUTING**: Nodes running sequentially/in parallel per DAG
- **AWAITING_APPROVAL**: Node reached a human-approval gate; paused
- **LOGGING**: log-write-skill called; DB updated
- **COMPLETE**: All nodes finished; log confirmed written
- **FAILED**: Any node errored; error captured in log; NOTIFIED

A workflow that ends in COMPLETE without a log write has not completed. The log write is the
final step, always.

### Workflow YAML Structure

```yaml
name: [workflow-name]
description: |
  Use when: [trigger context]
  Goal: [what this produces at completion]
version: "1.0.0"
trigger:
  type: fixed | dynamic | manual | workflow-based
  schedule: "0 23 * * *"   # (if fixed) cron expression

provider: claude
model: sonnet               # default; override per node for haiku (cheap/fast)

nodes:
  - id: [node-id]
    model: haiku | sonnet   # per-node override
    prompt: |
      [LLM instruction. Reference prior outputs with ${{ [node-id].output }}]
    depends_on: [node-id-list]   # omit for first node
    output_var: [variable-name]

  - id: [bash-node-id]
    bash: |
      [Shell command]
    depends_on: [node-id]

  - id: [loop-node-id]
    loop:
      prompt: |
        [Instruction applied to each item]
        Current item: ${{ item }}
      items: "${{ prior-node.output_list }}"
      until: COMPLETE
      max_iterations: 50
      output_var: [results-list]

  - id: [approval-node-id]
    approval:
      message: "[What you're asking the human to review/approve]"
      capture_response: true
    depends_on: [node-id]

  # ALWAYS LAST:
  - id: log-write
    model: haiku
    prompt: |
      Call log-write-skill with:
        workflow_id: [workflow-name]
        stage: complete
        summary: "[one-sentence description of what just happened]"
        errors: null (or list errors if any node failed)
        db_refs: [list of DB record IDs created]
    depends_on: [last-functional-node]
```

---

## Complete Workflow Example: `nightly-wiki-ingest`

```yaml
name: nightly-wiki-ingest
description: |
  Use when: nightly cron fires (23:00) OR new files detected in raw/pending/
  Goal: Compile all pending raw sources into wiki/ following the Second Brain schema
version: "1.0.0"
trigger:
  type: fixed
  schedule: "0 23 * * *"
provider: claude

nodes:
  - id: scan-pending
    bash: |
      ls Fresh_SecondBrain/raw/pending/ 2>/dev/null | wc -l
    output_var: pending_count

  - id: check-pending
    model: haiku
    prompt: |
      pending_count: ${{ scan-pending.pending_count }}
      If count is 0: return {"status": "nothing_to_do", "files": []}
      If count > 0: list all filenames in Fresh_SecondBrain/raw/pending/
      Return: {"status": "files_found", "files": ["filename1.md", ...]}
    depends_on: [scan-pending]
    output_var: pending_check

  - id: ingest-loop
    loop:
      prompt: |
        Read Fresh_SecondBrain/CLAUDE.md first for the full schema.
        Then ingest the file: Fresh_SecondBrain/raw/pending/${{ item }}

        Steps (do not skip any):
        1. Read the file completely
        2. Create wiki/sources/[slug].md using the Source Template
        3. Update all relevant wiki/entities/ pages
        4. Update all relevant wiki/concepts/ pages
        5. Check for contradictions — flag with [!CONFLICT], never resolve silently
        6. If the big picture changed, update wiki/synthesis/overview.md
        7. Append to log.md (Second Brain log, not OS log)
        8. Update index.md
        9. Move file from raw/pending/ to appropriate raw/ subfolder

        Report: {file, pages_created, pages_updated, conflicts_flagged, errors}
      items: "${{ check-pending.files }}"
      until: COMPLETE
      max_iterations: 20
    depends_on: [check-pending]
    output_var: ingest_results

  - id: lint-wiki
    model: haiku
    prompt: |
      Run a quick lint on the Second Brain wiki/:
      - Check that index.md was updated (look for today's date)
      - Check that log.md was appended (look for today's date)
      - Check that no wiki/ page exceeds 2000 words (bloat warning)
      Report: {index_updated: bool, log_updated: bool, bloated_pages: []}
    depends_on: [ingest-loop]
    output_var: lint_result

  - id: notify-if-conflicts
    model: haiku
    prompt: |
      Review ingest results: ${{ ingest-loop.ingest_results }}
      If any conflicts_flagged > 0:
        Use telegram-mcp-skill to send:
        "Nightly ingest complete. [N] files processed. [M] contradictions flagged
         in wiki/synthesis/open-questions.md — review when ready."
      If no conflicts: skip notification.
    depends_on: [ingest-loop]

  - id: log-write
    model: haiku
    prompt: |
      Call log-write-skill with:
        workflow_id: nightly-wiki-ingest
        stage: complete
        summary: "Processed ${{ scan-pending.pending_count }} files into wiki/"
        errors: ${{ ingest-loop.ingest_results[errors] }}
        db_refs: []
    depends_on: [notify-if-conflicts, lint-wiki]
```

---

## CRONJOBS — The Heartbeat

CronJobs are what make MYAIOS proactive rather than reactive. They are scheduled triggers,
not skills. They invoke workflows; they do not contain execution logic themselves.

### Trigger Types

| Type | Description | When to Use |
|---|---|---|
| `fixed` | Runs at a set time every day/week | Nightly ingest, morning briefing, weekly review |
| `dynamic` | Runs when a condition is met | New files in pending/, new MCP event |
| `manual` | Triggered explicitly | One-off workflows, testing |
| `workflow-based` | Triggered by another workflow's output | Cascade workflows, conditional branches |

### MVP CronJob Definitions

These 5 must exist before the system can be called operational:

| CronJob | Schedule | Triggers |
|---|---|---|
| `nightly-wiki-ingest` | `0 23 * * *` | nightly-wiki-ingest workflow |
| `morning-briefing` | `0 8 * * *` | morning-briefing workflow |
| `weekly-pr-review` | `0 9 * * MON` | weekly-pr-review workflow |
| `mcp-health-check` | `0 */6 * * *` | mcp-test-skill for all MCPs |
| `weekly-sb-lint` | `0 22 * * SUN` | second-brain-lint workflow |

---

## THE FUSION LAYER — Resolved

<!-- NEW → Fusion Layer fully resolved with concrete data contracts between judge and update
skills — the synthesis prompt required this; neither prior version made it buildable -->

The `fusion-skill-template-workflow.idea` file is blank in the repository. This is the
most important architectural gap, and it is resolved here.

**The Fusion Layer is the self-improvement loop.** It connects the OS's outputs back to the
OS's skills, allowing the system to get better at being itself over time.

### Design First, Build Last

The Fusion Layer's *design* (data contracts, schemas, approval gates) must be written before
the first skill is built — so that skills are created with evaluation in mind. The Fusion
Layer's *implementation* (the actual judge and pattern-learner skills) comes last, after you
have 5+ skills with enough log data to evaluate.

### The Fusion Layer Data Flow

```
EXECUTION (Skills + Workflows)
        │
        │ produces outputs, written to DB
        ▼
SKILL EVALUATOR SKILL (Tier 4)
  Input:
    skill_name: string
    skill_md: string (full skill.md content)
    recent_outputs: [last N output samples from DB]
    success_metrics: [from skill.md]
  Output:
    score: float (0.0–1.0)
    metric_breakdown: [{metric, passed: bool, note: string}]
    failure_patterns: [string]          ← key field for pattern learner
    recommended_changes: [
      {field: string,
       current_value: string,
       proposed_value: string,
       reasoning: string,
       conservative: bool}              ← must be true or change is rejected
    ]
    verdict: "pass" | "update_recommended" | "update_required"
        │
        │ if verdict != "pass"
        ▼
PATTERN LEARNER SKILL (Tier 4)
  Input:
    skill_evaluator_output: [evaluator outputs for multiple skills]
    log_entries: [recent log records from DB, last 30 days]
    behavioral_context: [how the human uses the system — from DB patterns]
  Output:
    patterns_found: [string]            ← what the learner observed
    improvement_perspectives: [
      {skill_name: string,
       perspective: string,
       tradeoffs: string,              ← required field — no tradeoff-free proposals
       evidence_citations: [log_entry_id]}
    ]
    proposed_skill_changes: [
      {skill_name: string,
       field: string,
       change: string,
       justification: string}
    ]
        │
        ▼
HUMAN APPROVAL GATE (mandatory, non-bypassable)
  Show: current skill.md
  Show: proposed changes with evidence
  Ask: approve / reject / ask for revision
        │
        │ if approved
        ▼
SKILL UPDATE SKILL (Tier 4)
  Applies approved changes to skill.md
  Increments version (MINOR for refinements, MAJOR for interface changes)
  Writes to CHANGELOG.md
  Re-runs test cases from tests/test-cases.md
  If tests fail: reverts and notifies
  If tests pass: deploys new version; logs to OS DB
```

### The Inviolable Constraint

The pattern learner must have `tradeoffs` as a required output field. No proposal without
stated tradeoffs is accepted. This prevents unconstrained optimization — every improvement
must acknowledge what it gives up. Skills cannot expand their scope through this process.

---

## Anti-Patterns

### Bad Skill Patterns

| Anti-Pattern | What It Looks Like | Why It Fails |
|---|---|---|
| The God Skill | "Help me with my work" as goal | No success metric is possible |
| The Bloated Skill | skill.md > 500 lines | Token waste; hallucination rate rises |
| The Vague Trigger | "When the user asks about something technical" | Activates unpredictably |
| The Static Skill | Never versioned; never tested | Drifts silently; undetectable failure |
| The Silent Failure | No error field in output format | Caller cannot detect failure |
| The Scope Creeper | "Just one more thing" added to existing skill | Becomes two skills poorly |

### Bad Workflow Patterns

| Anti-Pattern | What It Looks Like | Why It Fails |
|---|---|---|
| Missing log-write node | Last node is functional, not log-write | Execution is invisible to audit |
| No error propagation | Nodes don't pass errors forward | Failures look like completions |
| Monolithic prompt node | One node does everything | Cannot debug which step failed |
| Approval gate at end only | Human sees result, not decision | No meaningful oversight |
| Unbounded loop | loop without max_iterations | Can run forever |

---

## Version Strategy

Skills use semantic versioning: `MAJOR.MINOR.PATCH`

- `PATCH` bump: typo fixes, edge case additions — no behavior change
- `MINOR` bump: system prompt refinement, metric tuning — no interface change
- `MAJOR` bump: goal, input format, or output format changes — breaking change

When a skill bumps MAJOR, all workflows that invoke it must be reviewed and re-tested before
the new version is deployed.

The version lives in `skill.md` YAML header. The history lives in `CHANGELOG.md`.

---

*Cross-references: README_ARCHITECTURE_AND_LAYERS.md (where execution layer sits in the
organism), README_MEMORY_CONTEXT_SECONDBRAIN.md (how skills read from and write to memory),
README_BUILDING_AND_GAPS.md (build order for skills by tier).*

*Part of MYAIOS backbone documentation. v1.0 — Synthesis.*
