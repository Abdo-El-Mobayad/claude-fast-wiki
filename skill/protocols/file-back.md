# File-Back Protocol

Incorporate valuable query outputs back into the wiki so knowledge compounds. This is what makes the system grow smarter with every interaction.

---

## Trigger

User indicates a query output should become permanent wiki knowledge: "save that", "keep it", "add to the KB", "that's a good article". Also triggered when reviewing output/ and deciding what to keep.

## Pre-Load

Read `templates/wiki-article.md` for article format and `templates/index-format.md` for INDEX.json schema.

---

## Steps

### 1. Assess the Output

Read the output file from `Vault/output/`. Determine its knowledge value:

- **New concept:** Output covers a concept not yet in the wiki. Create a new article.
- **Update existing:** Output adds new insights to an existing article. Merge into existing.
- **Connection:** Output bridges concepts across topics. Create a connection article.
- **Discard:** Output is ephemeral (simple lookup, one-off answer). Don't file back.

If the value is unclear, ask the user.

### 2. Determine Placement

- Check which topic(s) the output relates to via INDEX.json
- For new concepts: place flat in `wiki/`
- For updates: identify the existing article to merge into
- For connections: place flat in `wiki/` with `conn-` prefix

### 3. Create or Update the Wiki Article

**If new article:**

- Create `Vault/wiki/[concept-slug].md`
- Use the wiki-article template frontmatter
- Set `date_created` and `date_updated` to today
- Add `sources` linking back to any summaries the output referenced
- Add `related` links to other concept articles
- Body: restructure the output into a proper wiki article (not just a pasted answer)

**If updating existing:**

- Read the existing article
- Integrate new insights, preserving existing content
- Add new source links if the output referenced sources not yet linked
- Update `date_updated` to today
- Do not overwrite existing content. Augment it.

**If connection article:**

- Create `Vault/wiki/conn-[bridge-name].md`
- Set `bridges` frontmatter to the connected topics
- Link to relevant articles in both topics

### 4. Update INDEX.json

- Add or update article entry in the appropriate topic's `articles` array
- Add connection entry if applicable
- Update `meta` counts
- Add to `recent_activity`

### 5. Archive the Output

Move the processed output file to `Vault/output/archived/`:

- If the file was fully incorporated, move it
- If only partially used, keep the original and note what was extracted

### 6. Report

Tell the user:

- What was filed back (new article, update, or connection)
- Where it was placed
- What cross-links were created
- Current wiki stats after the addition

---

## What NOT to File Back

- Simple factual lookups ("what year was X founded?")
- Restatements of existing wiki content
- Exploratory questions that didn't produce new insight
- Lint reports (these are diagnostic, not knowledge)

Only file back outputs that genuinely add new knowledge, synthesis, or connections to the wiki. The wiki should grow in quality, not just volume.
