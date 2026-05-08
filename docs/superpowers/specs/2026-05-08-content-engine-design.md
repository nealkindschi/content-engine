# Content Engine Design

**Date**: 2026-05-08
**Status**: Final

## Goal

A single orchestration skill that takes a source URL as input and produces a finished, optimized article as output through a 7-stage pipeline.

## Architecture

The agent is the context holder. Each stage invokes sub-skills via the `skill` tool and passes findings forward in conversation. No intermediate file storage is needed.

```
SOURCE URL → Stage 1 (Content Audit) → Stage 2 (SERP Research)
           → Stage 3 (Entity Analysis) → Stage 4 (Content Brief)
           → Stage 5 (Write) → Stage 6 (Fact-Check) → Stage 7 (Optimize)
           → FINISHED ARTICLE
```

## Stages

### Stage 1: Content Audit
agent-browser opens the source URL. Agent extracts entities, identifies primary topic, classifies content type, notes links/cluster/headers/tables/FAQs/schema/length.

### Stage 2: SERP Research
Invokes `keyword-deep-dive` for SERP analysis. Uses agent-browser to deep-read top 3+ competitor pages, extracting the same profile as Stage 1 plus SERP-level entities from search results.

### Stage 3: Entity Analysis
Builds an entity frequency matrix across SERP and all competitor pages. Ranks entities: table stakes (must-include) vs opportunities (differentiation).

### Stage 4: Content Brief
Invokes `content-brief`. Supplements with decisions on content type, external sources, length, tables, FAQ, schema, code snippets, lists, and hub/spoke classification — all derived from prior stage findings.

### Stage 5: Write
Invokes `write-content`. Additional rules: H1 contains primary keyword, question H2s answer immediately, max 3 external links, no em dashes, next-step ending. Reviews sitemap via robots.txt before writing.

### Stage 6: Fact-Check
Invokes `fact-checker`. Hard rule: unverifiable claims are removed, not softened.

### Stage 7: Optimize
Invokes `internal-linking-seo` for internal links (cluster-prioritized, natural anchor text, no content drift). Invokes `meta-tag-generator` for title and description. Final review checklist.

## Dependencies

- **agent-browser** CLI — for opening pages, taking snapshots, browsing SERPs
- **keyword-deep-dive** skill — SERP analysis and competitive read
- **content-brief** skill — brief generation with hub/spoke classification
- **write-content** skill — voice-driven content writing with anti-slop rules
- **fact-checker** skill — SIFT-based claim verification
- **internal-linking-seo** skill — internal link insertion with natural anchor text
- **meta-tag-generator** skill — title and meta description generation

## Constraints

- Linear pipeline: stages run in order, each completes before the next
- Stages can be run independently if the user requests
- All entity extraction is manual (agent analyzes content, no tool dependency)
- No em dashes in final article (scan and replace)
- Maximum 3 external links per article
- All claims must be verifiable (fact-check gate)

## Scope

Single orchestration skill. Does not re-implement any sub-skill functionality. Focuses on sequencing, context passing, and quality gates between stages.
