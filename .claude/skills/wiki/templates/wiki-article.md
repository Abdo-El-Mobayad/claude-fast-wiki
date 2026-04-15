# Wiki Article Template

Use this template when creating concept articles during compile or file-back.

---

## Frontmatter Schema

```yaml
---
title: "Human-Readable Concept Name"
topics: [primary-topic, secondary-topic] # Topic slugs this article belongs to
sources: ["[[source1-slug]]", "[[source2-slug]]"] # Wikilinks to source summaries (flat in wiki/)
related: ["[[other-concept]]", "[[another]]"] # Wikilinks to related concept articles
date_created: 2026-04-05 # Set once on creation, never change
date_updated: 2026-04-05 # Update on every edit
newest_source: 2026-04-02 # date_published of the most recent source
status: active # draft | active | stale | archived
word_count: 1247 # Approximate, update on edit
---
```

### Field Rules

- `title`: Human-readable, title case. This appears in INDEX.json.
- `topics`: Array of topic slugs matching topic keys in INDEX.json.
- `sources`: Array of [[wikilinks]] to the summaries this article synthesizes. Always wikilinks, never plain text.
- `related`: Array of [[wikilinks]] to other concept articles. Build cross-links aggressively.
- `date_created`: ISO date string. Set once. Never overwrite.
- `date_updated`: ISO date string. Update every time the article body changes.
- `newest_source`: The most recent `date_published` among linked sources. Used for relevance decay.
- `status`: `draft` = stub or incomplete. `active` = complete and current. `stale` = needs refresh. `archived` = no longer relevant.
- `word_count`: Approximate body word count. Helps lint detect thin articles.

### For HTML Companion Articles

Add these extra fields:

```yaml
type: html-document
html_path: "filename.html" # Relative path to the HTML file
```

### For Connection Articles

Replace `topics` with `bridges`:

```yaml
bridges: [topic-slug-1, topic-slug-2] # Topics this article connects
```

Place connection articles flat in `wiki/` with `conn-` filename prefix.

---

## Body Structure

```markdown
# {Title}

{Opening paragraph: 1-3 sentences defining the concept and why it matters.}

## Core Idea

{The central thesis or mechanism. What is this concept at its essence?}

## Key Points

{Structured breakdown of the concept. Use subsections if needed.}

### {Sub-point 1}

{Detail with citations: "According to [[source-name-slug]], ..."}

### {Sub-point 2}

{More detail. Link to related concepts: "This connects to [[other-concept]] because..."}

## Practical Implications

{What does this mean in practice? How is it applied?}

## Sources

- [[source1-slug]] - {one-line note on what this source contributed}
- [[source2-slug]] - {one-line note}

## Related

- [[related-concept-1]] - {why it's related}
- [[related-concept-2]] - {why it's related}
```

### Writing Guidelines

- Lead with the concept, not the sources. This is a wiki article, not a literature review.
- Synthesize across sources. Don't just list what each source says.
- Use [[wikilinks]] liberally for any concept that has or should have its own article.
- Include specific data points, metrics, and quotes from sources.
- Keep articles focused. One concept per article. If it's growing past 2000 words, consider splitting.
- Never copy-paste from sources verbatim without attribution. Synthesize and cite.
