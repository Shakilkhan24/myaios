---
type: source
title: <% tp.file.cursor(1) %>
date: <% tp.date.now("YYYY-MM-DD") %>
source_type: <% tp.system.suggester(["paper", "article", "book", "transcript", "note"], ["paper", "article", "book", "transcript", "note"]) %>
source_file: "[[raw/<% tp.file.cursor(2) %>]]"
authors: <% tp.file.cursor(3) %>
key_concepts: <% tp.file.cursor(4) %>
key_entities: <% tp.file.cursor(5) %>
tags:
  - <% tp.file.cursor(6) %>
sources: 1
last_updated: <% tp.date.now("YYYY-MM-DD") %>
confidence: <% tp.system.suggester(["high", "medium", "low"], ["high", "medium", "low"]) %>
---

## Summary

<% tp.file.cursor(7) %>

## Key Claims

- [Claim] (evidence: strong/moderate/weak)

## Data & Evidence

[Specific numbers, quotes, statistics, experimental results]

## Limitations & Caveats

[What this source misses, potential biases, scope limitations]

## Cross-references

Supports: 
Contradicts: 
Related: 
