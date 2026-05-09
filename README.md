# Content Engine

A 7-stage orchestration skill (skill) that turns a source URL into a finished, optimized article by chaining multiple SEO content skills together with agent-browser research.

## Skills

| Skill | Purpose |
|-------|---------|
| `content-engine` | Orchestration pipeline: audit → research → entity analysis → brief → write → fact-check → optimize |
| `agent-browser` | Browser automation CLI — open URLs, snapshot pages, extract text, do SERP research |

## Prerequisites

- [agent-browser](https://github.com/nealkindschi/agent-browser) CLI (`npm i -g agent-browser && agent-browser install`)
- Skills: `agent-browser`, `keyword-deep-dive`, `content-brief`, `write-content`, `fact-checker`, `internal-linking-seo`, `meta-tag-generator`

## Pipeline

```
URL → Content Audit → SERP Research → Entity Analysis → Content Brief → Write → Fact-Check → Optimize → Article
```
