# CLAUDE.md — Wiki Knowledge Base Schema

## Overview
This is a personal knowledge base Wiki maintained by an LLM. The Wiki accumulates knowledge with each ingest and query, growing richer over time.

**Core principle**: Humans curate sources and ask questions; the LLM handles all bookkeeping — summarizing, cross-referencing, archiving, and maintaining consistency.

## Directory Structure

```
.
├── VAULT-INDEX.md       # Live dashboard (auto-maintained)
├── CLAUDE.md            # This file
├── raw/                    # Layer 1: Raw data (human-written, LLM read-only)
│   ├── articles/          # Web clippings and articles
│   ├── papers/            # Academic papers, PDFs
│   ├── repos/            # Repository READMEs
│   ├── transcripts/       # Meeting/podcast transcripts
│   ├── data/             # CSV, JSON data files
│   └── assets/           # Images, attachments
│
└── wiki/                  # Layer 2: LLM-maintained knowledge base (LLM writes and maintains)
    ├── index.md         # Master index (start queries here)
    ├── log.md           # Activity timeline (append-only)
    ├── hot.md           # Hot cache (session context)
    ├── sources/         # Source summary pages
    ├── entities/        # Person/company/product pages
    ├── concepts/        # Concept explanation pages
    ├── comparisons/      # Comparative analysis pages
    └── syntheses/       # Synthesis essays/reports
```

## Wiki Page Format

### Required Frontmatter
```yaml
---
title: "Page Title"
created: 2026-04-14
modified: 2026-04-14
tags: [#concept, #ai, #llm]
related: [[wiki/concepts/other-concept]], [[wiki/entities/person-x]]
summary: "One-sentence summary"
---
```

## Four Core Operations

> **Mandatory rule: Upon receiving an operation command, you MUST first read the full content of the corresponding skill file before executing. Do not skip this step.**

### Ingest
**Trigger phrases**: "Ingest raw/...", "add to the wiki", "save to the knowledge base", "store in the wiki", "put this in the wiki", "process this article", or when the user sends file/text content and mentions the wiki
**Skill file**: `.claude/skills/ingest-wiki.md`
**Description**: Supports two trigger modes: user specifies a raw/ path, or sends content directly for the LLM to auto-archive into raw/. Extracts key points, integrates into Wiki, updates index / concepts / entities / comparisons / log / VAULT-INDEX, Git commit.

### Query
**Trigger phrases**: "according to the wiki", "look up X in the wiki", "what does the wiki say about", "is X in the wiki", as well as substantive questions about topics already in the wiki
**Skill file**: `.claude/skills/query-wiki.md`
**Description**: Locates relevant pages from index.md, synthesizes multi-page content into a sourced answer, asks whether to archive when the answer meets archival criteria.

### Synthesize
**Trigger phrases**: "synthesize", "write a report on X", "synthesis", "pull together everything in the wiki about X", "create a synthesis report on X"
**Skill file**: `.claude/skills/synthesize-wiki.md`
**Description**: Proactively synthesizes a specified topic across multiple wiki pages, generates a structured report written directly to wiki/syntheses/, updates index / log / VAULT-INDEX, Git commit.

### Lint
**Trigger phrases**: "Lint the wiki", "lint wiki", "health check", "check the wiki for issues", "are there issues with the wiki"
**Skill file**: `.claude/skills/lint-wiki.md`
**Description**: Scans for contradictions, orphan pages, missing concepts, outdated statements, structural issues, generates a diagnostic report, appends to log, Git commit.

## Conventions

1. **Strict layer boundaries**
   - `raw/` stores original sources: written by humans, or archived by LLM during Ingest on the human's behalf; LLM does not actively modify existing content
   - `wiki/` is written and maintained exclusively by the LLM
2. **Every operation appends to wiki/log.md**
3. **After creating a new page, wiki/index.md must be updated**
4. **Use double-bracket syntax [[page]] for internal links**
5. **VAULT-INDEX.md is auto-maintained by the LLM** — update stats and activity after every ingest/lint/synthesize

## Page Type Creation Rules

| Page Type | Location | When to Create |
|-----------|----------|----------------|
| sources/ | `wiki/sources/` | Required on every ingest — one summary page per source |
| concepts/ | `wiki/concepts/` | When a new concept is identified during ingest; or when a concept-level conclusion is produced during synthesize |
| entities/ | `wiki/entities/` | When a new person / company / product / organization is identified during ingest |
| comparisons/ | `wiki/comparisons/` | When a new source has **≥3 comparable dimensions** of competition/substitution with existing content (e.g. two tools, two methodologies) |
| syntheses/ | `wiki/syntheses/` | When the user explicitly triggers synthesize; or when a query answer meets archival criteria (synthesizes >2 sources, >200 words) and the user confirms archival |

## Git Workflow

After each Ingest / Lint operation:

1. `git add .`
2. `git commit -m "{operation type}: {short description}"`
3. `git push`
