# Ingest Protocol

Process new raw sources into structured wiki summaries. This is the entry point for all new knowledge entering the system.

---

## Trigger

User has added new files to `Vault/raw/` or asks to process/ingest new sources.

## Pre-Load

Read `templates/source-summary.md` for the summary format and `templates/index-format.md` for INDEX.json schema before executing.

---

## Steps

### 1. Discover Unprocessed Sources

Everything in `Vault/raw/` is unprocessed by definition (processed files are deleted after ingest). Scan `raw/` for all files. Supported types: `.md`, `.html`, `.txt`, `.pdf`. Ignore `assets/` subfolder (images, not standalone sources). Never scan `.obsidian/` (Obsidian's internal config folder).

For dedup, query INDEX.json for existing slugs to avoid creating duplicate summaries:

```bash
cat Vault/INDEX.json | python -c "import json,sys; d=json.load(sys.stdin); print([s['slug'] for t in d['topics'].values() for s in t.get('summaries',[])])"
```

### 2. Read Each New Source

For each unprocessed file:

- Read the full content
- Extract the publication date from byline, metadata, frontmatter, or page date. If none found, set `date_published: "unknown"` and flag it.
- Identify the author if present
- Determine which topic(s) this source belongs to. Check existing topics in INDEX.json first. Create a new topic only if no existing topic fits.

### 3. Create Summary

Create a summary file at `Vault/wiki/[slugified-source-name].md` (flat in wiki/, no subfolders).

Use the template from `templates/source-summary.md`. The summary must:

- Have complete frontmatter with all three dates and `source_url` (original URL, not a wikilink to raw/)
- Extract key concepts as a list (these become compile candidates)
- Preserve the most important claims, data points, and quotes
- Be structured with clear section headers
- Be comprehensive enough that someone reading only the summary understands the source's contribution

For multi-topic sources, list all relevant topics in frontmatter. The file still lives flat in wiki/.

### 4. Handle Non-Markdown Files

**HTML files:** Keep the .html flat in wiki/ alongside its companion. Create both:

- `Vault/wiki/[name].html` (copy from raw/, never modify)
- `Vault/wiki/[name].md` (companion with frontmatter + extracted insights)

Set `type: html-document` and `html_path` in the companion frontmatter.

**PDF files:** Same companion pattern. Copy PDF to wiki/, create .md companion with extracted content and insights.

### 5. Create Topic Entry If Needed

If a new topic is needed:

- Add the topic entry to INDEX.json with title, description, empty articles array, and the new summary
- No folder creation needed (flat wiki structure)

### 6. Update INDEX.json

After processing all sources, update INDEX.json:

- Add summary entries to the appropriate topic's `summaries` array
- Update topic `source_count` values
- Update `meta.total_sources`
- Add to `recent_activity` array
- If any concepts appear in 2+ sources across topics, note them in `needs_attention` as compile candidates
- Update `meta.last_compiled` timestamp

### 7. Clean Up raw/

After all sources are successfully processed and INDEX.json is updated, delete the processed files from `raw/`. Only leave files that failed processing or were skipped due to errors. The `raw/assets/` subfolder (images) is kept permanently.

### 8. Report

Tell the user:

- How many sources were processed
- Which topics they were filed under (new topics highlighted)
- Key concepts found across multiple sources (compile candidates)
- Any sources with unknown publication dates (may need manual dating)
- How many files were cleaned from raw/
- Suggest running compile if there are cross-source concepts to synthesize

---

## INDEX.json Query Best Practices

At small scale (under ~50 articles), you can read INDEX.json in full. At larger scale (100+ articles, 50K+ tokens), use targeted queries instead.

### Query Tiers

| Operation           | Method                                  | Cost         |
| ------------------- | --------------------------------------- | ------------ |
| Meta stats          | `d['meta']`                             | ~30 tokens   |
| Topic list + counts | Extract topic keys with `source_count`  | ~2K tokens   |
| Single topic lookup | `d['topics']['slug']`                   | ~1-3K tokens |
| Source dedup check  | Collect all `slug` values               | ~5K tokens   |
| Find by concept     | Filter summaries by `key_concepts`      | Variable     |
| Full read           | OK at small scale, avoid at large scale | Varies       |

### Per-Protocol Rules

- **Ingest:** Everything in raw/ is unprocessed. Check slugs for dedup + target topic section only
- **Compile:** Read 1-2 topic sections at a time
- **Query:** Read meta first, drill into relevant topic(s)
- **Lint:** Meta + `needs_attention` + selective topic checks
- **Updates:** Python read-modify-write (load full JSON in Python, modify in memory, write back). No need to read into LLM context first.

### Standard Query Pattern

```bash
cat Vault/INDEX.json | python -c "
import json,sys
d=json.load(sys.stdin)
# ... targeted extraction here
print(json.dumps(result, indent=2))
"
```

---

## Edge Cases

- **Duplicate sources:** If a file in raw/ has the same title/content as an existing summary, skip it and note the duplicate.
- **Very large sources:** For sources over 5000 words, focus the summary on the most novel/important sections. Note what was omitted.
- **Image-heavy sources:** Reference images by their relative path in raw/assets/ (this subfolder is permanent, not cleared during cleanup). Do not copy images to wiki/.
- **Sources spanning many topics:** Pick the primary topic for the summary location. List all relevant topics in frontmatter. The compile step will create cross-topic connections.
- **URL-only / thin sources (tools, repos, links):** Sources that are just a URL with a short description do NOT get their own wiki summary file. Instead, append them as a row in the appropriate master directory file:
  - **Tools / product URLs:** `wiki/master-tools-directory.md`
  - **GitHub repos:** `wiki/master-github-repos.md`
    These master files follow the same flat-index principle as INDEX.json: one consolidated file is better than many one-paragraph entries. Only create a standalone wiki summary for sources with substantial content.
