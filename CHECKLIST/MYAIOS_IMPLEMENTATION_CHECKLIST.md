# MYAIOS_IMPLEMENTATION_CHECKLIST.md
## The Definitive Agent Instruction Prompt for Building MYAIOS

> **What this file is:** A complete, recursive, multi-level implementation checklist that an
> AI agent follows to build MYAIOS from zero to a living, self-improving Personal Intelligence
> Operating System. Every item is anchored to one of the 5 backbone READMEs. Covers: folder
> structure, file contents, naming conventions, component relationships, data contracts,
> behavioral properties, semantic/abstract quality, and the living characteristics that
> separate an OS from a file collection.
>
> **How to use this:** Load this file plus all 5 READMEs into your agent's context. This file
> IS the instruction. The agent works sequentially, never skips a phase gate, and updates
> CLAUDE.md every time a component is built. The checklist is also a living document — update
> it when architecture evolves.
>
> **The 5 authoritative sources (agent must keep all in context):**
> - `README_VISION_AND_SOUL.md` → the why, the principles, the relationship model
> - `README_ARCHITECTURE_AND_LAYERS.md` → the 7 layers, the full diagram, the trust boundary
> - `README_SKILLS_TEMPLATES_WORKFLOWS.md` → the execution engine, skill ontology, Fusion Layer
> - `README_MEMORY_CONTEXT_SECONDBRAIN.md` → memory taxonomy, context types, Second Brain
> - `README_BUILDING_AND_GAPS.md` → build sequence, gap registry, risk register

---

## ══════════════════════════════════════════════════════
## AGENT OPERATING CONTRACT
## Read this before every single session. Non-negotiable.
## ══════════════════════════════════════════════════════

```
BEFORE every action, ask:
  1. Which README section governs this component?
  2. Which phase am I in? Do all upstream dependencies exist?
  3. After this action: what must be updated?
     → CLAUDE.md (wire the new component)
     → An index.md or log.md (record it)
     → gap.md (did this close a gap?)
     → AUDIT.md (does the architecture doc still match reality?)

AFTER every action, verify:
  1. Was a log entry written? (log-write-skill called — mandatory)
  2. Was CLAUDE.md updated? (orphan components are dead tissue)
  3. Does the filesystem match the intended architecture?
  4. Is the component connected in both directions? (it calls others AND others can call it)

MINDSET:
  You are growing a living organism, not installing software.
  A component that is built but not wired to CLAUDE.md does not exist.
  A workflow that runs but produces no log entry did not happen.
  A skill that has no success_metrics cannot know if it succeeded.
  Build vertically: one complete slice before the next.
  Never build horizontally: 10 half-finished things is worse than 3 complete ones.

THE ORGANISM METAPHOR GOVERNS EVERY DECISION:
  Substrate (L1) = DNA / immune system — self-improving, reads everything
  Nervous System (L2) = MCPs — sensory organs connecting OS to world
  Mind (L3) = Second Brain + DB + Logs — compiled memory
  Cognition (L4) = CLAUDE.md + Context — how the OS thinks and routes
  Execution (L5) = Skills + Workflows + CronJobs — muscles
  Awareness (L6) = Audit + Gap — self-model
  Voice (L7) = Notifier + Dashboard — how OS speaks outward
  All 7 layers must be alive for MYAIOS to function as intended.
```

---

## ══════════════════════════════════════════════════════
## PART A: UNIVERSAL NAMING CONVENTIONS
## Apply to every file, folder, skill, workflow created.
## ══════════════════════════════════════════════════════

### A.1 — Folder Naming
- [ ] All folders: `kebab-case` (no underscores, no capitals)
  - [ ] `skills/`, `workflows/`, `templates/`, `context/`, `scripts/`, `db/`
  - [ ] Inside `skills/`: `log-write-skill/`, `github-mcp-skill/` — not `LogWrite/`, not `log_write/`
- [ ] Folder names are self-describing without a README to explain them
- [ ] No folders named: `misc/`, `temp/`, `stuff/`, `old/`, `new/`, `v2/`

### A.2 — File Naming
- [ ] Skill specs: `skill.md` (always lowercase, always `skill.md` — not `spec.md`, not `README.md`)
- [ ] Workflow definitions: `[workflow-name].yaml` in kebab-case
- [ ] Context files: `static.md`, `dynamic.md` (predictable names — context-load-skill hardcodes them)
- [ ] Test files: `test-cases.md` inside each skill's `tests/` folder
- [ ] Version history: `CHANGELOG.md` (uppercase — a permanent record deserves prominence)
- [ ] Architecture docs: `CLAUDE.md`, `AUDIT.md`, `gap.md` (uppercase for OS-level docs)
- [ ] Config: `.env`, `config.json`, `.env.example` (exact names — scripts hardcode them)
- [ ] Log files: `log.md` (not `logs.md`, not `log.txt`)

### A.3 — Skill Naming
- [ ] Pattern: `[domain]-[action]-skill`
  - [ ] Domain examples: `github`, `slack`, `telegram`, `context`, `log`, `db`, `second-brain`,
        `pattern`, `audit`, `gap`, `notifier`, `daily`, `weekly`
  - [ ] Action examples: `ingest`, `query`, `write`, `update`, `test`, `judge`, `review`,
        `draft`, `load`, `detect`
- [ ] Examples: `github-mcp-skill`, `log-write-skill`, `context-load-skill`, `skill-judge-skill`
- [ ] A skill name must make its purpose obvious without reading its goal

### A.4 — Workflow Naming
- [ ] Pattern: `[cadence/trigger]-[subject]-[action]`
  - [ ] Examples: `nightly-wiki-ingest`, `weekly-pr-review`, `morning-briefing`,
        `skill-improvement-cycle`, `mcp-health-check`
- [ ] Workflow names are verb-phrases describing what they DO, not what they ARE

### A.5 — Wiki Slug Naming (Second Brain)
- [ ] Pattern: `[category]-[identifier]` in kebab-case
  - [ ] Sources: `paper-attention-is-all-you-need`, `transcript-karpathy-lecture-2024`
  - [ ] Concepts: `transformer-architecture`, `scaling-laws`, `rl-from-human-feedback`
  - [ ] Entities: `andrej-karpathy`, `openai`, `claude-sonnet`
  - [ ] Queries: `q-why-does-gpt4-struggle-with-math`
- [ ] Slugs are permanent identifiers — never rename after creation (update content, not slug)

### A.6 — Version Numbering
- [ ] Skills: `MAJOR.MINOR.PATCH` semantic versioning
  - [ ] PATCH: typo fixes, edge case additions — no behavior change
  - [ ] MINOR: prompt refinement, metric tuning — no interface change
  - [ ] MAJOR: goal, input format, or output format changes — breaking change
- [ ] CLAUDE.md: integer version `Version: N`
- [ ] Templates: `MAJOR.MINOR`
- [ ] Workflows: `MAJOR.MINOR`

---

## ══════════════════════════════════════════════════════
## PART B: COMPONENT RELATIONSHIP MAP
## Every connection that must exist. Build the map as you build the OS.
## ══════════════════════════════════════════════════════

### B.1 — The Wiring Map (what connects to what)

Every line below is a real dependency. If line N exists but not line N-1, N is broken.

```
YOU
 └──instructs──► CLAUDE.md (OS)
                  ├──routes to──► context-load-skill
                  │                └──reads──► context/static.md
                  │                └──queries──► DB (logs, issues)
                  │                └──queries──► Fresh_SecondBrain/index.md
                  │                └──queries──► DB (log patterns for reverse-engineering)
                  │
                  ├──routes to──► [any skill]
                  │                └──ends with──► log-write-skill
                  │                               └──writes to──► log.md (append-only)
                  │                               └──writes to──► DB logs table
                  │
                  ├──routes to──► [any workflow]
                  │                └──node N calls──► [skill]
                  │                └──final node calls──► log-write-skill
                  │                └──conditional calls──► notifier-skill
                  │
                  ├──routes to──► MCP skills (sensory layer)
                  │                └──each reads/writes──► external world
                  │                └──each ends with──► log-write-skill
                  │
                  └──governs──► escalation protocol
                                └──notifier-skill
                                   └──telegram-mcp-skill → Telegram
                                   └──google-workspace-skill → Gmail

Fresh_SecondBrain/CLAUDE.md (parallel, non-overlapping)
 └──governs──► second-brain-ingest-skill
 └──governs──► second-brain-query-skill
 └──governs──► wiki structure and contradiction protocol
 └──does NOT govern──► OS routing, MCP connections, skill invocation
```

- [ ] **B.1.1** — The wiring map above is visibly accurate for the current build state
- [ ] **B.1.2** — No skill exists that is not reachable from CLAUDE.md
- [ ] **B.1.3** — No workflow exists whose final node is not log-write
- [ ] **B.1.4** — No MCP skill exists without a corresponding entry in CLAUDE.md `## MCP Connections`
- [ ] **B.1.5** — The two CLAUDE.md instances have zero overlapping instructions

### B.2 — Skill Tier Dependencies (must be satisfied before building each tier)

- [ ] **B.2.1** — Before building ANY Tier 1 skill: all 4 Tier 0 skills are functional
- [ ] **B.2.2** — Before building ANY Tier 2 skill: at least 2 Tier 1 MCP skills are functional
- [ ] **B.2.3** — Before building ANY Tier 3 skill: all Tier 2 cognitive skills are functional
- [ ] **B.2.4** — Before building ANY Tier 4 skill:
  - [ ] 5+ Tier 1-3 skills have run for at least 2 weeks
  - [ ] DB logs table has ≥100 real entries
  - [ ] audit-skill passes with no CRITICAL failures
  - [ ] Fusion Layer design is documented in `fusion-skill-template-workflow.idea`

### B.3 — Data Flow Integrity

For every workflow that runs, these connections must produce data:

- [ ] **B.3.1** — Trigger → CLAUDE.md routing → context-load-skill → live context window
- [ ] **B.3.2** — Skill execution → output → downstream skill or output destination
- [ ] **B.3.3** — Every output destination is one of:
  - [ ] `wiki/sources/` or `wiki/` subdirectory (new knowledge)
  - [ ] `db/reports/` or equivalent (synthesized artifacts)
  - [ ] DB table (operational records)
  - [ ] External service via MCP (write operations)
  - [ ] Notifier (alerts)
  - [ ] NOT: ephemeral chat output that disappears
- [ ] **B.3.4** — Every output destination has a corresponding reading path
      (what is written must be readable by a future skill)

---

## ══════════════════════════════════════════════════════
## PART C: PHASE-BY-PHASE BUILD CHECKLIST
## Sequential. Never skip a phase gate.
## ══════════════════════════════════════════════════════

---

### ▶ PHASE 0: DESIGN FOUNDATION
**Gate to next phase:** All items in 0.A–0.G complete. Nothing built yet — only written.

#### 0.A — Root Directory Architecture

- [ ] **0.A.1** — Root structure is clean and intentional:
  ```
  /
  ├── CLAUDE.md              ← OS nervous system router
  ├── AUDIT.md               ← architecture ground truth (stub ok at Phase 0)
  ├── gap.md                 ← living gap registry (stub ok)
  ├── building.prompt        ← builder protocol (must be written)
  ├── .env                   ← credentials (never committed)
  ├── .env.example           ← credential template (committed)
  ├── config.json            ← MCP config + system settings
  ├── skills/                ← all skills live here
  ├── workflows/             ← all workflow YAMLs
  ├── templates/             ← execution output templates
  ├── context/               ← static.md + dynamic.md
  ├── db/                    ← database files + schema
  ├── scripts/               ← utility scripts (test_mcps.py etc)
  └── Fresh_SecondBrain/     ← compiled knowledge layer
  ```
- [ ] **0.A.2** — No folder exists at root that is not in the list above
- [ ] **0.A.3** — A new contributor can infer the purpose of every root item in <30 seconds
- [ ] **0.A.4** — `.gitignore` exists and includes: `.env`, `db/*.db`, `__pycache__/`,
      `*.pyc`, any large binary files
- [ ] **0.A.5** — A `README.md` at root briefly states what MYAIOS is and points to
      CLAUDE.md for operational instructions (not a long doc — 10 lines max)

#### 0.B — OS CLAUDE.md: The Nervous System

> Source: README_ARCHITECTURE_AND_LAYERS.md § "CLAUDE.md: The Nervous System (Anatomy)"

- [ ] **0.B.1** — File exists at root as `CLAUDE.md`
- [ ] **0.B.2** — Version header at top: `# CLAUDE.md — MYAIOS v[N] | Updated: [date]`
- [ ] **0.B.3** — Contains all 9 required sections, in this order:

  - [ ] **§ Identity** — one paragraph: who you are, what this OS serves, primary domains
    - [ ] Names the human (first person: "I am...")
    - [ ] Names the primary expertise domain (AI/ML research, deep learning, etc.)
    - [ ] States the OS's core purpose in one sentence
    - [ ] Is specific enough that an LLM reading it knows exactly who it's serving

  - [ ] **§ Directory Map** — every folder at root, what it contains, when it changes
    - [ ] Each folder has: path, purpose (one line), change frequency
    - [ ] Flags `Fresh_SecondBrain/raw/` as IMMUTABLE (LLM never writes here)
    - [ ] Flags `.env` as credentials (never read aloud, never logged)

  - [ ] **§ Available Skills** — table: skill name | purpose (one line) | tier
    - [ ] Empty at Phase 0 — populated as skills are built
    - [ ] Every skill added to CLAUDE.md same session it's built
    - [ ] No skill appears here without a corresponding `skills/[name]/skill.md`

  - [ ] **§ Available Workflows** — table: name | trigger type | workflow file path
    - [ ] Empty at Phase 0

  - [ ] **§ MCP Connections** — table: MCP name | config key in config.json | test ref
    - [ ] Empty at Phase 0

  - [ ] **§ Context Loading Protocol** — exact ordered steps:
    - [ ] Step 1: Read `context/static.md` (always, unconditionally)
    - [ ] Step 2: Query DB logs (last 7 days) + DB issues (open) for dynamic context
    - [ ] Step 3: If session has a known topic, query wiki via index.md (optional at start)
    - [ ] Step 4: Query DB log patterns for reverse-engineered behavioral observations
    - [ ] Step 5: Enforce token budgets (static: unlimited, dynamic: ≤1000 tokens,
          wiki: ≤2000 tokens, reverse-engineered: ≤500 tokens)
    - [ ] Step 6: Assemble and return structured context object

  - [ ] **§ Log Writing Protocol** — states:
    - [ ] log-write-skill is called after every action (no exceptions)
    - [ ] Every log summary must answer: what happened, what changed, did it succeed
    - [ ] log.md is append-only (never overwrite)
    - [ ] DB logs table is written simultaneously with log.md

  - [ ] **§ Escalation Protocol** — exact trigger conditions for notifier-skill:
    - [ ] Any skill returns an error field that is non-empty
    - [ ] Audit-skill finds CRITICAL failures
    - [ ] An approval gate is reached in a workflow
    - [ ] MCP health check fails for any MCP
    - [ ] gap.md has not been updated in 7 days

  - [ ] **§ Evolution Protocol** — states:
    - [ ] CLAUDE.md is modified only via explicit admin instruction
    - [ ] Every modification increments the version number
    - [ ] New components wired in same session they are built
    - [ ] File never exceeds 300 lines (implementation detail belongs in skill.md)
    - [ ] After any change: re-read and verify all cross-references still work

- [ ] **0.B.4** — Quality gates:
  - [ ] Every sentence is deterministic ("when X, do Y") — no aspirational language
  - [ ] No section describes HOW a skill works — only THAT it exists and WHAT it's called
  - [ ] File is ≤300 lines
  - [ ] Reading CLAUDE.md alone, an agent can locate any component without guessing

- [ ] **0.B.5** — Semantic check:
  - [ ] CLAUDE.md reads as firmware, not documentation
  - [ ] The Identity section makes clear this serves a specific human in a specific domain
  - [ ] Any LLM reading this understands: what to do, where things live, how to escalate

#### 0.C — building.prompt: The Builder Protocol

> Source: README_BUILDING_AND_GAPS.md § Gap 1

- [ ] **0.C.1** — File is NOT empty
- [ ] **0.C.2** — Provides exact step-by-step protocol for each build action:

  - [ ] **Building a new skill:**
    - [ ] Step 1: Create folder `skills/[skill-name]/`
    - [ ] Step 2: Create `skill.md` with all required sections (use template from README 3)
    - [ ] Step 3: Add skill to CLAUDE.md `## Available Skills` table
    - [ ] Step 4: Create `tests/test-cases.md` with ≥2 test cases
    - [ ] Step 5: Create `CHANGELOG.md` with initial version entry
    - [ ] Step 6: Run test cases — skill must pass before being considered deployed
    - [ ] Step 7: Write a log entry confirming skill deployment
    - [ ] Step 8: Update `gap.md` if this skill closed a gap

  - [ ] **Building a new workflow:**
    - [ ] Step 1: Create `workflows/[name].yaml`
    - [ ] Step 2: Every node has a `depends_on` chain
    - [ ] Step 3: Final node is always log-write
    - [ ] Step 4: Add workflow to CLAUDE.md `## Available Workflows`
    - [ ] Step 5: Run manually end-to-end before scheduling
    - [ ] Step 6: Verify log entry was written after manual run

  - [ ] **Connecting a new MCP:**
    - [ ] Step 1: Add credentials to `.env`
    - [ ] Step 2: Add config entry to `config.json`
    - [ ] Step 3: Update `scripts/test_mcps.py` to include new MCP
    - [ ] Step 4: Run `test_mcps.py` — MCP must be reachable before skill is built
    - [ ] Step 5: Build the MCP skill
    - [ ] Step 6: Add to CLAUDE.md `## MCP Connections`

  - [ ] **Ingesting a source into the Second Brain:**
    - [ ] Step 1: Drop file into `Fresh_SecondBrain/raw/pending/`
    - [ ] Step 2: Run second-brain-ingest-skill on that file (one at a time)
    - [ ] Step 3: Verify wiki page created in `wiki/sources/`
    - [ ] Step 4: Verify `index.md` and `log.md` updated
    - [ ] Step 5: Check if any contradictions were flagged
    - [ ] Step 6: Write OS log entry via log-write-skill

  - [ ] **Updating a README when a component is deployed:**
    - [ ] Find the README section that governs this component type
    - [ ] Update the "Current State Assessment" completion percentage
    - [ ] If a gap was closed: update the gap status in README_BUILDING_AND_GAPS.md
    - [ ] Increment README version header

- [ ] **0.C.3** — Uses "must" not "should" — prescriptive, not suggestive

#### 0.D — Infrastructure Config Files

- [ ] **0.D.1** — `.env` exists
  - [ ] Listed in `.gitignore` — never committed
  - [ ] Contains placeholder comment for each expected key
  - [ ] Keys: `ANTHROPIC_API_KEY`, `GITHUB_TOKEN`, `SLACK_TOKEN`,
        `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`, `GOOGLE_CLIENT_ID`,
        `GOOGLE_CLIENT_SECRET`, and any additional MCP keys

- [ ] **0.D.2** — `.env.example` exists (committed to git)
  - [ ] All keys listed with empty values and a comment describing each
  - [ ] New contributor can understand what credentials are needed from this file alone

- [ ] **0.D.3** — `config.json` exists with schema:
  - [ ] `mcps: {}` block (populated as MCPs are connected)
  - [ ] `models: { default: "claude-sonnet-4-20250514", fast: "claude-haiku-4-5-20251001" }`
  - [ ] `context_token_budgets: { static: null, dynamic: 1000, wiki: 2000, reverse_engineered: 500 }`
  - [ ] `feature_flags: { notifier_enabled: false, audit_enabled: false }` (start false; enable as built)
  - [ ] `second_brain_path: "Fresh_SecondBrain/"` (canonical path reference)
  - [ ] `log_path: "log.md"` (canonical OS log path)

- [ ] **0.D.4** — `scripts/` directory exists
  - [ ] `scripts/test_mcps.py` exists (even as a stub with TODO comments)
  - [ ] Script has a docstring explaining it is permanent infrastructure
  - [ ] Script reads from `.env` and `config.json` (not hardcoded credentials)

#### 0.E — Fusion Layer Design (Fill the Blank)

> Source: README_SKILLS_TEMPLATES_WORKFLOWS.md § "THE FUSION LAYER — Resolved"

- [ ] **0.E.1** — `fusion-skill-template-workflow.idea` is no longer empty
- [ ] **0.E.2** — Contains, at minimum:
  - [ ] The Fusion Layer purpose: "The self-improvement loop that makes skills better at being you"
  - [ ] The three skill types: skill-judge, pattern-learner, skill-update
  - [ ] skill-judge output schema (score, metric_breakdown, failure_patterns, recommended_changes)
  - [ ] `recommended_changes` sub-schema: `{field, current_value, proposed_value, reasoning, conservative: bool}`
  - [ ] pattern-learner required field: `tradeoffs` (no proposal without stated tradeoffs)
  - [ ] The 5-step update cycle: judge → learn → approve (HUMAN) → test → deploy
  - [ ] The inviolable constraint: no skill can expand its scope through this process
  - [ ] The gate: Fusion Layer is designed now, implemented only after 5+ skills + 2 weeks of logs

#### 0.F — Static Context File

> Source: README_MEMORY_CONTEXT_SECONDBRAIN.md § "Type 1: Static Context"

- [ ] **0.F.1** — `context/static.md` exists and is fully written (not a stub)
- [ ] **0.F.2** — Contains all required sections:
  - [ ] `## Identity` — name, background, primary expertise, current role
  - [ ] `## Expertise Domains` — specific domains with depth levels
        (e.g., "transformer architectures: deep; RL: intermediate; systems: foundational")
  - [ ] `## Writing Voice` — how you write; what you avoid; example sentence
  - [ ] `## Working Methods` — how you approach problems, what frameworks you use
  - [ ] `## Active Projects` — name, one-line description, current phase, priority
  - [ ] `## Long-Term Goals` — personal + professional, time horizon
  - [ ] `## Organizations` — orgs you're part of, your role, relevant context
  - [ ] `## Integrations` — which MCPs you use for what purpose
  - [ ] `## Preferences` — output format, depth, how you like decisions presented
  - [ ] `## What I Care About` — values that should influence OS behavior
- [ ] **0.F.3** — File is written in first person ("I am...", "I prefer...", "My goal is...")
- [ ] **0.F.4** — No placeholder text — every field has real content
- [ ] **0.F.5** — Semantic check: an LLM reading only this file can answer:
  - [ ] "What would this person prioritize today?"
  - [ ] "How would this person want this email written?"
  - [ ] "What does this person already know about X?"

#### 0.G — Phase 0 Gate Check

- [ ] CLAUDE.md exists and has all 9 sections ✓
- [ ] `building.prompt` is fully written ✓
- [ ] `.env`, `.env.example`, `config.json` exist ✓
- [ ] `fusion-skill-template-workflow.idea` is filled in ✓
- [ ] `context/static.md` is fully written ✓
- [ ] All required top-level directories exist ✓
- [ ] Root README.md exists ✓
- [ ] **→ Phase 0 complete. No code written yet. Foundation is laid.**

---

### ▶ PHASE 1: INFRASTRUCTURE
**Gate to next phase:** Tier 0 skills functional + DB queryable + end-to-end Tier 0 test passes.

#### 1.A — Database

> Source: README_ARCHITECTURE_AND_LAYERS.md § "DATABASE (Operational Record)"
> Source: README_MEMORY_CONTEXT_SECONDBRAIN.md § "Database: Operational Memory"

- [ ] **1.A.1** — Technology chosen and documented in `db/README.md`
  - [ ] Recommended: SQLite (`db/myaios.db`) for Phase 1-3; migrate later if needed
  - [ ] `db/README.md` states: what DB is for, what it is NOT for
        (operational memory ≠ knowledge; Second Brain is knowledge)

- [ ] **1.A.2** — Schema file exists: `db/schema.sql`

- [ ] **1.A.3** — `logs` table:
  - [ ] `id` (primary key, auto-increment)
  - [ ] `timestamp` (ISO8601, not null)
  - [ ] `workflow_id` (text, nullable — for workflow-originated entries)
  - [ ] `skill_name` (text, nullable — for skill-originated entries)
  - [ ] `stage` (text, not null — e.g., "complete", "error", "context-load")
  - [ ] `summary` (text ≤200 chars, not null — enough to reconstruct context)
  - [ ] `errors` (JSON array, nullable)
  - [ ] `db_refs` (JSON array of record IDs, nullable)
  - [ ] NO UPDATE or DELETE permitted on this table ever
  - [ ] `summary` validation: must answer "what happened, what changed, did it succeed"

- [ ] **1.A.4** — `reports` table:
  - [ ] `id`, `report_type` (daily/weekly/monthly), `date`, `filepath`,
        `summary` (≤200 chars), `created_at`

- [ ] **1.A.5** — `issues` table:
  - [ ] `id`, `detected_at`, `source` (audit/mcp/workflow/manual),
        `type`, `description`, `severity` (critical/high/medium/low),
        `status` (open/resolved), `resolved_at` (nullable), `resolution_note` (nullable)
  - [ ] Status only transitions open → resolved (never deleted)
  - [ ] Closed issues remain visible forever — history matters

- [ ] **1.A.6** — `second_brain_ingests` table:
  - [ ] `id`, `source_filename`, `source_subfolder` (articles/papers/notes/etc),
        `ingested_at`, `pages_created` (JSON array), `pages_updated` (JSON array),
        `conflicts_flagged` (integer), `status` (pending/complete/failed), `error_message` (nullable)

- [ ] **1.A.7** — `mcp_ingests` table:
  - [ ] `id`, `mcp_source`, `fetched_at`, `item_count`, `item_type`
        (issues/prs/messages/emails/etc), `processed` (boolean, default false),
        `processed_at` (nullable), `output_skill` (nullable), `output_ref` (nullable)

- [ ] **1.A.8** — `cronjobs` table:
  - [ ] `id`, `name`, `schedule` (cron expression), `workflow_name`,
        `last_run` (nullable), `next_run` (nullable),
        `last_status` (pending/running/complete/failed, nullable),
        `last_output_ref` (nullable), `enabled` (boolean, default true)

- [ ] **1.A.9** — `context_patterns` table (for reverse-engineered context):
  - [ ] `id`, `pattern_type`, `description`, `evidence_log_ids` (JSON array),
        `confidence` (float 0-1), `created_at`, `last_updated`
  - [ ] Written by pattern-learner-skill; read by context-load-skill

- [ ] **1.A.10** — DB is queryable from command line for debugging

#### 1.B — Tier 0 Skills

> Source: README_SKILLS_TEMPLATES_WORKFLOWS.md § "The Skill Ontology — Tier 0"

**For each Tier 0 skill, the following sub-checklist applies:**

**Universal skill sub-checklist (apply to EVERY skill built at every tier):**
- [ ] Folder exists: `skills/[skill-name]/`
- [ ] `skill.md` exists with YAML front-matter header:
  - [ ] `name:` matches folder name exactly
  - [ ] `version:` semantic version starting at "1.0.0"
  - [ ] `goal:` ONE sentence (hard limit — if it's two sentences, it's two skills)
  - [ ] `trigger_conditions:` at least 2 specific trigger phrases
  - [ ] `argument_hint:` what input this skill expects
  - [ ] `last_tested:` date of last successful test run
- [ ] `skill.md` body sections:
  - [ ] `## System Prompt` — deterministic LLM instruction, not aspirational
  - [ ] `## Input Format` — exact fields, types, required/optional
  - [ ] `## Output Format` — exact fields; errors field always present even if nullable
  - [ ] `## Step-by-Step Protocol` — numbered steps; each step is an action, not a suggestion
  - [ ] `## Success Metrics` — minimum 3 testable, binary pass/fail criteria
  - [ ] `## Edge Cases` — minimum 2 edge cases with explicit handling instructions
  - [ ] `## Dependencies` — list of MCPs or other skills required
  - [ ] `## Output Metadata` — footer appended to every output: `Skill: [name] | v[version] | [timestamp]`
- [ ] `tests/test-cases.md` exists with:
  - [ ] At least 2 test cases (more for complex skills)
  - [ ] Each test has: input, expected output, pass criteria
  - [ ] At least 1 edge case test
- [ ] `CHANGELOG.md` exists with initial entry: `## v1.0.0 — [date] — Initial deployment`
- [ ] Skill listed in OS CLAUDE.md `## Available Skills` same session it's built

---

**1.B.1 — log-write-skill** (`skills/log-write-skill/`)
- [ ] Goal is exactly: writing a structured log entry to both log.md and DB simultaneously
- [ ] Input format includes: `workflow_id`, `skill_name`, `stage`, `summary`, `errors`, `db_refs`
- [ ] Output format includes: `log_entry_id`, `written_to: [log.md, database]`, `timestamp`
- [ ] Protocol:
  - [ ] Step 1: Validate summary is ≤200 chars and answers "what happened, what changed, did it succeed"
  - [ ] Step 2: Format log.md entry: `## [timestamp] | [stage] | [workflow_id or skill_name]`
  - [ ] Step 3: Append to log.md (never overwrite — append means append)
  - [ ] Step 4: Write identical structured record to DB logs table
  - [ ] Step 5: Return `log_entry_id` from DB + confirmation of both writes
- [ ] Edge cases:
  - [ ] DB unavailable: still write to log.md; return partial success with warning
  - [ ] log.md path doesn't exist: create it; do not fail
  - [ ] Summary exceeds 200 chars: truncate to 197 + "..." rather than reject
- [ ] Success metrics:
  - [ ] log.md contains new entry with today's timestamp
  - [ ] DB logs table has new record with matching `log_entry_id`
  - [ ] Both writes happen in same function call (not sequential with risk of partial failure)
  - [ ] `errors` field present in output even when null

**1.B.2 — db-query-skill** (`skills/db-query-skill/`)
- [ ] Goal: query the operational database and return structured results
- [ ] Covers all 7 DB tables
- [ ] Input: `table`, `filters` (JSON, optional), `limit` (int, optional), `order_by` (optional)
- [ ] Output: `results: [records]`, `count: int`, `table_queried: string`, `errors`
- [ ] Returns empty `results: []` + error message on failure (never throws)
- [ ] Success metrics:
  - [ ] Returns valid JSON for all 7 tables
  - [ ] Respects limit parameter (never returns more than specified)
  - [ ] Empty table returns `results: []` not an error

**1.B.3 — context-load-skill** (`skills/context-load-skill/`)
- [ ] Goal: assemble the 4 context types into a fused context object at session start
- [ ] Step-by-step protocol exactly matches CLAUDE.md `## Context Loading Protocol`
- [ ] Enforces token budgets per type (reads from `config.json`)
- [ ] Returns structured context with 4 typed sections even if some are empty
- [ ] Output format:
  - [ ] `static: {content, token_count}`
  - [ ] `dynamic: {content, token_count, sources_queried}`
  - [ ] `second_brain: {content, token_count, pages_referenced}`
  - [ ] `reverse_engineered: {content, token_count, patterns_used}`
  - [ ] `total_token_count: int`
  - [ ] `assembly_timestamp: ISO8601`
- [ ] Edge case: DB unavailable → return static context only + warning in errors
- [ ] Edge case: Second Brain not yet seeded → return empty second_brain section + note

**1.B.4 — mcp-test-skill** (`skills/mcp-test-skill/`)
- [ ] Goal: test connectivity for all configured MCPs and return health status
- [ ] Reads MCP list from `config.json` (not hardcoded)
- [ ] Tests each MCP with minimal "ping" (not a full operation)
- [ ] Output:
  - [ ] `results: {[mcp_name]: {status: "healthy"|"degraded"|"unreachable", latency_ms, error}}`
  - [ ] `overall: "healthy"|"degraded"|"critical"`
  - [ ] `timestamp: ISO8601`
  - [ ] `healthy_count: int`, `total_count: int`
- [ ] `overall` logic: healthy if all pass; degraded if some fail; critical if >50% fail
- [ ] Does not crash when an MCP fails — catches all errors per MCP

Also:
- [ ] `scripts/test_mcps.py` is a standalone CLI script (not just a skill wrapper):
  - [ ] Reads `.env` and `config.json`
  - [ ] Prints per-MCP status with emoji indicators (✅/⚠️/❌)
  - [ ] Prints summary line at end
  - [ ] Exits with non-zero code if any MCP unreachable (for CI use)
  - [ ] Has a docstring: "Permanent infrastructure. Never delete this script."

#### 1.C — Phase 1 Integration Test

- [ ] **1.C.1** — Sequential integration test:
  - [ ] Call context-load-skill → succeeds → returns context object
  - [ ] Call log-write-skill with context-load result as summary → entry in log.md AND DB
  - [ ] Call db-query-skill on `logs` table → returns the entry just written
  - [ ] Three-step chain completes without error
- [ ] **1.C.2** — Verify log.md was created and has content
- [ ] **1.C.3** — Verify DB has the `logs` table with the test entry
- [ ] **→ Phase 1 complete. The OS can remember what it does.**

---

### ▶ PHASE 2: FIRST MCP + CONTEXT ENRICHMENT
**Gate:** filesystem-mcp connected + context-load verified + end-to-end log chain works.

#### 2.A — filesystem-mcp

- [ ] **2.A.1** — `config.json` has filesystem-mcp entry
- [ ] **2.A.2** — `scripts/test_mcps.py` tests filesystem-mcp
- [ ] **2.A.3** — filesystem-mcp-skill (`skills/filesystem-mcp-skill/`):
  - [ ] Goal: read, write, list, and search local files via filesystem MCP
  - [ ] Operations: `read`, `write`, `list`, `search`, `exists`, `move`
  - [ ] Hard rule: never writes to `Fresh_SecondBrain/raw/` (immutability guard)
  - [ ] Hard rule: never reads `.env` (credential protection)
  - [ ] Output always includes: `operation_performed`, `path_resolved`, `errors`
  - [ ] Registered in CLAUDE.md `## MCP Connections` AND `## Available Skills`

#### 2.B — Context Infrastructure

- [ ] **2.B.1** — `context/` directory contains:
  - [ ] `context/static.md` (written in Phase 0)
  - [ ] `context/dynamic.md` (placeholder — populated at runtime)
  - [ ] `context/README.md` explaining:
    - [ ] What each file is
    - [ ] Who writes to it (static: human only; dynamic: context-load-skill)
    - [ ] When each is updated
    - [ ] Token budget per type

- [ ] **2.B.2** — context-load-skill verified:
  - [ ] Reads `context/static.md` successfully
  - [ ] Queries DB for recent logs (dynamic context)
  - [ ] Returns structured context object
  - [ ] Log entry written after successful load

#### 2.C — Phase 2 Gate Check

- [ ] filesystem-mcp connected and health-checked ✓
- [ ] filesystem-mcp-skill built, tested, registered ✓
- [ ] Context loads with static content and DB-derived dynamic content ✓
- [ ] Full chain: context-load → action → log-write → db-query confirms log ✓
- [ ] **→ Phase 2 complete. OS has eyes and memory.**

---

### ▶ PHASE 3: SECOND BRAIN INTEGRATION
**Gate:** Second Brain schema complete + 5 sources ingested + wiki queryable from context-load.

#### 3.A — Fresh_SecondBrain/ Full Structure

> Source: README_MEMORY_CONTEXT_SECONDBRAIN.md § "Second Brain Structure"

- [ ] **3.A.1** — All directories exist:
  ```
  Fresh_SecondBrain/
  ├── raw/
  │   ├── articles/
  │   ├── papers/
  │   ├── notes/
  │   ├── transcripts/
  │   ├── data/
  │   ├── pending/          ← ingest queue
  │   └── assets/
  ├── wiki/
  │   ├── entities/
  │   ├── concepts/
  │   ├── sources/
  │   ├── comparisons/
  │   ├── timelines/
  │   ├── synthesis/
  │   │   ├── overview.md
  │   │   └── open-questions.md
  │   └── queries/
  └── templates/
  ```

- [ ] **3.A.2** — Root files in `Fresh_SecondBrain/`:
  - [ ] `CLAUDE.md` — operating schema (already mostly exists — verify completeness)
  - [ ] `index.md` — master catalog; acts as topological map (coordinates, not knowledge)
  - [ ] `log.md` — append-only Second Brain operation history
  - [ ] `schema-changelog.md` — when and why the schema changed

- [ ] **3.A.3** — `Fresh_SecondBrain/raw/README.md` states:
  - [ ] "This directory is IMMUTABLE. The LLM never writes here."
  - [ ] "Sources are dropped here by humans only."
  - [ ] "The wiki/ directory is the compiled output. raw/ is the source of truth."
  - [ ] "If wiki/ is ever corrupted, delete it and re-run ingest on all files in raw/."

- [ ] **3.A.4** — `index.md` structure:
  - [ ] Header with last-updated timestamp
  - [ ] Section per wiki subdirectory (entities, concepts, sources, synthesis, queries)
  - [ ] Each entry: slug → one-line description → last-modified date
  - [ ] Index is coordinates, not content — each entry points TO a page, not contains it

- [ ] **3.A.5** — `wiki/synthesis/overview.md` has initial content:
  - [ ] Header: "Master Synthesis — [domain, e.g., AI/ML Research]"
  - [ ] Sections: Current Understanding, Open Questions, Recent Developments
  - [ ] Updated after every ingest that changes the big picture

- [ ] **3.A.6** — `wiki/synthesis/open-questions.md` has initial content:
  - [ ] Format per question: `## [Conflict/Question Title]` + sources in tension + date flagged
  - [ ] Contradictions never silently resolved here — always human-reviewed

- [ ] **3.A.7** — `Fresh_SecondBrain/CLAUDE.md` scope is correct:
  - [ ] Contains: ingest protocol, wiki structure rules, template definitions,
        contradiction handling, lint protocol
  - [ ] Does NOT contain: OS routing, MCP config, skill invocation logic
  - [ ] Test: read both CLAUDE.md files; find zero overlapping instructions

- [ ] **3.A.8** — Templates in `Fresh_SecondBrain/templates/`:
  - [ ] `source-template.md` — for `wiki/sources/` pages:
    - [ ] Fields: title, source type, date, key claims (≥3), entities mentioned,
          concepts mentioned, contradictions with existing wiki, open questions raised,
          quality assessment (credibility, relevance)
  - [ ] `concept-template.md` — for `wiki/concepts/` pages:
    - [ ] Fields: definition, elaboration, evidence (citations), related concepts,
          entity associations, current understanding, open questions
  - [ ] `entity-template.md` — for `wiki/entities/` pages:
    - [ ] Fields: name, type (person/org/tool/model/dataset), role in domain,
          notable contributions, connections to other entities, source citations
  - [ ] `query-template.md` — for `wiki/queries/` pages:
    - [ ] Fields: question asked, date, context, synthesis answer, citations,
          confidence level, follow-up questions
  - [ ] `comparison-template.md`, `timeline-template.md`

#### 3.B — second-brain-ingest-skill

- [ ] **3.B.1** — Full skill.md complete (apply universal skill checklist)
- [ ] **3.B.2** — Protocol matches Fresh_SecondBrain/CLAUDE.md exactly:
  - [ ] Step 1: Read `Fresh_SecondBrain/CLAUDE.md` — every time, no skipping
  - [ ] Step 2: Read source file completely — never skim
  - [ ] Step 3: Create `wiki/sources/[slug].md` using source template
  - [ ] Step 4: Update all relevant `wiki/entities/` pages
  - [ ] Step 5: Update all relevant `wiki/concepts/` pages
  - [ ] Step 6: Contradiction check — flag with `> [!CONFLICT]`, never auto-resolve
  - [ ] Step 7: Update `wiki/synthesis/overview.md` if big picture changed
  - [ ] Step 8: Append to `Fresh_SecondBrain/log.md`
  - [ ] Step 9: Update `index.md`
  - [ ] Step 10: Move source from `raw/pending/` to appropriate `raw/` subfolder
  - [ ] Step 11: Write to `second_brain_ingests` DB table
  - [ ] Step 12: Call OS log-write-skill (OS log, separate from Second Brain log.md)
- [ ] **3.B.3** — One source at a time — never batch ingest (breaks contradiction detection)
- [ ] **3.B.4** — After skill runs, explicitly ask: "Should I file this as `wiki/queries/`?"

#### 3.C — github-mcp + github-mcp-skill

- [ ] **3.C.1** — `GITHUB_TOKEN` in `.env`, entry in `config.json`, test in `test_mcps.py`
- [ ] **3.C.2** — github-mcp-skill covers:
  - [ ] `fetch_issues`: list open issues since timestamp
  - [ ] `fetch_prs`: list PRs since timestamp with metadata
  - [ ] `fetch_commits`: list commits since timestamp
  - [ ] `post_review`: post review comment to a PR (has approval gate)
  - [ ] `create_issue`: create new issue (has approval gate)
- [ ] **3.C.3** — All write operations have an approval step before executing
- [ ] **3.C.4** — Registered in CLAUDE.md

#### 3.D — Second Brain Seeding

- [ ] **3.D.1** — At least 5 sources ingested:
  - [ ] At least 2 papers (arXiv or similar — domain-relevant)
  - [ ] At least 1 transcript or notes file
  - [ ] At least 2 articles
- [ ] **3.D.2** — After seeding:
  - [ ] `wiki/sources/` has ≥5 pages
  - [ ] `wiki/concepts/` has ≥3 pages (from cross-references during ingest)
  - [ ] `wiki/entities/` has ≥3 pages
  - [ ] `index.md` reflects all ingested sources
  - [ ] `Fresh_SecondBrain/log.md` has ≥5 entries (one per ingest)
  - [ ] `wiki/synthesis/open-questions.md` has at least 1 entry (contradictions exist in any real domain)

#### 3.E — Phase 3 Gate Check

- [ ] Second Brain structure complete ✓
- [ ] second-brain-ingest-skill functional, tested, registered ✓
- [ ] ≥5 sources ingested ✓
- [ ] context-load-skill now queries wiki via index.md when topic is known ✓
- [ ] Query test: ask a question covered by ingested sources →
      context contains relevant wiki content → answer cites compiled knowledge ✓
- [ ] **→ Phase 3 complete. OS has compiled memory.**

---

### ▶ PHASE 4: FIRST WORKFLOW + FIRST CRONJOB
**Gate:** nightly-wiki-ingest runs autonomously + log written + wiki updated. You did not trigger it.

#### 4.A — nightly-wiki-ingest Workflow

> Source: README_SKILLS_TEMPLATES_WORKFLOWS.md § "Complete Workflow Example: nightly-wiki-ingest"

- [ ] **4.A.1** — File exists: `workflows/nightly-wiki-ingest.yaml`
- [ ] **4.A.2** — YAML front-matter:
  - [ ] `name: nightly-wiki-ingest`
  - [ ] `version: "1.0.0"`
  - [ ] `description:` includes "Use when:" and "Goal:" fields
  - [ ] `trigger: { type: fixed, schedule: "0 23 * * *" }`
  - [ ] `provider: claude`, `model: sonnet`

- [ ] **4.A.3** — Required nodes in DAG order:
  - [ ] `scan-pending` — bash node: `ls Fresh_SecondBrain/raw/pending/ | wc -l`
  - [ ] `check-pending` — haiku node: parse count, return file list or nothing-to-do
  - [ ] `ingest-loop` — loop node:
    - [ ] `loop.items` references `check-pending.files`
    - [ ] `loop.max_iterations: 20` (prevents runaway)
    - [ ] `loop.prompt` reads `Fresh_SecondBrain/CLAUDE.md` each iteration
    - [ ] Each iteration ingests one file via second-brain-ingest-skill protocol
    - [ ] Each iteration returns: `{file, pages_created, pages_updated, conflicts_flagged, errors}`
  - [ ] `lint-wiki` — haiku node: checks index.md and log.md were updated today
  - [ ] `notify-if-conflicts` — haiku node: sends Telegram if conflicts > 0
  - [ ] `log-write` — **final node, always**:
    - [ ] Depends on: notify-if-conflicts AND lint-wiki
    - [ ] Summary includes: files processed count, pages created, any errors
    - [ ] No node depends on log-write (it is the terminal node)

- [ ] **4.A.4** — Workflow quality gates:
  - [ ] Graceful no-op when `raw/pending/` is empty (check-pending returns nothing-to-do)
  - [ ] Loop node has `max_iterations` set
  - [ ] Every node has `output_var` or `depends_on` — no orphan nodes
  - [ ] Error from any node propagates forward to log-write's `errors` field
  - [ ] Workflow can complete in FAILED state cleanly without hanging

#### 4.B — CronJob Registration

- [ ] **4.B.1** — DB `cronjobs` table has entry:
  - [ ] `name: nightly-wiki-ingest`
  - [ ] `schedule: "0 23 * * *"`
  - [ ] `workflow_name: nightly-wiki-ingest`
  - [ ] `enabled: true`
- [ ] **4.B.2** — CLAUDE.md `## Available Workflows` updated with nightly-wiki-ingest
- [ ] **4.B.3** — CronJob runner script exists (`scripts/run_cron.py` or equivalent)

#### 4.C — Workflow State Machine Verification

> Source: README_SKILLS_TEMPLATES_WORKFLOWS.md § "Workflow State Machine"

For the nightly-wiki-ingest workflow, verify all states reachable:
- [ ] PENDING → TRIGGERED → CONTEXT_LOADED → EXECUTING (normal path) ✓
- [ ] EXECUTING → AWAITING_APPROVAL (not present in this workflow — expected) ✓
- [ ] EXECUTING → LOGGING → COMPLETE (happy path) ✓
- [ ] EXECUTING → FAILED → NOTIFIED (error path — test by breaking one node) ✓
- [ ] COMPLETE is only reached when log-write node has confirmed writing ✓

#### 4.D — Phase 4 Milestone: The Proof of Concept

- [ ] **4.D.1** — Manual end-to-end run succeeds:
  - [ ] 2 test files in `raw/pending/`
  - [ ] Trigger workflow manually
  - [ ] wiki pages created ✓ / log entry written ✓ / DB updated ✓
- [ ] **4.D.2** — THE PROOF OF CONCEPT:
  - [ ] Set CronJob to trigger in 3 minutes
  - [ ] Wait. Do NOT manually trigger.
  - [ ] System runs. Wiki updated. Log entry exists. You did nothing.
  - [ ] **→ MYAIOS is alive. Write this down. Date it.**

---

### ▶ PHASE 5: EXPAND MCPs + NOTIFIER + HEARTBEAT
**Gate:** Telegram notifier working + 5 MVP CronJobs running + 3+ MCPs connected.

#### 5.A — telegram-mcp + Notifier

- [ ] **5.A.1** — `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID` in `.env`
- [ ] **5.A.2** — telegram-mcp-skill:
  - [ ] Goal: send a formatted Telegram message to the configured chat
  - [ ] Input: `message: string`, `parse_mode: "markdown"|"plain"` (optional)
  - [ ] Output: `sent: bool`, `message_id: string`, `errors`
  - [ ] Never fails silently — always returns `sent: false` + error on failure

- [ ] **5.A.3** — notifier-skill:
  - [ ] Goal: evaluate an event and route an alert to the right channel at the right severity
  - [ ] Input: `event_type: "error"|"decision"|"audit-failure"|"major-event"|"info"`,
        `message: string`, `severity: "critical"|"high"|"medium"|"low"`
  - [ ] Routing rules (from CLAUDE.md `## Escalation Protocol`):
    - [ ] critical/high → Telegram immediately
    - [ ] medium → Telegram at next scheduled check (or immediately if urgent)
    - [ ] low → OS log only (no Telegram)
  - [ ] After routing: always writes log entry

- [ ] **5.A.4** — config.json `feature_flags.notifier_enabled` set to `true`

- [ ] **5.A.5** — Notifier test:
  - [ ] Cause an intentional error in a skill
  - [ ] Verify Telegram message received within 30 seconds

#### 5.B — slack-mcp

- [ ] **5.B.1** — `SLACK_TOKEN` in `.env`, config entry, test script updated
- [ ] **5.B.2** — slack-mcp-skill covers: read messages, post to channel, post to thread, list channels

#### 5.C — google-workspace-mcp

- [ ] **5.C.1** — OAuth credentials in `.env`
- [ ] **5.C.2** — google-workspace-skill covers:
  - [ ] Gmail: `read_inbox`, `search_emails`, `draft_email`, `send_email`
  - [ ] Calendar: `list_events`, `check_availability`
  - [ ] Drive: `read_file`, `upload_file`, `list_files`
- [ ] **5.C.3** — All send/create operations have approval gates

#### 5.D — MVP CronJob Suite (all 5)

> Source: README_SKILLS_TEMPLATES_WORKFLOWS.md § "MVP CronJob Definitions"

- [ ] **5.D.1** — `nightly-wiki-ingest` — `0 23 * * *` ✓ (Phase 4)
- [ ] **5.D.2** — `morning-briefing` — `0 8 * * *`
  - [ ] Corresponding workflow `workflows/morning-briefing.yaml` exists
  - [ ] Uses: context-load + DB query (today's calendar + yesterday's log + open issues) + daily-report-skill
- [ ] **5.D.3** — `weekly-pr-review` — `0 9 * * MON`
  - [ ] Corresponding workflow exists
  - [ ] Uses: github-mcp-skill to fetch week's PRs + analysis
- [ ] **5.D.4** — `mcp-health-check` — `0 */6 * * *`
  - [ ] Corresponding workflow: calls mcp-test-skill → if degraded/critical → notifier
- [ ] **5.D.5** — `weekly-sb-lint` — `0 22 * * SUN`
  - [ ] Corresponding workflow: checks wiki for bloated pages (>2000 words), stale synthesis, missing index entries

- [ ] **5.D.6** — All 5 CronJobs have DB entries with correct schedule and workflow_name
- [ ] **5.D.7** — All 5 workflows exist as YAML files
- [ ] **5.D.8** — All 5 CronJobs registered in CLAUDE.md `## Available Workflows`

---

### ▶ PHASE 6: SELF-AWARENESS LAYER
**Gate:** AUDIT.md written + audit-skill catches real issues + gap.md populated.

#### 6.A — AUDIT.md: The Ground Truth

> Source: README_BUILDING_AND_GAPS.md § "What the Audit System Must Catch"

- [ ] **6.A.1** — `AUDIT.md` is fully written (not a stub)
- [ ] **6.A.2** — Header: `# AUDIT.md — MYAIOS Architecture Ground Truth | v[N] | [date]`
- [ ] **6.A.3** — Documents the INTENDED state of the OS:
  - [ ] All expected folders at root with their purposes
  - [ ] All skill names with their `skills/[name]/skill.md` paths
  - [ ] All workflow names with their YAML paths
  - [ ] All CronJob names with their schedules
  - [ ] All MCP connections with their config keys
  - [ ] All 7 DB tables
  - [ ] Both CLAUDE.md instances and their domains
  - [ ] `Fresh_SecondBrain/raw/` declared IMMUTABLE
  - [ ] `.env` declared credential file (existence checked, not contents)

- [ ] **6.A.4** — 10 specific audit checks listed:
  - [ ] All skills in CLAUDE.md have corresponding `skill.md` files
  - [ ] All workflows in CLAUDE.md have corresponding YAML files
  - [ ] All CronJobs in DB have valid corresponding workflows
  - [ ] DB has all required tables
  - [ ] `.env` exists (existence only — never read contents)
  - [ ] `scripts/test_mcps.py` exists and is runnable
  - [ ] `Fresh_SecondBrain/raw/` has not been modified (immutability check via git or hash)
  - [ ] `gap.md` has been updated in the last 7 days
  - [ ] No orphan skills (exist in filesystem but not in CLAUDE.md)
  - [ ] No skills missing `success_metrics` field in their skill.md

#### 6.B — audit-skill

- [ ] **6.B.1** — Full skill.md with all universal requirements
- [ ] **6.B.2** — Protocol runs all 10 checks from AUDIT.md
- [ ] **6.B.3** — Output:
  - [ ] `passed: [item descriptions]`
  - [ ] `warnings: [{item, description, severity}]`
  - [ ] `failures: [{item, description, severity, fix_hint}]`
  - [ ] `orphans: [skill or workflow names]`
  - [ ] `overall: "pass"|"warnings"|"failures"|"critical"`
- [ ] **6.B.4** — After running: appends last-run timestamp and summary to AUDIT.md
- [ ] **6.B.5** — If `overall` is `failures` or `critical`: triggers notifier-skill
- [ ] **6.B.6** — config.json `feature_flags.audit_enabled` set to `true`

#### 6.C — gap-detector-skill + gap.md

- [ ] **6.C.1** — `gap.md` exists at root with content (not a stub)
- [ ] **6.C.2** — gap.md format:
  - [ ] Header: `# gap.md — MYAIOS Gap Registry | Last updated: [date]`
  - [ ] Each entry format:
    ```
    ## [SEVERITY] Gap [N]: [Title]
    **Category:** Architectural / Capability / Integration / Philosophy
    **Description:** [what is missing or wrong]
    **Fix:** [specific action to resolve]
    **Status:** open / resolved ([date])
    ```
  - [ ] Severity levels: CRITICAL / HIGH / MEDIUM / LOW
  - [ ] Resolved gaps marked `[RESOLVED — date]` not deleted

- [ ] **6.C.3** — gap-detector-skill:
  - [ ] Input: audit-skill output + DB issues (open) + recent log errors
  - [ ] Synthesizes all into gap.md updates
  - [ ] Does NOT delete resolved gaps — marks them resolved
  - [ ] Updates gap.md timestamp header
  - [ ] Output: `{gaps_added: int, gaps_resolved: int, gaps_still_open: int}`

- [ ] **6.C.4** — gap.md seeded with the 11 known gaps from README_BUILDING_AND_GAPS.md
      (update as they are resolved through the build process)

---

### ▶ PHASE 7: TIER 2–3 COGNITIVE AND DOMAIN SKILLS
**Gate:** All Tier 2 cognitive skills built + template library complete + at least 3 Tier 3 skills.

#### 7.A — Tier 2 Cognitive Skills

**7.A.1 — second-brain-query-skill**
- [ ] Goal: query wiki/ for context injection — return structured knowledge block with citations
- [ ] Protocol: index.md → coordinates → read pages → return knowledge block
- [ ] Token budget enforced: ≤2000 tokens returned
- [ ] Output includes: `{topic, key_claims, contradictions, open_questions, source_citations, pages_read}`
- [ ] Post-query prompt: "Should I file this synthesis as `wiki/queries/[slug].md`?"
- [ ] Never loads entire wiki — only pages pointed to by index.md for the query

**7.A.2 — context-update-skill**
- [ ] Goal: update dynamic context mid-session when workflow output changes current state
- [ ] Updates `context/dynamic.md` only (never modifies `context/static.md`)
- [ ] Writes log entry noting what was updated and why
- [ ] Returns: `{updated_fields: [string], timestamp}`

**7.A.3 — context-judge-skill**
- [ ] Goal: evaluate whether assembled context is accurate, complete, and not polluted
- [ ] Checks three failure modes (from README 4):
  - [ ] Context Pollution: is irrelevant content crowding out signal?
  - [ ] Context Explosion: is token budget being exceeded?
  - [ ] Context Amnesia: is important persistent context missing?
- [ ] Output: `{accurate: bool, failure_modes_detected: [string], gaps: [string], staleness_warnings: [string], recommendation}`

#### 7.B — Tier 3 Domain Skills

**7.B.1 — daily-report-skill**
- [ ] Uses `templates/daily-report-template.md` schema
- [ ] Reads: DB logs (today), DB issues (open), gap.md summary, MCP ingest counts
- [ ] Output: structured daily report saved to `db/reports/daily-[date].md`
- [ ] Triggered by morning-briefing CronJob

**7.B.2 — weekly-briefing-skill**
- [ ] Synthesizes 7 days of: logs, Second Brain new entries, open decisions, gap changes
- [ ] Written in user's voice (calibrates to `context/static.md` writing style section)
- [ ] Explicitly notes: what improved, what broke, what's still open

**7.B.3 — voice-draft-skill**
- [ ] Goal: write any content in the user's voice for a specified purpose
- [ ] Input: `purpose`, `content_brief`, `length_hint`, `tone_hint`
- [ ] Calibrates to `context/static.md` § Writing Voice — never generic AI voice
- [ ] Success metric: output is indistinguishable in voice from user's own writing

**7.B.4 — Additional domain skills as needed** (each must follow universal skill checklist)

#### 7.C — Template Library

> Source: README_SKILLS_TEMPLATES_WORKFLOWS.md § "TEMPLATES — Reusable Patterns"

- [ ] **7.C.1** — Execution templates in `templates/` (OS root):
  - [ ] `templates/daily-report-template.md`:
    - [ ] Schema fields: date, sessions_run, workflows_triggered, mcps_queried,
          second_brain_ingests, errors_encountered, gap_updates, open_decisions
    - [ ] Includes a realistic filled example
    - [ ] References daily-report-skill as generation_skill
  - [ ] `templates/weekly-briefing-template.md`
  - [ ] `templates/log-entry-template.md` (mirrors DB logs schema)
  - [ ] `templates/notification-template.md` (for notifier-skill output formatting)
  - [ ] `templates/audit-report-template.md`

- [ ] **7.C.2** — Each template has YAML front-matter:
  - [ ] `name`, `type`, `domain`, `generation_skill`, `version`
  - [ ] Body: schema section + realistic filled example

---

### ▶ PHASE 8: SUBSTRATE / FUSION LAYER
**ENTRY GATE — must all pass before building any Phase 8 skill:**
- [ ] ≥5 skills across Tiers 1-3 have been functional for ≥2 weeks
- [ ] DB logs table has ≥100 real entries
- [ ] DB context_patterns table has ≥10 patterns from reverse-engineered context
- [ ] audit-skill passes with ZERO CRITICAL failures
- [ ] `fusion-skill-template-workflow.idea` is filled in (Phase 0)
- [ ] You have read the Fusion Layer section of README 3 again today

#### 8.A — skill-judge-skill (Tier 4)

- [ ] Full universal skill checklist applied
- [ ] Input schema (exact):
  - [ ] `skill_name: string`
  - [ ] `skill_md: string` (full file content)
  - [ ] `recent_outputs: [{ output, timestamp, workflow_id }]` (last N from DB)
  - [ ] `success_metrics: [string]` (parsed from skill.md)
- [ ] Output schema (exact):
  - [ ] `score: float` (0.0–1.0)
  - [ ] `metric_breakdown: [{ metric: string, passed: bool, note: string }]`
  - [ ] `failure_patterns: [string]`
  - [ ] `recommended_changes: [{ field: string, current_value: string, proposed_value: string, reasoning: string, conservative: bool }]`
  - [ ] `verdict: "pass" | "update_recommended" | "update_required"`
- [ ] Constraints enforced in system prompt:
  - [ ] Only proposes changes where `conservative: true`
  - [ ] Never recommends expanding skill scope beyond stated goal
  - [ ] Every recommended_change has reasoning citing specific output evidence

#### 8.B — pattern-learner-skill (Tier 4)

- [ ] Full universal skill checklist applied
- [ ] Input: evaluator outputs + DB log entries (last 30 days) + DB context_patterns
- [ ] Output (required fields — all must be present):
  - [ ] `patterns_found: [string]`
  - [ ] `improvement_perspectives: [{ skill_name, perspective, tradeoffs, evidence_citations: [log_id] }]`
  - [ ] `proposed_skill_changes: [{ skill_name, field, change, justification }]`
- [ ] **HARD REQUIREMENT:** `tradeoffs` field is required in every improvement_perspective
  - [ ] If tradeoffs field is empty or missing → proposal is rejected by skill-update-skill
  - [ ] No improvement without stated cost — this prevents unconstrained optimization
- [ ] Runs on weekly cadence (not per-workflow; added to `weekly-sb-lint` CronJob or dedicated cron)

#### 8.C — skill-update-skill (Tier 4)

- [ ] Full universal skill checklist applied
- [ ] Input: `approved_changes` (from human approval gate), `target_skill_path`
- [ ] Steps:
  - [ ] Step 1: Read current skill.md at target path
  - [ ] Step 2: Apply each approved change to the correct field
  - [ ] Step 3: Determine version bump (MINOR if no interface change; MAJOR if I/O format changed)
  - [ ] Step 4: Write new skill.md
  - [ ] Step 5: Append to CHANGELOG.md
  - [ ] Step 6: Run test cases from `tests/test-cases.md`
  - [ ] Step 7a: If tests pass → write deployment log entry → return `{deployed: true}`
  - [ ] Step 7b: If tests fail → revert to previous version → write log entry → trigger notifier → return `{deployed: false, reverted: true}`

#### 8.D — skill-improvement-cycle Workflow

- [ ] `workflows/skill-improvement-cycle.yaml` exists
- [ ] Nodes in order:
  - [ ] Node 1: `run-judge` — calls skill-judge-skill on target skill
  - [ ] Node 2: `run-learner` — calls pattern-learner-skill with judge output + recent logs
  - [ ] Node 3: `human-approval` — **approval gate node**:
    - [ ] Shows: current skill.md content
    - [ ] Shows: proposed changes with evidence citations
    - [ ] Shows: tradeoffs for each proposal
    - [ ] Captures: approved_changes (human selects which to apply)
    - [ ] If rejected entirely: workflow exits here; nothing applied
  - [ ] Node 4: `apply-update` — calls skill-update-skill with approved_changes
  - [ ] Node 5: `log-write` — final node
- [ ] **The approval gate is non-bypassable** — no parameter or flag can skip it

#### 8.E — Phase 8 Milestone: First Self-Improvement

- [ ] **8.E.1** — Select the oldest Tier 1-3 skill with most log history
- [ ] **8.E.2** — Run skill-improvement-cycle:
  - [ ] Judge runs → score < 1.0 (any real skill will have room for improvement) ✓
  - [ ] Pattern learner produces ≥1 perspective with tradeoffs stated ✓
  - [ ] Human approval gate presents evidence clearly ✓
  - [ ] You review, select 1-2 changes to approve ✓
  - [ ] skill-update-skill applies changes → tests pass → new version deployed ✓
  - [ ] Log entry records the cycle with before/after version numbers ✓
- [ ] **8.E.3** — Over the next week, verify the updated skill performs better on its metrics
- [ ] **→ V1.0 MILESTONE. MYAIOS improves itself with your oversight.**

---

## ══════════════════════════════════════════════════════
## PART D: CROSS-CUTTING CHECKS
## Apply these at EVERY phase. Not optional. Not periodic.
## ══════════════════════════════════════════════════════

### D.1 — After Every Component Built (the wiring check)

- [ ] Component is listed in CLAUDE.md same session it was built
- [ ] CLAUDE.md version incremented
- [ ] CLAUDE.md is still ≤300 lines after update
- [ ] No implementation detail has crept into CLAUDE.md
- [ ] The new component's trigger conditions are unique — no overlap with existing skills
- [ ] If it closes a gap: gap.md updated with `[RESOLVED]` status

### D.2 — After Every Skill Execution (the log check)

- [ ] log-write-skill was called
- [ ] Log entry summary answers all three: what happened? what changed? did it succeed?
- [ ] log.md was appended (not overwritten — check line count increases)
- [ ] DB logs table has new record
- [ ] If errors occurred: errors field is populated, notifier was triggered

### D.3 — Before Every External Write (the trust boundary check)

> Source: README_ARCHITECTURE_AND_LAYERS.md § "The Trust Boundary Map"

- [ ] Write operation goes through an MCP skill (never direct API calls in a workflow node)
- [ ] Consequential writes (post, commit, send) have approval gates in the workflow
- [ ] `Fresh_SecondBrain/raw/` is not the target of any write
- [ ] `.env` contents are not being logged, displayed, or included in any output
- [ ] The MCP skill explicitly returns errors rather than failing silently

### D.4 — Weekly (the organism health check)

- [ ] All 5 CronJobs ran in the past 7 days (check DB cronjobs table)
- [ ] log.md has grown (new entries this week)
- [ ] `wiki/` has new or updated pages this week
- [ ] gap.md has been reviewed and updated if needed
- [ ] Run `scripts/test_mcps.py` — all MCPs healthy
- [ ] Run audit-skill — no new CRITICAL failures

### D.5 — Monthly (the evolution check)

- [ ] OS CLAUDE.md still accurately describes the current system (no drift)
- [ ] `context/static.md` still accurately describes you (projects change)
- [ ] `wiki/synthesis/overview.md` reflects current understanding (not stale)
- [ ] DB `issues` table has no issues open >30 days without a note
- [ ] At least one skill has been reviewed by skill-judge-skill (Phase 8+)
- [ ] Check: is the gap between what you know and what MYAIOS knows narrowing?

---

## ══════════════════════════════════════════════════════
## PART E: FILE CONTENT QUALITY STANDARDS
## What "good" looks like for each file type.
## ══════════════════════════════════════════════════════

### E.1 — What a Good skill.md Looks Like

**Smells of a bad skill.md:**
- Goal is a paragraph, not a sentence
- System prompt says "be helpful" or "try to"
- Success metrics are subjective ("output is good quality")
- No edge cases listed
- Output format has no `errors` field
- `last_tested` is blank or >30 days ago

**Markers of a good skill.md:**
- Goal is exactly one sentence with a measurable outcome
- System prompt is 3-8 sentences, each describing a specific behavior
- Every success metric is binary (passed/failed) and testable programmatically
- Edge cases describe real scenarios that will actually occur
- Output format schema is tight enough that a downstream skill knows exactly what to expect
- `trigger_conditions` are specific enough to not activate accidentally

### E.2 — What a Good Workflow YAML Looks Like

**Smells of a bad workflow:**
- No `log-write` as final node
- Loop node without `max_iterations`
- Approval gates only at the end (not at decision points)
- Monolithic prompt nodes that do everything in one step
- No error propagation between nodes

**Markers of a good workflow:**
- Every node's output is consumed by exactly one downstream node
- The FAILED path is as explicit as the SUCCESS path
- A human reading the YAML can trace the data flow without running it
- Loop nodes have clear termination conditions
- log-write final node has a specific summary format — not just "workflow complete"

### E.3 — What a Good wiki Page Looks Like

**Smells of a bad wiki page:**
- More than 2000 words (lint flag)
- Sources cited by URL only (no compiled summary)
- No connection to related concepts or entities
- Contradictions absent (no real domain has zero internal tensions)
- Last updated date is from initial ingest (never enriched)

**Markers of a good wiki page:**
- Self-contained: readable without needing to follow links
- All claims are attributed to specific sources
- Explicitly notes what is uncertain or disputed
- Has a "Related" section pointing to other wiki pages
- Has grown through multiple ingest cycles (reflects accumulation)

### E.4 — What Good Context Looks Like at Session Start

**Smells of bad context:**
- Static context wasn't loaded (session starts without knowing who it's serving)
- Dynamic context is just "today is [date]" with no real activity summary
- Second Brain section is empty or says "wiki not searched"
- Context total tokens exceed window limits

**Markers of good context:**
- Static section reads like a genuine identity brief
- Dynamic section contains actual recent activity (specific workflows, specific issues)
- Second Brain section contains compiled page excerpts (not just page titles)
- Reverse-engineered section has at least one behavioral observation ("you typically do X on Mondays")
- Total fits within the model's effective context window with room for the actual task

### E.5 — What a Good Log Entry Looks Like

**Bad:**
> `workflow ran`

**Bad:**
> `nightly-wiki-ingest completed successfully`

**Good:**
> `nightly-wiki-ingest processed 3 files from raw/pending/ — created wiki/sources/paper-attention-is-all-you-need.md, updated wiki/concepts/transformer-architecture.md, flagged 1 contradiction in open-questions.md — no errors`

The good log entry can reconstruct context for a future session that has never seen this conversation.

---

## ══════════════════════════════════════════════════════
## PART F: SEMANTIC AND BEHAVIORAL QUALITY CHECKS
## These cannot be automated. Human judgment required.
## Run quarterly or after major build milestones.
## ══════════════════════════════════════════════════════

### F.1 — The Voice Test
Start a fresh session. Ask: "Draft a brief response to this email: [paste a real email]."
Does the output sound like YOU wrote it — your vocabulary, your level of formality, your
characteristic way of opening? If not: audit `context/static.md` § Writing Voice and
recalibrate voice-draft-skill.

### F.2 — The Continuity Test
Start a session. Say nothing about your projects. Ask: "What should I be working on this week?"
Does MYAIOS answer from your actual projects, your actual priorities, your actual open decisions?
If it gives a generic response: context amnesia has occurred. Diagnose: was context-load-skill
called? Did dynamic context assemble correctly?

### F.3 — The Knowledge Test
Ask a question your Second Brain contains the answer to (something from an ingested paper or
article). Does the answer cite your compiled wiki? Does it surface open questions from
`open-questions.md`? Does it feel like YOUR understanding of the topic, not a generic answer?
If not: audit second-brain-query-skill and the context-load-skill's Second Brain section.

### F.4 — The Self-Awareness Test
Introduce a deliberate inconsistency: spell a skill name slightly wrong in CLAUDE.md.
Run audit-skill. Does it catch the orphan? Does it trigger notifier? Does gap.md get updated?
If not: audit-skill needs strengthening or is not being triggered properly.

### F.5 — The Aliveness Test
Without doing anything, check Monday morning:
- Did the nightly CronJob run last night? (Check DB cronjobs.last_run)
- Did any wiki ingest happen this week? (Check `Fresh_SecondBrain/log.md`)
- Did the health check run in the last 6 hours? (Check DB)
- Is the log.md file growing? (Check last 10 lines)
If answers are no: the organism is not alive. The heartbeat stopped. Check CronJob runner.

### F.6 — The Organism Test
Ask yourself: "Does MYAIOS feel like a living system or a file system?"
A living system: acts unprompted, remembers across time, improves itself, flags what needs you.
A file system: sits inert until asked, forgets everything, never changes.
The test is the CronJobs. If they're running, it's alive. If they're not, it's a file system.

### F.7 — The Narrowing Gap Test (Long-Term)
Every month, ask: is the gap between what I know and what MYAIOS knows smaller than last month?
Indicators: more wiki pages, richer concepts, better reverse-engineered context.
If gap is widening: ingest pipeline is broken or Second Brain is not being queried in context.

### F.8 — The Precision Test
Pick any action in the OS. Trace it from trigger to output to log. Can you follow every step
without ambiguity? Is every decision point explicit? Every handoff clean?
If you hit ambiguity: that is where a future failure will occur. Write it into gap.md now.

---

## ══════════════════════════════════════════════════════
## PART G: ANTI-PATTERN WATCHLIST
## Conditions the agent must detect and refuse to create.
## ══════════════════════════════════════════════════════

| Anti-Pattern | Signature | Why Catastrophic |
|---|---|---|
| **God Skill** | goal is multiple sentences; skill does many things | Untestable; outputs unpredictable; unfixable |
| **CLAUDE.md Prose Creep** | CLAUDE.md describes HOW skills work | Hits 300-line cap; becomes unnavigable |
| **Silent MCP Failure** | MCP skill catches error but returns empty output | Notifier never fires; failures invisible |
| **Orphan Component** | Built but not wired to CLAUDE.md | Dead tissue; never invoked |
| **Wireless Build** | Component added; CLAUDE.md not updated same session | System drifts from its own self-description |
| **Vanilla Wiki** | No contradictions flagged after dozens of ingests | False confidence; real knowledge has tensions |
| **Log Desert** | Workflow runs; no log entry written | Pattern learner starved; audit blind; history lost |
| **Horizontal Build** | 10 half-finished skills | Apparent richness; nothing reliable |
| **Tradeoff-Free Proposal** | Pattern learner proposes without stating tradeoffs | Unconstrained optimization; scope creep |
| **Raw/ Violation** | Any skill writes to `Fresh_SecondBrain/raw/` | Destroys reproducibility of compiled knowledge |
| **Context Serving Everyone** | Static context is generic; could be anyone | MYAIOS serves nobody specifically |
| **Approval Bypass** | skill-update-skill runs without human approval | System drifts from human intent |
| **Missing Error Field** | Skill output format has no `errors` field | Callers cannot detect or handle failure |
| **Version-Free Skill** | skill.md has no version; CHANGELOG.md missing | Cannot track what changed or why |
| **Stale AUDIT.md** | Audit.md not updated when new components added | Audit-skill compares against wrong ground truth |

---

## ══════════════════════════════════════════════════════
## PART H: COMPLETION STATES
## The observable evidence that each milestone is real.
## ══════════════════════════════════════════════════════

### MV-MYAIOS — Proof of Concept
```
The evidence:
  □ You went to sleep.
  □ A CronJob fired while you slept.
  □ log.md has an entry you didn't write.
  □ wiki/ has a page you didn't create.
  □ You wake up: the system ran without you. It worked.
  □ You didn't prompt it. You didn't trigger it. It happened.

This is the proof of concept. Everything else is expansion.
```

### V1.0 — The OS Knows You
```
The evidence:
  □ You start a session. You say nothing about your projects.
  □ You ask: "What should I focus on today?"
  □ MYAIOS answers from your actual projects, your actual priorities.
  □ No briefing required. Context was already there.
  □ The Second Brain answer cites your compiled papers, not general knowledge.
  □ The OS flags two things that need your attention before you ask.
  □ This session felt like continuing a conversation, not starting one.
```

### V3.0 — The Second Instance
```
The evidence:
  □ A skill has been through the Fusion Layer and improved.
  □ You have the before/after versions and the log evidence of improvement.
  □ The pattern learner has updated context assembly based on your behavior.
  □ Your Telegram shows: 3 decisions this week that MYAIOS surfaced before you noticed.
  □ You asked a complex question. The answer was indistinguishable from your own thinking.
  □ MYAIOS has been running for 6 months. It has learned who you are.
  □ The gap between what you know and what MYAIOS knows is measurably smaller.
  □ It is not a tool you use. It is a cognitive environment you inhabit.
```

---

*This checklist is a living document.*
*Update it when: architecture evolves, a new component type is introduced,*
*an anti-pattern is discovered, a phase gate reveals gaps.*

*Anchored to:*
*README_VISION_AND_SOUL.md · README_ARCHITECTURE_AND_LAYERS.md ·*
*README_SKILLS_TEMPLATES_WORKFLOWS.md · README_MEMORY_CONTEXT_SECONDBRAIN.md ·*
*README_BUILDING_AND_GAPS.md*

*Version: 2.0 | Full synthesis rebuild.*
