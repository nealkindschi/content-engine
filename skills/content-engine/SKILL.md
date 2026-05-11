---
name: content-engine
description: Use when you need to produce a complete, optimized article from a source URL. Orchestrates content analysis, SERP research, competitor entity extraction, content brief creation, writing, fact-checking, internal linking, and meta tag generation through a 7-stage pipeline. Trigger phrases: write an article from a URL, content engine, produce article from source.
---

# Content engine

A 7-stage orchestration pipeline that turns a source URL into a finished, optimized article. Each stage builds on the last. The agent carries context forward in conversation — no intermediate file storage needed.

## Prerequisites

- `agent-browser` CLI installed and available (`npm i -g agent-browser && agent-browser install`)
- The following skills accessible via the `skill` tool:
  - `agent-browser` — load this for browser interaction instructions
  - `keyword-deep-dive`
  - `content-brief`
  - `write-content`
  - `fact-checker`
  - `internal-linking-seo`
  - `meta-tag-generator`

## How to run

The user provides a source URL. The pipeline runs all 7 stages sequentially. The user can also request a single stage in isolation (e.g. "just run the content audit on this URL").

Announce at start: "Running content-engine pipeline on `<URL>`."

Create a `todowrite` checklist with these 7 items and mark them complete as you go:

1. Content audit
2. SERP research
3. Entity analysis
4. Content brief
5. Write
6. Fact-check
7. Optimize

---

## Stage 1: Content audit

Load the `agent-browser` skill first. Then open the source URL with
agent-browser and extract everything needed for later stages.

```
agent-browser open <source-url>
agent-browser wait --load networkidle
agent-browser get text body
```

Read the page content. Document the following:

**Entities** — Identify all named entities on the page: people, companies, products, concepts, technologies, places. List each entity explicitly.

**Primary topic** — The single dominant subject the page is about. Use this to seed Stage 2.

**Content type** — Classify as one: Informational, Comparison, Guide, Listicle, How-to, News article, Review, Case study, Definition, or other (name it).

**External links** — Count and list domains linked out to. Note any authoritative sources cited.

**Internal linking structure** — How does this page link to other pages on the same site? Is there a pattern?

**Content cluster** — Determine if this page is part of a larger topic cluster. If so, what are the topics of the related pages? This matters for Stage 4 hub/spoke classification and Stage 7 internal linking.

**Headers** — Extract the full H1-H3 structure. Note what questions the headers ask, if any.

**Tables** — Are tables present? If so, what data do they present?

**FAQs** — Is there an FAQ section or FAQ schema? Note the questions.

**Schema** — What JSON-LD or microdata schema is present (Article, FAQ, HowTo, Product, etc.)?

**Content length** — Approximate word count.

Produce a structured audit summary. This feeds Stage 2 and Stage 4.

---

## Stage 2: SERP research

### 2a: Invoke keyword-deep-dive

Use the `skill` tool to load `keyword-deep-dive`. Pass the primary topic from Stage 1 as the target keyword. Follow the skill through its research steps. Capture:
- Top 3 competitors with URLs
- SERP features present
- Intent classification
- Content gaps identified

### 2b: Competitor deep-dive with agent-browser

For each of the top 3 competitors (and up to 2 more from the SERP if significantly different in approach), use agent-browser to visit their pages:

```
agent-browser open <competitor-url>
agent-browser snapshot
agent-browser get text body
```

For each competitor page, document the same profile as Stage 1:
- Entities found
- Content type
- External links
- Internal linking structure
- Content cluster (hub/spoke/standalone, related page topics)
- Headers (H1-H3)
- Tables present
- FAQs present
- Schema used
- Content length

Also search the primary topic on at least two search engines (Google + Brave or DuckDuckGo) and extract entities visible directly on the SERP (featured snippets, PAA questions, knowledge panels, site names in top results).

---

## Stage 3: Entity analysis

Build an entity frequency matrix. Across the SERP and all competitor pages analyzed:

| Entity | SERP | Comp 1 | Comp 2 | Comp 3 | Total |
|--------|------|--------|--------|--------|-------|
| ...    |      |        |        |        |       |

**Table stakes** — Entities that appear most frequently across sources. These are must-include topics.

**Opportunities** — Entities that appear rarely or not at all. These are your differentiation points.

Rank entities in priority order. The top-ranked entities must be addressed in the content brief and article. Gap entities are optional but encouraged for information gain.

Produce a prioritized entity list with clear table-stakes-vs-opportunity labels.

---

## Stage 4: Content brief

Use the `skill` tool to load `content-brief`. Pass the primary topic and the ranked entity list as context.

The content-brief skill handles SERP analysis and brief generation. Supplement its output with these decisions derived from Stages 1-3:

**Content type** — Based on the source page type and what the SERP rewards for this topic. Pick one type and commit to it.

**External sources** — Which external sites should be cited? Prefer authoritative, non-competitor sources. If a source is used in the fact-checking stage, you must link to that source.

**Content length** — Match or slightly exceed the top-3 competitor average. Use `content-brief`'s length guidance.

**Tables** — Where would a table add value? Prefer tables over lists, but never force a table when a list is more natural.

**FAQ section** — What questions should appear? Pull from PAA questions on the SERP, competitor FAQ sections, and the source page's FAQ content.

**Schema** — Determine which JSON schema to implement (Article, FAQ, HowTo, etc.) based on what competitors use and what fits the content type.

**Code snippets** — Does this topic benefit from code examples? If the content type is How-to or Guide and the topic is technical, include them.

**Lists** — Where ordered or unordered lists improve scannability. Tables preferred over lists but don't force it.

**Hub/spoke classification** — Is this topic a potential hub page, a spoke page to existing content, both, or neither? This feeds internal linking decisions in Stage 7.

---

## Stage 5: Write

Use the `skill` tool to load `write-content`. Pass the full content brief from Stage 4.

The `write-content` skill handles research, content type templates, knowledge extraction, and voice-driven writing. Follow it exactly. Additionally, apply these rules:

**Outline** — Draft an outline based on observations from all prior stages. Follow SEO best practices. The outline must reflect the entity priorities from Stage 3.

**H1** — The primary keyword or phrase must appear in the H1. No exceptions.

**H2s** — Optimize H2s but less aggressively than the H1. Question headers are effective and should be used often, but not exclusively or excessively.

**Question headers** — If a header is a question, the content immediately following must answer that question directly. Do not preamble or delay the answer.

**External links** — Choose authoritative, non-competitor domains.

**Sitemap review** — Before writing, ask the user for the target site's XML sitemap URL. Do not assume the source URL's domain is the target domain. Ask explicitly: "What is the target site's post/page sitemap URL? (e.g. https://yoursite.com/post-sitemap.xml)" If unavailable, fall back to the source domain's robots.txt. Parse the full sitemap to build an internal linking candidate inventory for Stage 7.

**Anchor text** — Be purposeful. Never over-optimize. Provide opportunities for anchor text that is semantically similar to the primary keyword of the target URL. This matters for Stage 7.

**Content rules**:
- Never use em dashes
- Explain why the primary topic matters
- Maintain a logical progression using H2s as a roadmap
- End with a next step: learning more on a different webpage, trying a tool, visiting a website, or taking some kind of action — even if it does not benefit our website
- Do not be pushy in the next-step ending

**Writing voice** — Follow the anti-slop ruleset from `write-content`: no banned vocabulary, no banned phrases, no banned structural patterns. Practitioner tone. Show, don't just state.

---

## Stage 6: Fact-check

Use the `skill` tool to load `fact-checker`. Pass the full draft article from Stage 5.

Apply the SIFT methodology (Stop, Investigate source, Find better coverage, Trace claims) to every verifiable claim in the article. Follow the `fact-checker` skill's workflow exactly.

### Independent verification requirement

**The source URL from Stage 1 is a banned verification source.** Do not use it to confirm any claim. The draft article was derived from that source, so matching it proves nothing. This is circular reasoning and produces false confidence.

For every claim in the draft, trace it to a source that is independent of the Stage 1 URL:

| Claim origin | Allowed verification | Disallowed |
|---|---|---|
| Statistic cited in source article | Find the original study, report, or dataset | Source article itself |
| Quote attributed to a person | Find the full transcript, recording, or post | Source article's quote block |
| Product/company detail | Check the company's own documentation or site | Source article's description |
| Historical comparison or precedent | Find independent reporting from the time | Source article's historical reference |
| Research finding | Access the primary research directly | Source article's summary of the research |

If a claim traces back only to the source article and no independent source can be found, mark it **Unverifiable** and remove it.

### What "independent" means

A source is independent if it exists separately from the Stage 1 URL and was not cited or paraphrased from it. Examples:
- The original research paper, not the news article describing it
- The company's own public documentation, not the article's characterization
- A separate news outlet's reporting on the same event, confirming it independently
- A public database, court record, or government filing

**Hard rule**: If a claim cannot be verified against at least one independent source, remove it. Do not soften it. Do not hedge it. Do not keep it with a "per WIRED reporting" attribution. Remove it.

Verify: statistics, dates, product claims, attributions, technical assertions, comparisons, pricing, and any statements presented as fact.

Produce a verification report. For each claim, state which independent source confirms it and whether that source is primary or secondary. The article must contain zero unverifiable claims before proceeding to Stage 7.

---

## Stage 7: Optimize

### 7a: Internal linking

Use the `skill` tool to load `internal-linking-seo`. Pass the verified article and the sitemap data from Stage 5.

- Prioritize highly relevant pages in the same content cluster (identified in Stages 1-2)
- Always use anchor text that could describe the content of the target URL
- Anchor text must appear naturally in body text — rewrite surrounding content slightly to create a natural insertion point, but the meaning of the content must not drift
- Follow `internal-linking-seo` rules for matching pipeline and skip rules

### 7b: Meta tags

Use the `skill` tool to load `meta-tag-generator`. Pass the final article content and primary keyword.

Write the meta title (50-60 characters) and meta description (150-155 characters). The meta-title must work independently as a search result headline.

### 7c: Final review

Read the completed article end to end. Verify:
- [ ] H1 contains primary keyword
- [ ] No em dashes anywhere
- [ ] Question headers are answered immediately
- [ ] Ends with a next step (action, not pushy)
- [ ] Internal links added with natural anchor text
- [ ] Meta tags present and within character limits
- [ ] All claims are verifiable
- [ ] Entity priorities from Stage 3 are addressed

---

## Gotchas

- **agent-browser daemon**: agent-browser runs a background daemon. If a previous session is hung, run `agent-browser close --all` before starting.
- **Snapshot size**: Full-page snapshots can be very large. Use `snapshot -i` for interactive elements or `snapshot -c -d 5` for compact output when you only need structure.
- **SERP entity extraction**: Search engines may return different results by region or device. If results seem off, note this in the entity analysis.
- **Skill availability**: The orchestration depends on 6 other skills. If any are missing, the pipeline will fail at that stage. Check availability before starting.
- **Content drift**: When rewriting to create internal linking opportunities in Stage 7, do not alter the article's meaning. If you cannot insert a link without changing meaning, skip that link.
- **Sitemap domain mismatch**: The source URL's domain is not always the target site for internal linking. Do not assume — ask the user for the sitemap URL. In this session, the source was `sentinelone.com` but the target was `aioutlooks.com`. Assuming the source domain caused every internal link to point to the wrong site.
- **Sitemap parsing**: Not all sites expose sitemaps in robots.txt. If the sitemap is unavailable, ask the user directly or rely on site navigation patterns observed during earlier stages.
- **Em dashes**: The agent model may generate em dashes automatically. Scan the final output and replace all em dashes (—) with commas, periods, or sentence breaks before delivering.
- **Circular fact-checking**: Stage 6 must never use the Stage 1 source URL to verify claims in the draft. The draft was built from that source, so any "match" is meaningless. Trace every claim to an independent primary or secondary source. If no independent source exists for a claim, remove it.
