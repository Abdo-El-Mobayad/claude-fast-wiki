# INDEX.json Schema and Query Reference

The single source of navigational truth for the entire wiki. Lives at `Vault/INDEX.json`.

---

## Full Schema

```json
{
  "meta": {
    "last_compiled": "2026-04-05",
    "total_topics": 8,
    "total_articles": 47,
    "total_sources": 31,
    "total_connections": 3
  },

  "topics": {
    "topic-slug": {
      "title": "Human-Readable Topic Name",
      "description": "One-line description of what this topic covers",
      "article_count": 12,
      "source_count": 8,
      "last_updated": "2026-04-05",
      "relevance_decay_days": 180,
      "articles": [
        {
          "slug": "article-slug",
          "title": "Article Title",
          "summary": "One-line summary for index scanning",
          "path": "wiki/article-slug.md",
          "type": "article",
          "sources": 4,
          "backlinks": 8,
          "status": "active",
          "date_updated": "2026-04-05",
          "newest_source": "2026-04-02"
        }
      ],
      "summaries": [
        {
          "slug": "summary-slug",
          "title": "Original Source Title",
          "path": "wiki/summary-slug.md",
          "author": "Author Name",
          "date_published": "2026-04-02",
          "date_ingested": "2026-04-05",
          "key_concepts": ["concept-1", "concept-2"]
        }
      ],
      "connections_to": ["other-topic-slug"],
      "gaps": ["Description of missing concept (mentioned in N sources)"]
    }
  },

  "connections": [
    {
      "slug": "connection-slug",
      "title": "Connection Article Title",
      "bridges": ["topic-1", "topic-2"],
      "path": "wiki/connection-slug.md",
      "date_created": "2026-04-05"
    }
  ],

  "needs_attention": [
    {
      "type": "orphan",
      "path": "wiki/article-slug.md",
      "detail": "0 backlinks, consider linking or archiving"
    },
    {
      "type": "unprocessed",
      "count": 3,
      "detail": "3 sources in raw/ awaiting ingest"
    },
    {
      "type": "stale",
      "path": "wiki/article-slug.md",
      "detail": "Newest source is 8 months old, topic decay is 90 days"
    },
    {
      "type": "gap",
      "concept": "concept-name",
      "mentioned_in_topics": ["topic-1", "topic-2"],
      "detail": "Mentioned in 4 articles but has no dedicated page"
    },
    {
      "type": "compile_candidate",
      "concept": "concept-name",
      "source_count": 3,
      "detail": "Appears in 3 sources, ready for concept article"
    }
  ],

  "recent_activity": [
    {
      "date": "2026-04-05",
      "action": "ingested",
      "detail": "Processed 4 new sources into obsidian-knowledge-bases topic"
    },
    {
      "date": "2026-04-05",
      "action": "compiled",
      "detail": "Created 3 new concept articles, updated 2 existing"
    }
  ]
}
```

---

## Empty Index (for init)

```json
{
  "meta": {
    "last_compiled": null,
    "total_topics": 0,
    "total_articles": 0,
    "total_sources": 0,
    "total_connections": 0
  },
  "topics": {},
  "connections": [],
  "needs_attention": [],
  "recent_activity": []
}
```

---

## jq Query Reference

### Navigation Queries

```bash
# Meta stats
cat Vault/INDEX.json | jq '.meta'

# List all topics with counts
cat Vault/INDEX.json | jq '[.topics | to_entries[] | {topic: .key, title: .value.title, articles: .value.article_count, sources: .value.source_count}]'

# Get a specific topic's full entry
cat Vault/INDEX.json | jq '.topics["topic-slug"]'

# List all articles in a topic
cat Vault/INDEX.json | jq '.topics["topic-slug"].articles'

# List all summaries in a topic
cat Vault/INDEX.json | jq '.topics["topic-slug"].summaries'

# List all connection articles
cat Vault/INDEX.json | jq '.connections'
```

### Search Queries

```bash
# Search articles by title (case-insensitive)
cat Vault/INDEX.json | jq '[.topics[].articles[] | select(.title | test("keyword"; "i"))]'

# Search summaries by author
cat Vault/INDEX.json | jq '[.topics[].summaries[] | select(.author | test("name"; "i"))]'

# Find articles mentioning a concept in their summary
cat Vault/INDEX.json | jq '[.topics[].articles[] | select(.summary | test("concept"; "i"))]'

# Find all tracked summary slugs (to detect duplicates during ingest)
cat Vault/INDEX.json | jq '[.topics[].summaries[].slug]'
```

### Temporal Queries

```bash
# Articles updated in the last 30 days
cat Vault/INDEX.json | jq '[.topics[].articles[] | select(.date_updated > "2026-03-06")]'

# Sources published in the last 90 days
cat Vault/INDEX.json | jq '[.topics[].summaries[] | select(.date_published > "2026-01-05" and .date_published != "unknown")]'

# Stale articles (not updated in 2026)
cat Vault/INDEX.json | jq '[.topics[].articles[] | select(.date_updated < "2026-01-01")]'

# Per-topic freshness (newest source per topic)
cat Vault/INDEX.json | jq '[.topics | to_entries[] | {topic: .key, newest: ([.value.summaries[].date_published] | map(select(. != "unknown")) | max // "none"), decay: (.value.relevance_decay_days // 180)}]'
```

### Health Queries

```bash
# Get needs_attention items
cat Vault/INDEX.json | jq '.needs_attention'

# Get gaps across all topics
cat Vault/INDEX.json | jq '[.topics | to_entries[] | {topic: .key, gaps: .value.gaps} | select(.gaps | length > 0)]'

# Find HTML documents (type check)
cat Vault/INDEX.json | jq '[.topics[].articles[] | select(.type == "html-document")]'

# Find draft articles
cat Vault/INDEX.json | jq '[.topics[].articles[] | select(.status == "draft")]'

# Topics with no recent activity
cat Vault/INDEX.json | jq '[.topics | to_entries[] | select(.value.last_updated < "2026-01-01") | .key]'
```

---

## Update Patterns

### After Ingest (adding a summary)

```
1. Add entry to topics[topic-slug].summaries[]
2. Increment topics[topic-slug].source_count
3. Increment meta.total_sources
4. Update topics[topic-slug].last_updated
5. Add to recent_activity[]
6. If new topic: add full topic object to topics{}. Increment meta.total_topics
7. If concept appears in 2+ sources: add to needs_attention[] as compile_candidate
```

### After Compile (adding/updating articles)

```
1. Add/update entries in topics[topic-slug].articles[]
2. Update topics[topic-slug].article_count
3. Update meta.total_articles
4. Update topics[topic-slug].last_updated
5. Add/update connections[] if cross-topic articles created
6. Update meta.total_connections
7. Refresh needs_attention[] (remove resolved, add new)
8. Refresh per-topic gaps[]
9. Add to recent_activity[]
10. Set meta.last_compiled to today
```

### After File-Back (incorporating output)

```
1. Add article entry to topics[topic-slug].articles[]
2. Update counts (article_count, meta.total_articles)
3. Add to recent_activity[]
4. Update topics[topic-slug].last_updated
```

### After Lint (recording findings)

```
1. Replace needs_attention[] with current findings
2. Refresh per-topic gaps[]
3. Add to recent_activity[] noting lint run
```
