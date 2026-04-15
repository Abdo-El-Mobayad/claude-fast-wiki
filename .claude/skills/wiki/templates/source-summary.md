# Source Summary Template

Use this template when creating summaries during ingest.

---

## Frontmatter Schema

```yaml
---
title: "Source Title As Published"
source_url: "https://original-source-url.com" # Original URL for provenance (raw file is deleted after ingest)
author: "Author Name" # Extracted from byline or metadata
date_published: 2026-04-02 # When the source was originally published
date_ingested: 2026-04-05 # When we processed it (today)
date_updated: 2026-04-05 # When this summary was last modified
topics: [primary-topic, secondary-topic] # Topic slugs
status: processed # unprocessed | processed | stale
key_concepts: [concept-1, concept-2, concept-3] # Slugified concept names for compile
---
```

### Field Rules

- `title`: Use the original source title as published. Don't rename.
- `source_url`: Original URL where the source was published or clipped from. This is the provenance chain. Raw files are deleted after ingest, so the URL is the permanent reference.
- `author`: Extract from byline, meta tags, or "Author" field. "Unknown" if not found.
- `date_published`: ISO date. Extract from the source's byline, publication date, or metadata. Set to `"unknown"` if not determinable, and flag in the ingest report.
- `date_ingested`: Today's date. Set once during ingest.
- `date_updated`: Update if the summary is ever revised (e.g., after re-reading the source).
- `topics`: Array of topic slugs. Primary topic first.
- `status`: `processed` = summary is complete. `stale` = source may have newer information available. `unprocessed` = placeholder only.
- `key_concepts`: Array of concept slugs that this source discusses. These feed the compile step's concept clustering. Use kebab-case slugs that could become article filenames.

### For HTML Companion Summaries

Add these extra fields:

```yaml
type: html-document
html_path: "filename.html" # Relative path to the HTML file in same folder
```

---

## Body Structure

```markdown
# {Source Title}

> {One-sentence TL;DR of the source's main contribution}

## Key Claims

{The most important assertions, findings, or arguments from the source. Be specific.}

- {Claim 1 with supporting data if available}
- {Claim 2 with context}
- {Claim 3}

## Important Data

{Specific numbers, metrics, benchmarks, statistics from the source. Preserve exact figures.}

- {Data point 1}
- {Data point 2}

## Methodology / Approach

{How did the author reach their conclusions? What framework, research method, or reasoning did they use? Skip this section if the source is an opinion piece or announcement.}

## Notable Quotes

{2-3 direct quotes that capture the source's most important or distinctive points. Use blockquotes.}

> "Exact quote from the source" - Author

## Connections

{How does this source relate to other topics in the KB? What existing concepts does it support, contradict, or extend?}

- Relates to [[concept-name]]: {how}
- Contradicts/supports [[other-concept]]: {how}

## Gaps

{What does this source NOT cover that would be useful to know? What questions does it raise?}
```

### Writing Guidelines

- Summaries should be comprehensive enough that someone reading ONLY the summary understands the source's full contribution. The raw file is deleted after ingest, so the summary is the permanent record. Make it count.
- Preserve specific data: numbers, dates, metrics, percentages. These are what make the KB valuable.
- Don't editorialize. Report what the source says. Save synthesis for concept articles.
- Key concepts list should use consistent slugs across summaries. Check existing summaries for established slug conventions before creating new ones.
- For very long sources (5000+ words), focus on the most novel and important sections. Note what was omitted at the end.
