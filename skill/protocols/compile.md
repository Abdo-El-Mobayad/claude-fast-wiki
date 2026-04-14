# Compile Protocol

Build and maintain the wiki's concept articles, cross-links, and topic taxonomy. This is where raw summaries become structured knowledge.

---

## Trigger

User asks to organize, rebuild, or restructure the wiki. Also appropriate after a large ingest session when new cross-source concepts have emerged.

## Pre-Load

Read `templates/wiki-article.md` for article format and `templates/index-format.md` for INDEX.json schema before executing.

---

## Steps

### 1. Read Current State

Read `Vault/INDEX.json` in full. Understand:

- What topics exist and how many articles each has
- What concept articles already exist (avoid duplicating)
- What gaps and needs_attention items are flagged
- What summaries exist and their key_concepts lists

### 2. Identify Concept Clusters

Scan all summaries' `key_concepts` fields across all topics. A concept becomes an article candidate when it:

- Appears in 2+ source summaries
- Represents a distinct, nameable idea (not a generic term)
- Is not already covered by an existing concept article

Also check `needs_attention` for previously flagged compile candidates.

### 3. Create or Update Concept Articles

For each concept cluster:

**If new concept:**

- Create `Vault/wiki/[concept-slug].md` using the wiki-article template (flat in wiki/, no subfolders)
- Synthesize information from all relevant summaries
- Cross-link to source summaries with [[wikilinks]]
- Cross-link to related concept articles in the same and other topics
- Set `date_created` and `date_updated` to today
- Compute `newest_source` from the most recent source's `date_published`

**If existing concept needs update (new sources reference it):**

- Read the existing article
- Integrate new information from new summaries
- Add new source [[wikilinks]]
- Update `date_updated` to today
- Update `newest_source` if a newer source was added

### 4. Build Connections

Identify cross-topic themes: concepts that appear in articles across 2+ topics.

For significant cross-topic themes, create connection articles flat in `Vault/wiki/`:

- Filename prefixed with `conn-` (e.g., `conn-ai-tools-for-knowledge-management.md`)
- `bridges` frontmatter field lists the connected topics
- Body synthesizes the cross-topic relationship
- Links to relevant articles in both topics

### 5. Review Topic Health

For each topic:

- Are there orphan summaries with no concept articles referencing them?
- Are there thin topics (1-2 articles) that should merge into another topic?
- Are there oversized topics (20+ articles) that should split?
- Are there gaps: concepts mentioned frequently but with no dedicated article?

Record findings in INDEX.json `needs_attention` and per-topic `gaps` arrays.

### 6. Regenerate INDEX.json

Rebuild INDEX.json from the current state of all wiki files:

- Update every topic's `articles` array with current articles (slug, title, summary, path, sources, backlinks, status, dates)
- Update every topic's `summaries` array
- Update `connections` array
- Recalculate all counts in `meta`
- Update `needs_attention` with current issues
- Update `recent_activity`
- Set `meta.last_compiled` to today

### 7. Report

Tell the user:

- New concept articles created (with one-line descriptions)
- Existing articles updated
- New connections identified
- Topic health issues (orphans, thin topics, gaps)
- Suggested next steps (lint for deeper health check, or specific gaps to research)

---

## Idempotency

The compile protocol is idempotent. Running it multiple times produces the same result. It reads current state, diffs against what should exist, and only creates/updates what's needed. It never deletes articles (that's a manual decision or lint recommendation).

## Compile vs. Ingest

Ingest creates summaries from raw sources. Compile creates concept articles from summaries. Never create concept articles during ingest. The separation ensures compile can see patterns across ALL sources before deciding what deserves its own article.
