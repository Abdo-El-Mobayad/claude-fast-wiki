# Obsidian Knowledge Base Skill

You maintain a living knowledge base in an Obsidian vault. The user collects raw sources. You organize them into a structured wiki with indexes, concept articles, and cross-links. The user asks questions in natural language. You research the wiki and produce answers. The best answers file back into the wiki. Knowledge compounds over time.

**You write the wiki. The user steers. Every answer compounds.**

---

## Vault Location

The vault lives at `Vault/` relative to your project root. All paths in this skill are relative to the vault root unless stated otherwise.

```
Vault/
├── .obsidian/              # Obsidian config (auto-created by Obsidian). Never modify.
├── raw/                    # Processing queue. Unprocessed sources land here, get deleted after ingest.
│   ├── assets/             # Downloaded images referenced by sources
│   └── [source-files]      # .md, .html, .txt, .pdf clipped or dumped
├── wiki/                   # YOUR territory. Flat folder. You write and maintain everything here.
│   └── [all-wiki-files]    # Summaries, concept articles, directories — all flat, no subfolders.
├── output/                 # Staging area. Query results before filing back.
│   └── archived/           # Processed outputs
└── INDEX.json              # The brain. Single queryable index of all topics + articles.
```

**Flat wiki design:** All wiki files live directly in `wiki/`. No `topics/`, `connections/`, or `summaries/` subfolders. INDEX.json is the sole organizer. Topic membership, article type, and cross-topic connections are tracked in INDEX.json metadata, not folder paths. This optimizes for LLM navigation: one INDEX.json read tells you where everything is.

**Note:** `.obsidian/` is created automatically when you open the Vault folder in Obsidian. It contains app settings, plugin configs, and workspace state. Never read, modify, or index files in this folder. Exclude it from all scans and ingests.

---

## File Registry

| File                             | Purpose                                             | ~Tokens | Load When                                           |
| -------------------------------- | --------------------------------------------------- | ------- | --------------------------------------------------- |
| `protocols/ingest.md`            | Process raw sources into wiki summaries             | ~800    | User mentions new sources, clipping, or processing  |
| `protocols/compile.md`           | Build wiki structure, concept articles, cross-links | ~1000   | User mentions organizing, rebuilding, restructuring |
| `protocols/query.md`             | 3-tier query system (Quick/Standard/Deep)           | ~800    | User asks any question about wiki content           |
| `protocols/lint.md`              | Health checks, self-healing, evolution              | ~800    | User mentions health, gaps, issues, cleanup         |
| `protocols/file-back.md`         | Incorporate outputs back into wiki                  | ~400    | User wants to save an answer to the wiki            |
| `templates/wiki-article.md`      | Concept article frontmatter + body structure        | ~300    | Creating concept articles (ingest/compile)          |
| `templates/source-summary.md`    | Source summary frontmatter + body structure         | ~300    | Creating source summaries (ingest)                  |
| `templates/index-format.md`      | INDEX.json schema and query reference               | ~500    | Reading, querying, or rebuilding the index          |
| `templates/marp-deck.md`         | Marp slide deck syntax reference                    | ~500    | Generating slide deck output                        |
| `references/obsidian-cli-ref.md` | Essential CLI commands for wiki operations          | ~600    | Using Obsidian CLI for search/link analysis         |
| `references/canvas-spec.md`      | JSON Canvas format for visual maps                  | ~500    | Generating canvas visualizations                    |

---

## Intent Detection

The user talks naturally. You detect intent and route to the right protocol. Never ask the user to use specific subcommands.

### Ingest Signals

Load: `protocols/ingest.md` + `templates/source-summary.md` + `templates/index-format.md`

- "I just clipped some articles into the vault"
- "Process the new stuff in raw/"
- "I dropped some papers in there, take a look"
- "There's new research to process"
- User pastes a URL and says "add this to the KB"
- Any mention of new content arriving in raw/

### Compile Signals

Load: `protocols/compile.md` + `templates/wiki-article.md` + `templates/index-format.md`

- "Organize the wiki"
- "The wiki needs restructuring"
- "Can you rebuild the indexes?"
- "I think there are connections we're missing"
- After a large ingest, compile is the natural next step

### Query Signals

Load: `protocols/query.md`

- Any question about topics the wiki covers
- "What do we know about X?"
- "Give me an overview of the KB"
- "Compare X and Y based on what we have"
- "Write me a presentation on X" (query + Marp output)
- "I need a deep analysis of X" (deep tier)
- "Show me a knowledge map" (query + canvas output)

### Lint Signals

Load: `protocols/lint.md` + `references/obsidian-cli-ref.md`

- "How healthy is the wiki?"
- "Check for any issues"
- "What's missing? What should we add?"
- "Find gaps and fix them"
- "Clean up the wiki"

### File-Back Signals

Load: `protocols/file-back.md` + `templates/wiki-article.md` + `templates/index-format.md`

- "That last answer was really good, save it"
- "Add this to the knowledge base"
- "This should be a permanent article"
- "Keep that" after reviewing a query output

### Init Signals

No protocol file needed. Execute directly:

- "Set up the knowledge base"
- "Create the vault structure"
- Create `Vault/raw/`, `Vault/wiki/`, `Vault/output/archived/`
- Initialize empty `Vault/INDEX.json` with empty schema
- Report ready

### Implicit Routing

When no explicit signal is given, infer from context:

- User drops content/URLs while KB is active -> Ingest
- User asks a question while KB is active -> Query (auto-detect depth)
- User says "nice" or "keep that" after output -> File-back
- After a large ingest session -> Suggest compile
- After compile -> Suggest lint

---

## Workflows

### First Use

```
1. Init (create vault structure)
2. User clips sources into raw/
3. Ingest (process raw/ into wiki/ summaries)
4. Compile (build concept articles + cross-links)
5. Lint (health check)
```

### Incremental Growth

```
1. User clips new source into raw/
2. Ingest (process only new sources)
3. Compile (update wiki with new concepts)
4. User asks questions
5. File-back (best outputs become wiki articles)
```

### Research Session

```
1. User asks complex question -> Query (Deep)
2. User reviews output in Obsidian
3. File-back (incorporate into wiki)
4. Lint (check wiki health after additions)
```

---

## Core Conventions

### Wikilinks

All internal links use `[[wikilinks]]` for Obsidian compatibility. Never use standard markdown links `[text](path)` for internal wiki references. Wikilinks enable graph view, backlinks panel, and auto-update on file moves via Obsidian CLI.

### Frontmatter

Every wiki file has YAML frontmatter. Frontmatter is the source of truth for all state. INDEX.json is a compiled cache that can be regenerated from frontmatter at any time.

### Three Dates Per File

Every wiki file carries three temporal markers:

- `date_published` - When the ORIGINAL source was published (summaries only)
- `date_ingested` - When we processed it into the KB
- `date_updated` - When this article was last modified

The LLM sets these automatically. Never rely on file system timestamps (git/sync breaks them).

### Relevance Decay

Topics can set `relevance_decay_days` in INDEX.json. During queries, prefer recent sources and flag stale content. Default: 180 days. Fast-moving fields (AI, agents): 90 days.

### HTML Companion Pattern

Non-markdown files (.html, .pdf) get a companion .md file with the same name. The companion has frontmatter, extracted key insights, and wikilinks. The original file is never modified. The companion makes non-markdown content queryable and linkable.

### INDEX.json

Single JSON file at `Vault/INDEX.json`. Contains all topics, articles, summaries, connections, gaps, needs_attention, and recent_activity. Can be read in full at small scale or queried via `jq`/Python for specific slices at larger scale. See `templates/index-format.md` for schema and queries.

### Master Directory Files

Thin sources (just a URL + short description) do not get standalone wiki files. They go into consolidated master directories:

- **Tools / product URLs:** `wiki/master-tools-directory.md`
- **GitHub repos:** `wiki/master-github-repos.md`

One consolidated, scannable file beats hundreds of one-paragraph entries. Only sources with substantial content (articles, guides, tutorials, research, threads with real insights) get their own wiki summary file.

### Raw Is a Processing Queue

`raw/` is a staging area for unprocessed source material. Once a source has been fully ingested into wiki/ (summary created, INDEX.json updated), the raw file should be deleted from `raw/`. The wiki summary becomes the permanent record. The original source URL is preserved in the summary's `source_url` frontmatter field for provenance.

**After each ingest run:** delete all successfully processed files from `raw/`. Only unprocessed or failed files should remain.

---

## Anti-Patterns

- **Never touch `.obsidian/`.** This folder is Obsidian's internal config. Never read, modify, scan, or index anything in it.
- **Never skip index updates.** Every wiki change must update INDEX.json. An out-of-sync index causes the LLM to miss content or create duplicates.
- **Never create concept articles during ingest.** Ingest creates summaries only. Concept articles are the compile step's job, because they require seeing patterns across multiple sources.
- **Never convert HTML to markdown.** HTML files are kept as-is with companion .md files. The visual format is the value.
- **Never use grep for structural queries when Obsidian CLI is available.** The CLI is 54x faster for orphan detection and 6x faster for search. It accesses Obsidian's internal indexes.
- **Never answer from stale sources without saying so.** If the best sources are old relative to the topic's decay threshold, explicitly note the age.
- **Never file back ephemeral answers.** Only file back outputs that add genuine new knowledge. One-off lookups and simple factual answers don't need to be permanent articles.
- **Never put query outputs directly in wiki/.** Outputs go to output/ first. Filing back is a deliberate step that adds frontmatter, cross-links, and index entries.
