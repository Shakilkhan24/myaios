---
title: Dashboard
type: meta
tags:
  - meta
  - dashboard
  - navigation
last_updated: 2026-05-08
cssclasses:
  - wide-page
---

# 🧠 Karpathy Second Brain — Dashboard

> [!QUOTE] _"The wiki is a persistent, compounding artifact. The LLM is the programmer. The human is the architect."_

---

## 🗺️ Quick Navigation

| Page | Purpose |
|------|---------|
| 📚 [[index]] | Master catalog of all wiki pages |
| 📋 [[log]] | Append-only operation history |
| 🧠 [[wiki/synthesis/overview\|Overview]] | Master synthesis of all knowledge |
| ❓ [[wiki/synthesis/open-questions\|Open Questions]] | Knowledge gaps & contradictions |
| 📋 [[wiki-ops-board]] | Kanban board for tracking operations |
| 🔄 [[schema-changelog]] | Schema evolution history |
| 📖 [[CLAUDE.md]] | LLM schema & instructions |

---

## 📊 Wiki Health

### Recently Updated Pages

```dataview
TABLE type AS "Type", confidence AS "Confidence", sources AS "Sources", last_updated AS "Updated"
FROM "wiki"
WHERE last_updated != null
SORT last_updated DESC
LIMIT 10
```

### Pages by Confidence

```dataview
TABLE length(rows) AS "Count"
FROM "wiki"
WHERE confidence != null
GROUP BY confidence
```

### Open Contradictions

```dataview
LIST
FROM "wiki"
WHERE contains(file.content, "[!CONFLICT]")
SORT file.mtime ASC
```

---

## 🔬 Operations

### Ingest a Source
```
"Read CLAUDE.md for context, then ingest raw/[type]/[filename].md"
```

### Run a Query
```
"Based on everything in the wiki, synthesize the current state of knowledge on [TOPIC]."
```

### Run a Lint
```
"Run a full lint on the wiki. File report to wiki/maintenance/lint-[date].md"
```

---

## 📈 Growth Tracker

| Metric | Count |
|--------|-------|
| Sources Processed | `$= dv.pages('"wiki/sources"').length` |
| Concept Pages | `$= dv.pages('"wiki/concepts"').length` |
| Entity Pages | `$= dv.pages('"wiki/entities"').length` |
| Filed Queries | `$= dv.pages('"wiki/queries"').length` |
| Comparisons | `$= dv.pages('"wiki/comparisons"').length` |
| Total Wiki Pages | `$= dv.pages('"wiki"').length` |

---

*Dashboard v1.0 — Updated with every session.*
