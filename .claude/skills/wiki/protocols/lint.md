# Lint Protocol

Audit the wiki's health and suggest improvements. The self-healing mechanism that keeps the knowledge base clean and evolving.

---

## Trigger

User mentions health, gaps, issues, cleanup, audit, or evolution. Also good to run periodically after large ingest/compile cycles.

## Pre-Load

Read `references/obsidian-cli-ref.md` for CLI commands when Obsidian is running.

---

## Checks

### Structural Checks

**Via Obsidian CLI (when available, preferred):**

```bash
# Notes with no incoming links
obsidian orphans vault="Vault"

# Broken wikilinks pointing to nonexistent files
obsidian unresolved vault="Vault"

# Tag distribution
obsidian tags sort=count counts vault="Vault"
```

**Fallback (when CLI unavailable):**

- Scan all .md files for [[wikilinks]]
- Cross-reference against actual file paths
- Report broken links and orphan notes

### Content Checks

Read INDEX.json and spot-check articles:

- **Stale summaries:** `date_ingested` is old but `date_published` of source is even older. The field may have moved on.

  ```bash
  cat Vault/INDEX.json | jq '[.topics[] | {topic: .key, stale: [.summaries[] | select(.date_published != "unknown" and .date_published < "THRESHOLD")]}] | map(select(.stale | length > 0))'
  ```

- **Thin articles:** Concept articles under 200 words. Need expansion or merging.
  Read articles flagged as potentially thin and check word count.

- **Missing cross-references:** Read a sample of articles and look for concept terms that appear in the text but are not wrapped in [[wikilinks]].

- **Duplicate coverage:** Multiple articles in the same topic covering the same concept. Check for overlapping titles, summaries, and key_concepts.

- **Orphan topics:** Topics with only 1-2 articles. Consider merging into a related topic.

### Temporal Health

Per-topic freshness analysis:

```bash
# For each topic, find the newest and oldest source
cat Vault/INDEX.json | jq '[.topics | to_entries[] | {topic: .key, newest: ([.value.summaries[].date_published] | map(select(. != "unknown")) | max // "none"), oldest: ([.value.summaries[].date_published] | map(select(. != "unknown")) | min // "none"), decay_days: (.value.relevance_decay_days // 180)}]'
```

Flag topics where the newest source is older than the topic's `relevance_decay_days`.

### Evolution Suggestions

- **Concept gaps:** Terms mentioned in 3+ articles but with no dedicated concept article. Scan article bodies for frequently occurring terms not covered by existing articles.
- **Connection opportunities:** Articles in different topics that reference similar concepts but have no connection article bridging them.
- **New article candidates:** Concept clusters emerging from recent sources that warrant their own articles.
- **Topic splits:** Topics with 20+ articles that may benefit from subdivision.
- **Topic merges:** Small related topics (3 or fewer articles each) that overlap significantly.

---

## Output

Write findings to `Vault/output/lint-report-YYYY-MM-DD.md`:

```markdown
---
title: "Wiki Health Report"
type: lint-report
date_created: YYYY-MM-DD
---

# Wiki Health Report - YYYY-MM-DD

## Summary

- Total articles: N | Sources: N | Topics: N
- Issues found: N | Suggestions: N

## Structural Issues

- [list of orphans, broken links]

## Content Issues

- [stale summaries, thin articles, duplicates]

## Temporal Health

- [per-topic freshness, topics needing refresh]

## Evolution Suggestions

- [concept gaps, connection opportunities, split/merge candidates]

## Recommended Actions

1. [prioritized list of what to fix/add]
```

---

## Auto-Fix Mode

When the user says "fix them" or "clean it up", the lint protocol can automatically:

- Add missing [[wikilinks]] for mentioned concepts
- Create stub articles for concept gaps (marked `status: draft`)
- Update stale index entries
- Merge duplicate articles (keep the more comprehensive one, redirect links)
- Update INDEX.json with all changes

Always report what was auto-fixed. Never silently change content.
