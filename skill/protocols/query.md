# Query Protocol

Research questions against the wiki and produce structured answers. Auto-detects query depth from the question's complexity.

---

## Trigger

User asks any question that can be answered from wiki content. Also triggered by requests for presentations, reports, or visual maps.

---

## Depth Detection

Detect the appropriate tier from the user's language. Do not ask "which tier?" Just pick the right one.

### Quick (index-level only)

**Signals:** overview questions, existence checks, "what do we have on", "how many", "list topics"

**Process:**

1. Query INDEX.json meta and topic list:
   ```bash
   cat Vault/INDEX.json | jq '.meta'
   cat Vault/INDEX.json | jq '[.topics | to_entries[] | {topic: .key, title: .value.title, articles: .value.article_count, sources: .value.source_count}]'
   ```
2. Answer from topic-level knowledge
3. No output file. Answer inline.

### Standard (topic deep-dive)

**Signals:** specific questions, "tell me about X", "compare X and Y", "explain", "what's the relationship between"

**Process:**

1. Query INDEX.json for relevant topic(s):
   ```bash
   cat Vault/INDEX.json | jq '.topics["relevant-topic"]'
   ```
2. Read the articles listed in that topic's `articles` array
3. If needed, use Obsidian CLI for additional discovery:
   ```bash
   obsidian search:context query="specific term" limit=10
   ```
4. Synthesize answer from multiple articles
5. Check source dates against topic's `relevance_decay_days`. If best sources are older than threshold, note: "The most recent source on this is from [date]. This field moves fast, consider refreshing."
6. Write output to `Vault/output/[slugified-question].md` with frontmatter:
   ```yaml
   ---
   title: "Answer title"
   query: "Original question"
   depth: standard
   sources_used: ["[[article1]]", "[[article2]]"]
   date_created: 2026-04-05
   ---
   ```

### Deep (multi-agent research)

**Signals:** "deep dive", "comprehensive analysis", "thorough look", "write me a report", complex multi-part questions, thesis-level inquiries

**Process:**

1. Delegate INDEX.json scanning to a sub-agent (Sonnet or Haiku):
   - Pass the question context to the sub-agent
   - Sub-agent loads INDEX.json, identifies all relevant topics and articles, returns a list of file paths and brief context for each
   - This keeps the main thread's context window lean and reduces cost (cheaper model reads the large index)
2. Main session reads only the recommended wiki files
3. Dispatch additional sub-agents to read different wiki sections in parallel if needed:
   - Each agent gets a topic or subset of articles
   - Each produces structured findings with citations
4. Main session synthesizes all findings into a comprehensive answer
5. If wiki has gaps on the question, optionally use WebSearch to fill them (note which claims come from external search vs. wiki)
6. Write comprehensive output to `Vault/output/`
7. Suggest file-back if the output adds significant new knowledge

**Why sub-agent delegation matters:** INDEX.json can grow to 50K+ tokens at scale. Loading it directly into the main thread wastes expensive context window capacity. The sub-agent reads the index on a cheaper model, and the main thread only ever sees the specific articles it needs. This pattern works at any scale without changing the workflow.

---

## Output Formats

Detect the desired format from context. Default is markdown.

### Markdown Article (default)

Standard .md file in output/ with frontmatter and structured body.

### Marp Slide Deck

**Signals:** "slides", "presentation", "deck"

Load `templates/marp-deck.md` for syntax reference. Write a `.md` file with Marp frontmatter to `Vault/output/`. Viewable in Obsidian with Marp Slides plugin.

### Canvas Visualization

**Signals:** "visual map", "diagram", "canvas", "knowledge map"

Load `references/canvas-spec.md` for JSON Canvas spec. Generate a `.canvas` file in `Vault/output/` with nodes for articles and edges for relationships.

---

## Temporal Awareness

Always check dates when answering. For each source used:

- If `date_published` is within the topic's `relevance_decay_days`: use normally
- If 1-2x the decay period: include but note the age
- If beyond 2x the decay period: include only if no newer source exists, flag for re-research

For "what's the latest on X?" questions, sort by `date_published` descending and lead with the most recent.

Query INDEX.json for temporal analysis:

```bash
# Find stale articles in a topic
cat Vault/INDEX.json | jq '.topics["topic-name"].articles | sort_by(.date_updated) | .[0:5]'

# Find most recent sources
cat Vault/INDEX.json | jq '[.topics[].summaries[] | select(.date_published > "2026-03-01")] | sort_by(.date_published) | reverse'
```
