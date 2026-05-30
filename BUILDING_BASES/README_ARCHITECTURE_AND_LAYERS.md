# README_ARCHITECTURE_AND_LAYERS.md
## How MYAIOS Works — The Layers, The Components, and The Data Flow

> **What this README is for:** The technical blueprint of the MYAIOS organism. Every layer,
> every component, every data flow, every trust boundary — named and shown. A developer
> reading this once should be able to draw the system from memory. A builder should be able
> to place any component or question into its correct layer immediately.

---

## The Seven Layers of MYAIOS

<!-- NEW → Layer names derived from the organism metaphor (Substrate, Nervous System, etc.)
rather than generic technical names. 7 layers (not 6) adopting Minimax's substrate layer
insight, with MYAIOS-specific naming coined in synthesis -->

MYAIOS has seven layers. They are not equal — lower layers are more fundamental and must
exist before upper layers can function. Each layer has a name that is specific to this
organism, not borrowed from generic architecture vocabulary.

```
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 7: VOICE                                                      │
│  How the OS speaks outward. Notifier (Telegram, Gmail).              │
│  Dashboard. CLI. Alerts, reports, escalations.                       │
│  Governing .idea: notifier.idea                                      │
├─────────────────────────────────────────────────────────────────────┤
│  LAYER 6: AWARENESS                                                  │
│  How the OS sees itself. Audit system. Gap detector. AUDIT.md.       │
│  gap.md. Health monitors. The self-model layer.                      │
│  Governing .idea: audit.idea, gap.idea                               │
├─────────────────────────────────────────────────────────────────────┤
│  LAYER 5: EXECUTION                                                  │
│  How the OS acts. Skills (atomic). Templates (patterns).             │
│  Workflows (pipelines). CronJobs (heartbeat).                        │
│  Governing .idea: skills.idea, workflows.idea, cronjobs.idea,        │
│                   fusion-skill-template-workflow.idea                │
├─────────────────────────────────────────────────────────────────────┤
│  LAYER 4: COGNITION                                                  │
│  How the OS thinks. CLAUDE.md (the nervous system router).           │
│  MCP connections (sensory inputs). Context Manager (4 types fused).  │
│  System prompts.                                                     │
│  Governing .idea: claudemd.idea, mcps.idea, context.idea             │
├─────────────────────────────────────────────────────────────────────┤
│  LAYER 3: MIND                                                       │
│  What the OS knows. Second Brain (compiled wiki). Database.          │
│  Log system. Artifacts. The permanent memory of the organism.        │
│  Governing .idea: second-brain.idea, db.idea, log.idea               │
├─────────────────────────────────────────────────────────────────────┤
│  LAYER 2: NERVOUS SYSTEM                                             │
│  How the OS perceives the world. MCP servers — GitHub, Slack,        │
│  Telegram, Google Workspace, filesystem, browser. The sensory        │
│  organs that connect the OS to external reality.                     │
│  Governing .idea: mcps.idea                                          │
├─────────────────────────────────────────────────────────────────────┤
│  LAYER 1: SUBSTRATE                                                  │
│  The DNA. Self-improving skills — the pattern learner and skill      │
│  evaluator — that read the log, judge performance, and propose       │
│  improvements to every layer above. The immune system and            │
│  evolutionary engine of the organism.                                │
│  Governing .idea: skills.idea (pattern learning section)             │
└─────────────────────────────────────────────────────────────────────┘
```

The dependency graph is not strictly top-down. Layer 1 (Substrate) reads the outputs of
Layer 5 (Execution) and Layer 3 (Mind) to do its job. The organism has feedback loops, not
just a call stack.

---

## Full Architecture Diagram

```
╔══════════════════════════════════════════════════════════════════════╗
║                       MYAIOS ARCHITECTURE                           ║
╠══════════════════════════════════════════════════════════════════════╣
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │                  YOU (The Architect)                        │    ║
║  │      Goals · Values · Taste · Approval · Direction         │    ║
║  └────────────────────────┬────────────────────────────────────┘    ║
║                           │ intent, commands, approvals             ║
║  ┌────────────────────────▼────────────────────────────────────┐    ║
║  │  LAYER 4/COGNITION: CLAUDE.md — The Nervous System Router   │    ║
║  │  • Wires every component to every other component           │    ║
║  │  • Routes every request to the right skill/workflow         │    ║
║  │  • Deterministic instruction layer — not aspirational        │    ║
║  │  • Evolves only via consultation with you                   │    ║
║  │  • TWO instances: OS CLAUDE.md + Fresh_SecondBrain/CLAUDE.md│    ║
║  └──────────┬──────────────────────┬───────────────────────────┘    ║
║             │                      │                                 ║
║  ┌──────────▼──────┐    ┌──────────▼──────────────────────────┐    ║
║  │  L2: NERVOUS    │    │       L4/COGNITION: CONTEXT          │    ║
║  │  SYSTEM (MCPs)  │    │  ┌─────────────┐ ┌───────────────┐  │    ║
║  │                 │    │  │ STATIC      │ │ DYNAMIC       │  │    ║
║  │  github         │    │  │ (who you   │ │ (what's       │  │    ║
║  │  slack          │    │  │  are, your │ │  active now,  │  │    ║
║  │  telegram       │    │  │  goals,    │ │  today's      │  │    ║
║  │  google-ws      │    │  │  methods)  │ │  priorities)  │  │    ║
║  │  filesystem     │    │  └─────────────┘ └───────────────┘  │    ║
║  │  browser        │    │  ┌─────────────┐ ┌───────────────┐  │    ║
║  │  + 4 more       │    │  │ SECOND      │ │ REVERSE-      │  │    ║
║  │                 │    │  │ BRAIN       │ │ ENGINEERED    │  │    ║
║  │ .env ─► config  │    │  │ (compiled  │ │ (inferred     │  │    ║
║  │ test_mcps.py    │    │  │  wiki      │ │  from log     │  │    ║
║  │                 │    │  │  queries)  │ │  history)     │  │    ║
║  └────────┬────────┘    │  └─────────────┘ └───────────────┘  │    ║
║           │             │              │                         │    ║
║           │             │       CONTEXT FUSION SKILL            │    ║
║           │             │              ▼                         │    ║
║           │             │       LIVE CONTEXT WINDOW             │    ║
║           │             └────────────────────────────────────────┘   ║
║           │                                                           ║
║           │             ┌────────────────────────────────────────┐   ║
║           └────────────▶│       L3: MIND (Memory Systems)        │   ║
║                         │                                        │   ║
║                         │  ┌──────────────────────────────────┐ │   ║
║                         │  │  SECOND BRAIN (Compiled Wiki)    │ │   ║
║                         │  │  Fresh_SecondBrain/              │ │   ║
║                         │  │  raw/ (IMMUTABLE sources)        │ │   ║
║                         │  │  wiki/ (compiled: entities,      │ │   ║
║                         │  │         concepts, sources,       │ │   ║
║                         │  │         synthesis, queries)      │ │   ║
║                         │  │  index.md · log.md · CLAUDE.md   │ │   ║
║                         │  └──────────────────────────────────┘ │   ║
║                         │  ┌──────────────────────────────────┐ │   ║
║                         │  │  DATABASE (Operational Record)   │ │   ║
║                         │  │  logs · reports · issues          │ │   ║
║                         │  │  second_brain_ingests            │ │   ║
║                         │  │  mcp_ingests (raw + processed)   │ │   ║
║                         │  │  cronjobs                        │ │   ║
║                         │  └──────────────────────────────────┘ │   ║
║                         └──────────────┬───────────────────────┘    ║
║                                        │                             ║
║                         ┌──────────────▼───────────────────────┐    ║
║                         │       L5: EXECUTION ENGINE            │    ║
║                         │                                       │    ║
║                         │  SKILLS ──────────────────────┐      │    ║
║                         │  (atomic, versioned, tested)  │      │    ║
║                         │                               │      │    ║
║                         │  TEMPLATES ───────────────────┼────► │    ║
║                         │  (cognitive/output patterns)  │      │    ║
║                         │                               │  FUSION   ║
║                         │  WORKFLOWS ───────────────────┘  LAYER   ║
║                         │  (composed DAG pipelines)          │      ║
║                         │                               ◄────┘      │    ║
║                         │  CRONJOBS ─────────────────────────►      │    ║
║                         │  (scheduled triggers)           SKILL     │    ║
║                         │                               EVALUATOR   │    ║
║                         └──────────────┬───────────────────────┘    ║
║                                        │ outputs feed substrate      ║
║                         ┌──────────────▼───────────────────────┐    ║
║                         │       L1: SUBSTRATE (Self-Improvement)│    ║
║                         │                                       │    ║
║                         │  PATTERN LEARNER SKILL                │    ║
║                         │  "Read the log. Find behavioral        │    ║
║                         │   patterns. Propose controlled        │    ║
║                         │   improvements. Human approves."      │    ║
║                         │                                       │    ║
║                         │  SKILL EVALUATOR SKILL                │    ║
║                         │  "Judge each skill's outputs against  │    ║
║                         │   its success metrics. Flag drift.    │    ║
║                         │   Recommend updates."                 │    ║
║                         └──────────────┬───────────────────────┘    ║
║                                        │                             ║
║            ┌───────────────────────────▼──────────────────────┐     ║
║            │  L6: AWARENESS (Self-Model)                       │     ║
║            │  Audit system · AUDIT.md · gap.md · health checks │     ║
║            └───────────────────────────┬──────────────────────┘     ║
║                                        │ alerts                      ║
║            ┌───────────────────────────▼──────────────────────┐     ║
║            │  L7: VOICE (Communication)                        │     ║
║            │  Telegram · Gmail · Dashboard · CLI               │     ║
║            └───────────────────────────┬──────────────────────┘     ║
║                                        │                             ║
║                          ┌─────────────▼──────────────┐             ║
║                          │  YOU (The Loop Closes)      │             ║
║                          │  Respond · Approve · Direct │             ║
║                          └────────────────────────────┘             ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## The Trust Boundary Map

<!-- NEW → Trust Boundary Map as specified by the synthesis prompt — not produced by either
prior model, defines what is inside the OS vs outside and what crosses the line -->

The OS has a boundary. Understanding what is inside, outside, and crossing that boundary is
essential for security, reliability, and architectural coherence.

```
INSIDE THE OS BOUNDARY
─────────────────────
  CLAUDE.md (both instances)
  All skills, templates, workflows
  Database (all tables)
  Fresh_SecondBrain/wiki/ (LLM-owned)
  Context files (static and dynamic)
  Log system (log.md + DB logs)
  Audit system and gap.md
  CronJob definitions

OUTSIDE THE OS BOUNDARY
────────────────────────
  GitHub (external service)
  Slack, Telegram (external services)
  Google Workspace (external service)
  The web (browser/Playwright)
  Your local filesystem (boundary is at MCP interface)
  Fresh_SecondBrain/raw/ (IMMUTABLE — outside OS write access)

CROSSING THE BOUNDARY (MCP interface)
──────────────────────────────────────
  DATA INBOUND: MCP skills pull external data → structured records → DB
  DATA OUTBOUND: Notifier pushes alerts → Telegram/Gmail
  WRITE OUTBOUND: Workflow nodes post via Slack MCP, commit via GitHub MCP
  KNOWLEDGE INBOUND: Sources dropped in raw/pending/ → compiled into wiki/

  WHAT NEVER CROSSES:
  - raw/ is read-only for the OS (immutability rule)
  - Credentials (.env) never leave the OS or appear in output
  - The OS never modifies external services without a skill with explicit
    approval gates for consequential actions (post, commit, send)
```

---

## CLAUDE.md: The Nervous System (Anatomy)

<!-- NEW → Structural sketch of what OS CLAUDE.md should contain — neither prior model
produced this, yet it's the most important file in the system -->

Neither prior version showed what OS CLAUDE.md actually looks like. Here is its required
structure:

```markdown
# CLAUDE.md — MYAIOS Operating System
Version: [N] | Updated: [date]

## Identity
[Who you are, what this OS serves, one paragraph]

## Directory Map
[Every folder, what it contains, when it changes]

## Available Skills
[Exact skill names + one-line purpose for each]
[Updated every time a skill is added]

## Available Workflows
[Exact workflow names + trigger types]

## MCP Connections
[MCP name → config location → test script reference]

## Context Loading Protocol
[How to load context at session start: which files, in which order]

## Log Writing Protocol
[When and how to call log-write-skill — mandatory for every action]

## Escalation Protocol
[What triggers a notifier call: errors, decisions needed, audit failures]

## Evolution Protocol
[How CLAUDE.md itself can be modified: only via admin consultation,
 only after component testing, with version increment]
```

This file must never exceed ~300 lines. Implementation detail belongs in skill.md files —
CLAUDE.md carries only the wiring map.

---

## MCP Integration Philosophy

The MCPs are not add-ons. They are the sensory organs of the organism (Layer 2).

From mcps.idea, the key philosophical rule: **skills for MCPs are not static.** The skill
evaluator judges them, the pattern learner proposes improvements, and they evolve with usage.
An MCP skill that fetches GitHub data the same way in year 2 as it did in week 1 has failed
to learn what data matters most.

MCP integration philosophy:
- Each MCP has one dedicated skill (minimum). Its job: test connectivity, process the source,
  trigger new-data detection.
- Combination skills exist for workflows that need multiple MCPs simultaneously.
- MCP skills explicitly return errors rather than failing silently — the notifier depends on
  this.
- The `config.json` governs MCP settings. The `.env` governs credentials. `test_mcps.py` is
  permanent infrastructure — it lives forever and is run whenever something behaves unexpectedly.

---

## Failure Modes

<!-- NEW → Failure Modes section as specified by synthesis prompt — neither prior model
addressed this systematically -->

**What breaks first when the architecture is stressed:**

| Failure | Cause | Effect | Detection |
|---|---|---|---|
| Context Pollution | Context fusion includes too much irrelevant content | LLM answers drift; skill outputs become unfocused | Pattern learner flags drift in outputs |
| Context Explosion | Wiki grows without query precision | Session startup slow; context window overwhelmed | Lint protocol detects bloated pages |
| CLAUDE.md Drift | New components added but CLAUDE.md not updated | Routing breaks; new skills invisible | Audit system catches missing wire |
| Log Gap | log-write-skill fails silently | Pattern learner loses data; audit cannot verify | Periodic log continuity check |
| Skill Scope Inflation | "Just one more thing" added to existing skill | Token waste; hallucination rate rises | Skill evaluator flags metric degradation |
| MCP Credential Expiry | API token expires | All workflows using that MCP fail silently | MCP health check cronjob + notifier |
| Substrate Runaway | Pattern learner proposes too many changes | Core behavior drifts from intent | Human approval gate on all changes |

The audit system (Layer 6) is the primary detection mechanism for all failures above. Build
it early, before you need it.

---

## Complete Data Flow: A Full Request Lifecycle

```
1. TRIGGER
   ├── Manual: you speak or type a request
   ├── Scheduled: CronJob fires at 23:00
   └── Event: MCP detects new data
         │
2. CLAUDE.md RECEIVES AND ROUTES
   Routes to: which skill(s)? which workflow? which context?
         │
3. CONTEXT MANAGER ASSEMBLES
   Static: who you are, your goals, your methods
   Dynamic: today's date, active projects, recent events
   Second Brain: relevant compiled wiki pages (queried, not loaded)
   Reverse-engineered: behavioral patterns from log history
   [Token budget enforced per type]
         │
4. LLM RECEIVES FUSED CONTEXT + INSTRUCTION
   Executes via the relevant skill or workflow
         │
5. MCPs ACT ON EXTERNAL WORLD (if needed)
   Read: GitHub issues, Slack messages, Gmail
   Write: Post to Slack, commit to GitHub, send notification
         │
6. RESULT PRODUCED
   Output written to appropriate destination:
   → wiki/sources/ (if new knowledge)
   → reports/ (if synthesized artifact)
   → context update (if dynamic context changed)
         │
7. LOG WRITTEN (mandatory)
   log-write-skill: {timestamp, workflow_id, stage, summary, errors, db_refs}
   DB updated simultaneously
         │
8. NOTIFIER FIRES (conditional)
   Only if: error, decision needed, audit failure, major event
         │
9. SUBSTRATE READS (asynchronous)
   Pattern learner reads log on weekly cadence
   Skill evaluator reads outputs on per-workflow cadence
   Proposes improvements → requires human approval
         │
10. CLAUDE.md EVOLVES (per-iteration)
    If new component added: CLAUDE.md updated and re-tested
    If behavior change confirmed: version incremented
    [The loop closes. The organism grows.]
```

---

## Integration Points

| Integration | Direction | Technology | Layer |
|---|---|---|---|
| GitHub | Bidirectional | MCP | L2 Nervous System |
| Slack | Bidirectional | MCP | L2 Nervous System |
| Telegram | Outbound primary | MCP | L2 + L7 Voice |
| Gmail | Bidirectional | MCP | L2 Nervous System |
| Google Drive | Bidirectional | MCP | L2 Nervous System |
| Web/Browser | Outbound | MCP (Playwright) | L2 Nervous System |
| Local filesystem | Bidirectional | MCP | L2 Nervous System |
| You (human) | Bidirectional | Notifier + CLI | L7 Voice |

---

*Cross-references: README_VISION_AND_SOUL.md (why these layers exist),
README_SKILLS_TEMPLATES_WORKFLOWS.md (Layer 5 Execution in detail),
README_MEMORY_CONTEXT_SECONDBRAIN.md (Layer 3 Mind in detail),
README_BUILDING_AND_GAPS.md (build order for these layers).*

*Part of MYAIOS backbone documentation. v1.0 — Synthesis.*
