---
name: internal-linking-seo
description: Adds contextual internal links to website content using Search Analytics for Sheets CSV data (Query + Landing page) to determine which terms each page should rank for. Use when optimizing content for internal linking, adding links to blog posts/articles, building topical authority through strategic interlinking, or processing a GSC export for link opportunity analysis.
---

# Internal linking SEO

## Workflow

Progress:
- [ ] Step 1: Discover all site pages via sitemap + load GSC CSV data
- [ ] Step 2: Read the target content and extract its body text themes
- [ ] Step 3: Fetch relevant candidate pages and match against body text in parallel
- [ ] Step 4: Insert links with optimized anchor text
- [ ] Step 5: Verify rules compliance

## Step 1: Discover site pages + load GSC data

### 1a. Fetch sitemap(s)

**Always fetch the sitemap before matching.** Many site pages are absent from GSC data (new pages, low-traffic pages) but are strong semantic matches.

1. Start with the sitemap index: `https://yoursite.com/sitemap_index.xml`
2. Parse it for child sitemap URLs (e.g. `post-sitemap.xml`, `category-sitemap.xml`)
3. Fetch all child sitemaps to build a complete URL inventory
4. Extract every `<loc>` URL. Skip category/tag/archive pages — only keep post/pillar pages.

### 1b. Load GSC CSV

The CSV from Search Analytics for Sheets has these columns:
`Query, Page, Clicks, Impressions, CTR, Position`

Build a mapping:
- Group queries by `Page` (URL)
- For each page, the queries associated with it are the terms that page should rank for. These are anchor text candidates to use when linking TO that page from other content.
- Prioritize queries with higher impressions or lower (better) position as stronger signals.

### 1c. Merge into a combined page inventory

Combine the sitemap URL list with the GSC query mapping:

- **GSC pages** get priority from total impressions (sum all queries per page, sort descending)
- **Sitemap-only pages** (not in GSC) go at the bottom of the priority list but are NOT skipped — they still have link potential
- For sitemap-only pages, derive anchor text candidates from the URL slug and page `<title>` / `<h1>` (fetched in Step 3)

### 1d. Page prioritization

Rank all pages by total GSC impressions (descending). Sitemap-only pages with no GSC data share the lowest tier. In Step 3, iterate pages in this order — high-traffic pages get first crack at the limited link slots before lower-value pages consume them.

## Step 2: Read target content

Read the full content of the page being optimized. Identify:
- The subject and entities covered (to determine relevance)
- Body paragraphs where natural link placement is possible
- Sections to avoid for linking (see Gotchas)

Extract the body text for matching. Strip navigation, headers, footers, sidebars, and template content — keep only the article body.

## Step 3: Find link opportunities (parallel matching)

### 3a. Pre-scan: skip already-linked pages

Before checking any candidates, scan the target content for all existing links that point to the same domain. Check both formats:
- HTML anchors: `<a href="https://yoursite.com/...">`
- Markdown links: `[text](https://yoursite.com/...)`

Build a skip-set of page paths that are already linked. Pages in the skip-set get excluded from matching entirely. Only count links in body paragraph text — ignore template-generated links (footer, sidebar, related posts widgets).

EXCEPTION: If an already-linked page shares a query string with an unlinked page, check that query for the unlinked page only. The already-linked page does not get a second link.

### 3b. Filter candidates by semantic relevance

Before fetching and matching, filter the combined page inventory by relevance to the target content. Skip pages with zero topical overlap (e.g. don't check a "bird identification" page against an article about "Claude Code tokens"). This avoids wasting fetches on irrelevant pages.

Use these signals for relevance:
- URL slug keywords that overlap with target body themes
- Category/tag alignment
- GSC queries that share keywords with the target body

### 3c. Fetch candidate pages in parallel

For pages that pass the relevance filter:

1. **GSC pages:** Already have queries — skip fetch unless the query match failed and slug-derived anchors are needed as fallback
2. **Sitemap-only pages:** Fetch the page (as markdown) to extract its `<h1>` title and confirm the topic. Derive anchor text candidates from the slug and title

Fetch all candidate pages in parallel to minimize round-trips. Use a multi-agent dispatch pattern if the candidate list exceeds ~6 pages — assign subsets to parallel agents, each agent returning matched opportunities.

### 3d. Matching, per page

For each candidate page not in the skip-set:

**Anchor text sources (in priority order):**
1. GSC queries (if available)
2. URL slug, hyphenated → space-separated → as potential anchor
3. Page `<h1>` / `<title>` key phrase

**Matching process:**
1. Normalize all anchor candidates: lowercase, expand common abbreviations (e.g. "ML" → "machine learning", "NLP" → "natural language processing")
2. Check each candidate against the body text using case-insensitive substring matching. The phrase must appear as a contiguous phrase in a body paragraph — not in headers, code blocks, or tables
3. Stop scanning that page's remaining candidates as soon as you find one match. A page can only be linked once
4. If zero anchor candidates match, skip the page

### 3e. Candidate-skipping rules

Skip a candidate immediately (don't scan the body) if:
- It's >5 words — exceeds max anchor length
- It's already been matched and linked against a different page in this session

### 3f. Remaining rules

For each valid match:
1. Confirm semantic relevance between the source content topic and the target page's topic
2. Limit to 2-5 contextual links per 1,000 words of content
3. Ensure each target page appears only once in the entire content

## Step 4: Insert links

For each valid opportunity:
- Anchor text: Use the query phrase exactly or a close natural variant. Max 5 words per anchor.
- Link format: `<a href="https://yoursite.com/target-page">anchor text</a>`
- Placement: Insert into inline body text at the first natural occurrence of the phrase. Do not disrupt sentence flow.
- Mix anchor text types: exact-match, partial-match, and branded variations across links in the same piece of content. Avoid repeating the same anchor text for different targets.
- If the target page is a "Pillar" (comprehensive guide / high-value page), nodes should link TO pillars. Ensure topical equity flows toward authority pages.

## Step 5: Verify rules

After inserting all links, verify EVERY link against the rules below.

## Output format

When presenting results, only show what was linked and close calls worth discussing. Do NOT produce a catalog of every skipped page with reasons. The user doesn't need an audit trail of misses.

**Always include:**
- The link(s) inserted, with anchor text and placement location
- Existing link inventory (what was already on the page)
- Total link count vs. the 2-5 per 1K word target

**Only include when interesting:**
- Near-misses — a page was highly relevant to the topic but the exact phrase didn't appear in body text. Flag these briefly: "Close match: `/example-page/` — 'example phrase' appears as 'partial phrase' but not contiguously. Worth a manual review."
- Pages that were already linked (the skip-set), so the user knows they're covered

**Never include:**
- Lists of irrelevant pages with reasons they were skipped
- Pages with zero topical relevance to the target content
- Play-by-play of the matching algorithm for every page scanned

## Gotchas

- **Never link in headers** (`<h1>`-`<h6>`), page titles, image captions, code blocks, or pull quotes. Only link in body paragraph text.
- **A page URL can only be linked once per page.** If page X is already linked somewhere in the content, do not link to it again.
- **Standard `<a href="">` elements only.** No `onclick`, `<span>`, or JavaScript-based links. Google can only reliably crawl `<a>` tags with `href` attributes.
- **All internal links must be "dofollow"** (default). Never add `rel="nofollow"` to internal links.
- **Max 5 words per anchor text.** Shorter anchors (2-3 words) perform better.
- **Avoid generic anchors** like "click here", "read more", "learn more". The anchor text must describe the destination page content.
- **Don't force unnatural links.** If no good match exists in body text for a target page, skip it. Forced links harm user experience and may trigger spam classifiers.
- **Keep total links under 150 per page** to avoid equity dilution.
- **Redirects are not allowed.** Every linked URL must resolve to a 200 status. Update any old URLs before linking.
- **Human review required for high-traffic pages.** Flag pages with significant organic traffic for manual review before inserting AI-generated links.
- **Sitemap is mandatory.** Always fetch and parse the full sitemap before matching. GSC data alone is incomplete — many high-relevance pages have zero impressions and won't appear in the CSV.
- **Use URL slugs as anchor clues.** For sitemap-only pages, derive anchor text from the slug (`/llm-context-windows-explained/` → "context window", `/best-ai-coding-tools/` → "AI coding tools" or "coding tools") and validate against body text.
- **Fetch candidate pages in parallel.** Never fetch pages one-at-a-time. Parallel fetches + multi-agent dispatch dramatically reduces wall-clock time.

## CSV format example

```
Query,Page,Clicks,Impressions,CTR,Position
automl,https://site.com/what-is-automl/,0,268,0.0%,66.5
```

This means the page `https://site.com/what-is-automl/` should rank for "automl" — so when other pages use the word "automl" in body text, that phrase is a candidate to link to that page.
