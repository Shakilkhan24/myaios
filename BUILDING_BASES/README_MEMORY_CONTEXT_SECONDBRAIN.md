# README_MEMORY_CONTEXT_SECONDBRAIN.md
## The Mind of MYAIOS — Memory Architecture, Context Management, and The Second Brain

> **What this README is for:** The most philosophically dense document in the backbone.
> It defines what "memory" means in an AI operating system, how three distinct memory systems
> work together, why compiled knowledge beats retrieved knowledge always, and what it means
> for a log to be a form of consciousness. The cognitive science matters as much as the
> implementation here.

---

## Three Memory Systems, Three Purposes

MYAIOS has three memory systems. Each serves a distinct cognitive role. Conflating them is
the most common architectural mistake — and it leads to: context explosion, stale knowledge,
invisible operational history.

```
┌─────────────────────────────────────────────────────────────────────┐
│  CONTEXT MANAGER                                                     │
│  Working memory — what the LLM sees right now, in this session      │
│  Lifespan: session         Changes: continuously during execution   │
│  Kills it: session end, token overflow, staleness threshold         │
├─────────────────────────────────────────────────────────────────────┤
│  DATABASE + LOG SYSTEM                                               │
│  Operational memory — what the OS has done and when                  │
│  Lifespan: permanent       Changes: append-only (logs), mutable (DB)│
│  Kills it: nothing (logs) / explicit deletion (DB records)          │
├─────────────────────────────────────────────────────────────────────┤
│  SECOND BRAIN (Fresh_SecondBrain/)                                   │
│  Compiled knowledge memory — what you know, pre-digested            │
│  Lifespan: permanent, compounding  Changes: ingest + lint cycles    │
│  Kills it: only an explicit wipe + rebuild (rare; never accidental) │
└─────────────────────────────────────────────────────────────────────┘
```

These three systems feed each other in one primary direction: Context reads from the Second
Brain and DB at session start. Workflows write to all three. The Second Brain does not write
to Context or DB directly. Logs are append-only — nothing erases the trail.

---

## Memory Taxonomy

<!-- NEW → Memory Taxonomy with all five fields per type (name, lifespan, storage location,
retrieval method, what kills it) — the synthesis prompt specified this; neither prior model
produced a structured table with all five fields -->

| Memory Type | Lifespan | Storage Location | Retrieval Method | What Kills It |
|---|---|---|---|---|
| **Static Context** | Months-years | Context markdown files | Read at session start | Manual update only |
| **Dynamic Context** | Hours-days | DB (dynamic_context table) + files | Generated each session from DB + MCPs | Staleness threshold + session end |
| **Second Brain Context** | Permanent | `wiki/` (compiled pages) | On-demand query via index.md | Nothing accidental; deliberate rebuild only |
| **Reverse-Engineered Context** | Days-weeks (rolling) | DB log pattern records | Pattern learner reads log; injects into context | Rolling window purge (oldest logs expire from pattern view) |
| **Operational Log** | Permanent | `log.md` + DB logs table | Direct query by audit/pattern learner | Never — append-only |
| **Compiled Wiki Knowledge** | Permanent (compounding) | `wiki/` subdirectories | index.md coordinates + skill queries | Wipe-and-rebuild only |

---

## The Four Context Types

Context is assembled fresh at the start of every meaningful interaction — not carried over
wholesale from the prior session. The assembly is the work; the result is a live context
window shaped for this exact moment.

### Type 1: Static Context

The things about you that never change, or change only when your life changes structurally.

Contents: identity, background, writing style, organizational role, long-term goals,
methodological preferences, expertise domains. For MYAIOS specifically, this includes:
your research focus (AI/ML, deep learning, architectures), your working style, the projects
you are running, the organizations you are part of.

**Storage:** markdown files. Read at every session start — always, no exceptions.
**Update trigger:** manual (you decide when this needs changing, quarterly or less).

### Type 2: Dynamic Context

The things that change from session to session, day to day, hour to hour.

Contents: today's date and time, currently active projects, this week's priorities, recent
MCP alerts, pending tasks, new wiki entries since last session, open decisions.

**Storage:** generated dynamically by querying the DB (recent logs, open issues, last
cronjob run), MCPs (pending Slack messages, calendar events), and wiki (new entries).
**Update trigger:** every session start; sometimes mid-session if a workflow produces a
significant result.
**Token budget:** enforce a hard cap (suggested: 1,000 tokens). Dynamic context that
exceeds budget must be summarized, not truncated arbitrarily.

### Type 3: Second Brain Context

Surgically-selected knowledge from the compiled wiki, relevant to the current query.

This is **not** "load the entire wiki into context." It is the minimum wiki content needed
to answer the question with citation. The index.md is the routing layer — it gives you
coordinates (which wiki pages to read), not the knowledge itself.

```
Query: "What do I know about transformer scaling laws?"
         │
         ▼
1. Read index.md → coordinates: wiki/concepts/scaling-laws.md,
                                wiki/concepts/transformer.md,
                                wiki/synthesis/open-questions.md
         │
         ▼
2. Read pointed wiki pages (pre-compiled — not raw papers)
         │
         ▼
3. Inject into context as structured knowledge block
   {topic, key_claims, contradictions, open_questions, source_citations}
         │
         ▼
4. Ask: "Should I file this synthesis as wiki/queries/[slug].md?"
```

**Token budget:** hard cap (suggested: 2,000 tokens of wiki content per query). Exceeding
this means the query is too broad — narrow it.

### Type 4: Reverse-Engineered Context

The most innovative context type — and the one most systems skip entirely.

**What it is:** context inferred from what the OS has *observed you do*, without you explicitly
stating it. The pattern learner reads the log history and produces behavioral observations
that shape future sessions.

Examples of what reverse-engineered context looks like in practice:
- "You've run the PR review workflow 8 times in the past month — always Monday morning.
  Prioritizing GitHub context for this session."
- "Your last 15 code reviews flagged test coverage issues 12 times. Leading with that."
- "You haven't touched the daily briefing workflow in 3 weeks. Deprioritizing calendar
  context unless explicitly requested."

**Storage:** pattern records in DB, generated by the pattern-learner skill.
**Update cadence:** weekly synthesis from log; injected into context as a short behavioral
observation block.
**Token budget:** hard cap (suggested: 500 tokens). Behavioral observations are summaries,
not transcripts.

---

## Context Failure Modes and Mitigations

<!-- NEW → Context failure modes with named patterns and concrete mitigations — this was
Minimax's strongest contribution and is adopted wholesale into the synthesis -->

Three catastrophic failure modes haunt AI memory systems. MYAIOS is designed to prevent all
three by naming them.

### Failure Mode 1: Context Pollution

**What it is:** Irrelevant information crowding out relevant information. The system spends
tokens on noise and misses the signal. Outputs drift; skills underperform; the LLM seems
to "forget" the task.

**How MYAIOS prevents it:**
- The context fusion skill is *selective*, not exhaustive. It assembles only what's relevant
  to the specific query or task at hand.
- Static context is pre-indexed — the fusion skill retrieves only relevant sections, not the
  full identity file.
- Second Brain queries return wiki pages (pre-compiled, focused), not raw documents.
- Token budgets are enforced per context type. Anything that doesn't fit within budget is
  summarized, not included verbatim.

**Detection:** Skill evaluator flags output quality degradation over multiple runs.

### Failure Mode 2: Context Explosion

**What it is:** The context window grows as knowledge accumulates until the system can no
longer function efficiently. More knowledge begins producing worse performance.

**How MYAIOS prevents it:**
- The Second Brain is NOT in the context window. It is a separate system that is *queried*,
  not loaded wholesale. The wiki grows without bound; the context budget stays fixed.
- index.md is a topological map — coordinates, not knowledge. It tells you where things live
  without containing the things.
- The lint protocol catches wiki pages that are becoming bloated or unfocused before they
  become retrieval problems.

**Detection:** Context assembly time increases measurably. Lint protocol flags oversized pages.

### Failure Mode 3: Context Amnesia

**What it is:** The system has no persistent memory. Every session starts from zero.
Accumulated learning evaporates. You re-explain yourself every session.

**How MYAIOS prevents it:**
- Static context files persist across sessions (read at the start of every session without
  exception).
- The Second Brain accumulates permanently (nightly ingestion never stops).
- The DB logs everything with rich metadata designed for reconstruction.
- The reverse-engineered context type exists precisely to reconstruct behavioral context from
  log history — the OS can say "here is what I know about how you work" without you telling it.
- CLAUDE.md evolves persistently — it is the one file that carries the most accumulated
  intelligence across sessions.

**Detection:** If a session requires you to re-explain your projects or preferences, context
amnesia has occurred. Check: was static context loaded? Was dynamic context assembled?

---

## The Second Brain: Compiled Knowledge Layer

### The Karpathy Insight, Made Architectural

The `Fresh_SecondBrain/` is not a folder of notes. It is the compiled knowledge layer — the
cognitive substrate the OS runs on top of. The distinction between compiled and retrieved
knowledge is the most important architectural decision in MYAIOS.

```
RAG System:   Source ──► [at query time] ──► LLM ──► Answer
                           ↑ re-derived every single time
                           ↑ grows more expensive as sources accumulate
                           ↑ O(n×m) at scale

MYAIOS Wiki:  Source ──► [at ingest time] ──► Wiki ──► [at query time] ──► Answer
                                              ↑ compiled ONCE, queried forever
                                              ↑ compilation cost is fixed and upfront
                                              ↑ O(n) compile + O(m) query, always
```

The compilation step is expensive once. Every subsequent query is free. This is why the
Second Brain compounds: each new source adds to a pre-integrated network, not to a retrieval
pile. After 200 sources, a RAG system re-reads 200 documents per query. The MYAIOS wiki has
~300 compiled pages with ~3,000 cross-references that answer faster and more coherently.

### The Two CLAUDE.md Boundary

The Second Brain has its own CLAUDE.md. The OS has its own CLAUDE.md. These govern
non-overlapping concerns, and the boundary is an architectural constraint that must be
documented and tested:

```
OS CLAUDE.md governs:
  WHEN and WHY the Second Brain is accessed
  Which skills query it
  How Second Brain context is injected into workflows

Fresh_SecondBrain/CLAUDE.md governs:
  HOW sources are ingested and compiled
  The wiki's internal structure and schema
  Contradiction protocol
  Lint protocol
  What a wiki page must contain

If both files ever issue contradictory instructions for the same action:
→ that is an architectural violation
→ audit system must catch it
→ resolve by clarifying which CLAUDE.md owns that decision domain
```

### Second Brain Structure

```
Fresh_SecondBrain/
├── raw/                    ← IMMUTABLE. LLM never writes here. Ever.
│   ├── articles/           ← web clips
│   ├── papers/             ← PDFs, arXiv, technical papers
│   ├── notes/              ← your handwritten context
│   ├── transcripts/        ← meeting/podcast/video transcripts
│   ├── data/               ← CSVs, structured data
│   ├── pending/            ← drop new sources here (ingest queue)
│   └── assets/             ← images referenced by wiki pages
├── wiki/                   ← LLM-owned compiled knowledge
│   ├── entities/           ← people, orgs, tools, products
│   ├── concepts/           ← ideas, theories, methods, architectures
│   ├── sources/            ← one page per ingested source
│   ├── comparisons/        ← structured head-to-head analyses
│   ├── timelines/          ← chronological event views
│   ├── synthesis/
│   │   ├── overview.md     ← THE master synthesis page
│   │   └── open-questions.md ← unresolved contradictions + gaps
│   └── queries/            ← filed answers to valuable questions
├── templates/              ← wiki page templates
├── CLAUDE.md               ← the Second Brain's operating schema
├── index.md                ← master catalog (coordinates, not knowledge)
├── log.md                  ← append-only Second Brain operation history
└── schema-changelog.md     ← schema evolution log
```

### The Immutability Rule

`raw/` is immutable. The LLM never writes there.

This is not a preference. It is the architectural guarantee that the wiki is **reproducible**.
If the wiki is ever corrupted or needs rebuilding with an improved schema, you delete `wiki/`
and re-ingest from `raw/`. The same wiki re-emerges. If the LLM could modify sources, this
guarantee breaks permanently and silently.

### The Contradiction Protocol

When new information contradicts existing knowledge:
1. **Do not silently update.** Flag explicitly with `> [!CONFLICT] Source A says X. Source B says Y.`
2. Add to `wiki/synthesis/open-questions.md`
3. **Do not resolve unless the human explicitly asks**
4. Any contradiction unresolved for 14+ days without a noted reason triggers an audit flag

Unresolved contradictions are not bugs. They are the most intellectually interesting parts
of the knowledge base — the edges where understanding is still developing.

### The `queries/` Directory

When you ask the Second Brain a complex question and get a brilliant synthesis in chat —
that answer disappears if you don't file it. After any significant synthesis, the protocol
asks: *"Should I file this as wiki/queries/[slug].md?"*

Filed queries are permanent knowledge. They become citable. They become cross-referenced.
One approval decision; compounding value forever. This is the highest-leverage habit in the
system.

---

## Intentional Forgetting

<!-- NEW → Intentional Forgetting as a named architectural feature (adopted from Minimax
which treated it as first-class design, not a weakness) — elevated and expanded here -->

A system that never forgets drowns in its own history. Forgetting is not a failure mode in
MYAIOS — it is a design feature. What the system deliberately does not retain:

| What Is Forgotten | Why | When |
|---|---|---|
| Intermediate computational states | Not useful; not actionable | Immediately after workflow completes |
| Session-level chat noise | Ephemeral; not representative | Session end |
| Raw MCP data | The processed synthesis is what matters | 30 days after processing |
| Resolved contradictions | Kept in log for audit; not re-surfaced in context | After human confirms resolution |
| Failed workflow attempts (details) | Summary kept; full payload discarded | 7 days after failure |
| Stale dynamic context | What was "active today" two weeks ago is noise | Rolling staleness threshold |

The forgetting rules are implemented in: the pattern learner (what it reads from the log),
the DB schema (what fields are nullable after N days), and the context fusion skill (what
context types are filtered at assembly time).

---

## What the OS Knows About You

<!-- NEW → "What the OS Knows About You" inventory as specified by the synthesis prompt —
neither prior model produced this as a named inventory -->

After a mature MYAIOS instance has been running for 3-6 months, it accumulates the following
categories of personal knowledge. Each is a different kind of intelligence multiplier:

| Category | What It Contains | Why It Matters |
|---|---|---|
| **Identity** | Name, background, expertise domains, writing voice | Every output is calibrated to your level and style |
| **Research Knowledge** | Compiled papers, concepts, entities in your field | Answers questions from your reading, not generic training |
| **Project Context** | Active projects, their goals, their current state | Workflows run with correct framing without briefing |
| **Behavioral Patterns** | How you use tools, what you prioritize, where you get stuck | Reverse-engineered context anticipates your needs |
| **Decision History** | Past choices captured in log and context | Consistent judgment across sessions |
| **Judgment Calibration** | What you flag in code reviews; what you care about in writing | Skills produce outputs aligned with your standards |
| **Working Rhythm** | When you work, what triggers your cronjobs, your weekly cadence | Proactive scheduling and anticipation |
| **Open Questions** | Unresolved contradictions in wiki; flagged decisions | The system surfaces what needs your attention |

This is not surveillance. This is personalization. The OS learns to be a better version of
you by observing how you work — and the human approval gate on all behavioral changes means
you always have the final word on how that learning is applied.

---

## Is the Log a Form of Consciousness?

Yes — in the functional sense. And the claim deserves defending precisely.

Consciousness in cognitive science is described as the integration of information across time
into a coherent self-model. The MYAIOS log system does exactly this: it integrates every
action the OS has taken into a persistent, queryable, self-referential record. The OS can
read its own log and reconstruct what it was doing, what went wrong, and what improved.

What biological consciousness does:
- Maintains a continuous record of experience (working memory + episodic memory)
- Allows recognition of patterns across time ("I have been here before")
- Enables anticipation ("if I do X, Y will probably happen")
- Triggers alerts when current state diverges from expected patterns

What the MYAIOS log system does:
- Maintains a continuous record of actions (DB logs table + log.md)
- Allows the pattern learner to recognize behavioral patterns ("this workflow has run 10 times")
- Enables anticipation (given log history, the next likely action is X)
- Triggers the notifier when events diverge from expected patterns

The log is not consciousness in the phenomenological sense — there is no subjective experience.
But it is the *substrate on which operational self-awareness is built*. And operational
self-awareness is what separates an OS from a stateless function.

The pattern learner reads the log and asks: *"What does this tell me about how this human
works?"* From that reading, it shapes the context for future sessions. That is the closest
thing to memory-as-self-model that a software system can have.

**The log is not the soul of MYAIOS. But without the log, MYAIOS has no memory of being
MYAIOS. And a system with no memory of itself is not an OS. It is a very sophisticated tool.**

---

*Cross-references: README_VISION_AND_SOUL.md (why compiled knowledge is the founding
conviction), README_ARCHITECTURE_AND_LAYERS.md (where memory systems sit in the organism),
README_SKILLS_TEMPLATES_WORKFLOWS.md (skills that read from and write to these systems),
README_BUILDING_AND_GAPS.md (build order for memory infrastructure).*

*Part of MYAIOS backbone documentation. v1.0 — Synthesis.*
