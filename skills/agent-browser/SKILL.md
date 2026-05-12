---
name: agent-browser
description: Browser automation CLI for AI agents. Load this skill whenever the content-engine pipeline needs to open a URL, read page content, extract text, take snapshots, or do SERP research. Also use when the user asks to visit a website, inspect a page, or gather data from the web. Install once: `npm i -g agent-browser && agent-browser install`.
allowed-tools: Bash(agent-browser:*), Bash(npx agent-browser:*)
---

# agent-browser

Fast browser automation CLI for AI agents. Chrome/Chromium via CDP, no
Playwright or Puppeteer dependency. Accessibility-tree snapshots with
compact `@eN` refs let agents read and interact with pages in ~200-400
tokens instead of parsing raw HTML.

## Prerequisite for content-engine

The content-engine pipeline depends on agent-browser for every stage
that involves reading a live URL: content audit, SERP research, competitor
deep-dives, and fact-checking. Install it before running the pipeline:

```bash
npm i -g agent-browser
agent-browser install         # downloads Chrome for Testing (one-time)
agent-browser doctor          # verify everything works
```

## The core loop

Every content-engine interaction follows this pattern:

```bash
agent-browser open <url>        # 1. Open a page
agent-browser snapshot -i       # 2. See what's on it (interactive elements only)
agent-browser get text @e1      # 3. Extract content from refs
agent-browser close             # 4. Clean up
```

Refs (`@e1`, `@e2`, ...) are assigned fresh on every snapshot. They become
**stale the moment the page changes** — after navigation, form submits,
dynamic re-renders. Always re-snapshot before your next ref interaction.

## Reading a page (for content analysis)

These are the commands you'll use most in the content-engine pipeline:

```bash
# Snapshots: get a structured view of the page
agent-browser snapshot                    # full tree (verbose)
agent-browser snapshot -i                 # interactive elements only (preferred)
agent-browser snapshot -i -c              # compact (no empty structural nodes)
agent-browser snapshot -i -d 4            # cap depth at 4 levels
agent-browser snapshot -s "#main"         # scope to a CSS selector
agent-browser snapshot -i --json          # machine-readable

# Get page metadata
agent-browser get title                   # page title
agent-browser get url                     # current URL

# Extract text content
agent-browser get text body               # all visible text on the page
agent-browser get text @e1                # text of a specific element
agent-browser get html @e1                # innerHTML of a specific element
```

Snapshot output looks like:

```
Page: Example Domain
URL: https://example.com

@e1 [heading] "Example Domain"
@e2 [paragraph] "This domain is for use..."
@e3 [link] "More information"
```

### Scrolling through long content

For long-form articles, scroll and re-snapshot to see lower sections:

```bash
agent-browser scroll down 800
agent-browser snapshot -i
agent-browser get text body               # full text after scrolling
```

### Search engine research

For SERP research (used in content-engine Stage 2), search and read results:

```bash
# Brave Search with headed mode (most reliable — avoids bot blocks):
agent-browser open "https://search.brave.com/search?q=your+keyword" --headed
agent-browser wait --load networkidle
agent-browser snapshot -i -c -d 5
agent-browser get text body               # full SERP text with AI overview, results, and related queries

# Google (may block headless browsers — fall back to Brave if blocked):
agent-browser open "https://www.google.com/search?q=your+keyword"
agent-browser snapshot -i
agent-browser wait --load networkidle
agent-browser get text body               # full SERP text

# DuckDuckGo HTML mode (lightweight, no JS — may also block bots):
agent-browser open "https://html.duckduckgo.com/html/?q=your+keyword"
agent-browser get text body               # clean HTML results
```

## Navigating and interacting

When you need to interact with a page (click links, fill forms):

```bash
agent-browser click @e1                   # click by ref
agent-browser click @e1 --new-tab         # open in new tab
agent-browser fill @e2 "hello"            # clear then type
agent-browser type @e2 " world"           # type without clearing
agent-browser press Enter                 # press a key at current focus
agent-browser hover @e1                   # hover
agent-browser scrollintoview @e1          # scroll element into view
```

Semantic locators (alternative when refs don't work):

```bash
agent-browser find role button click --name "Submit"
agent-browser find text "Sign In" click
agent-browser find label "Email" fill "user@test.com"
```

## Tab management (multi-page research)

When analyzing multiple competitor pages:

```bash
agent-browser tab new https://competitor1.com   # open in new tab
agent-browser snapshot -i                        # read it
agent-browser tab new https://competitor2.com    # open another
agent-browser snapshot -i
agent-browser tab 1                              # switch back to first tab
agent-browser tab close [id]                     # close a tab
```

## Waiting

Agents fail more often from bad waits than from bad selectors:

```bash
agent-browser wait @e1                     # until element appears
agent-browser wait --load networkidle      # until network idle (post-navigation)
agent-browser wait --text "Welcome"        # until text appears
agent-browser wait --url "**/dashboard"    # until URL matches (glob)
agent-browser wait 2000                    # dumb wait in ms (last resort)
```

After any page-changing action, wait for something specific. Avoid bare
`wait <ms>` except as a last resort.

## Taking screenshots

Less common in the pipeline but useful for visual verification:

```bash
agent-browser screenshot page.png          # viewport screenshot
agent-browser screenshot --full            # full page
agent-browser screenshot --annotate        # numbered labels + refs legend
```

## Common content-engine workflows

### Content audit (Stage 1)

```bash
agent-browser open <source-url>
agent-browser wait --load networkidle
agent-browser snapshot -i -c -d 5
agent-browser get text body
```

### Competitor analysis (Stage 2)

```bash
agent-browser open <competitor-url>
agent-browser wait --load networkidle
agent-browser get title
agent-browser snapshot -i -c -d 4
agent-browser get text body
agent-browser scroll down 1000
agent-browser get text body                # get lower content
```

### SERP entity extraction (Stage 2)

```bash
agent-browser open "https://www.google.com/search?q=<primary-topic>"
agent-browser wait --load networkidle
agent-browser snapshot -i -c
```

### Fact-check verification (Stage 6)

```bash
agent-browser open <independent-source-url>
agent-browser wait --load networkidle
agent-browser get text body
```

## Cleaning up

Always close the browser when done with a pipeline stage:

```bash
agent-browser close                        # close current session
agent-browser close --all                  # close all sessions (if daemon is stuck)
```

## Troubleshooting

**"Ref not found" / "Element not found: @eN"**
Page changed since the snapshot. Re-snapshot first.

**Snapshot is empty or missing content**
The page may not have finished loading. Run `wait --load networkidle` and
retry. Some SPAs need a few seconds for dynamic content.

**"Failed to connect" / daemon issues**
```bash
agent-browser doctor                       # diagnose
agent-browser doctor --fix                 # repair
agent-browser close --all                  # clear hung sessions
```

**Page content is truncated**
Use `-d 4` or `-d 5` to limit snapshot depth for very large pages. For
full text extraction, use `get text body` instead of snapshot.

**Daemon from previous session is still running**
```bash
agent-browser close --all
```

## Full command reference

See `references/commands.md` for the complete command listing.

## Gotchas

- **Refs are ephemeral.** After any navigation or DOM change, previous
  refs are invalid. Always snapshot before interacting.
- **Daemon persistence.** The browser daemon stays running between
  commands. If a prior session crashed, run `close --all` before starting.
- **Full snapshots are large.** Always use `-i` (interactive only) or
  `-c -d 4` for content-engine work. Full tree snapshots of complex
  pages can blow your context.
- **Google SERP blocks.** Google may serve different content to headless
  Chrome. DuckDuckGo HTML mode and Startpage may also block headless
  browsers. **Brave Search with `--headed` is the most reliable option**
  for SERP research — it consistently returns AI overviews, organic
  results, related queries, and video carousels without blocking.
  Use `--headed` liberally for search engine access.
- **SPA navigation.** Single-page apps may not trigger `networkidle`
  reliably. After navigation, wait for a specific element or text
  instead of relying on load events.
