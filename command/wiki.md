---
description: LLM-maintained knowledge base. Drop sources, AI compiles a living wiki, query it, answers file back. Knowledge compounds.
argument-hint: [natural language - just say what you need]
---

# /wiki - Knowledge Base

Your AI-maintained knowledge base powered by Claude Code and Obsidian. Talk naturally. The system detects what you need and routes to the right workflow.

## How It Works

You drop source material (articles, PDFs, notes, bookmarks) into `Vault/raw/`. The AI processes them into structured wiki summaries, builds concept articles that synthesize across sources, and maintains a queryable index. You ask questions in natural language. The best answers file back into the wiki. Knowledge compounds over time.

## Activation

1. Read the SKILL.md at the skill location for full system context, file registry, intent detection, and conventions.
2. Detect what the user wants from their natural language (see Intent Detection in SKILL.md).
3. Load the relevant protocol and template files based on detected intent.
4. Execute the protocol.

## Quick Reference

The vault lives at `Vault/` relative to your project root. Structure:

```
Vault/
├── raw/          # Drop sources here. Processing queue (cleared after ingest).
├── wiki/         # AI-maintained wiki. Flat folder, no subfolders.
├── INDEX.json    # The brain. Single queryable index of all topics + articles.
└── output/       # Staging area for query results before filing back.
```

## What You Can Say

### Adding Knowledge

```
/wiki process the new stuff in raw/
/wiki I dropped some articles in there
/wiki add this URL to the knowledge base
```

The AI reads each source, creates a structured summary in wiki/, updates INDEX.json, and deletes the processed raw file. The summary preserves key claims, data points, quotes, and connections.

### Organizing

```
/wiki organize the wiki
/wiki there are connections we're missing
/wiki rebuild the indexes
```

The AI scans all summaries for recurring concepts, creates synthesis articles that connect ideas across sources, builds cross-topic connections, and identifies gaps.

### Asking Questions

```
/wiki what do we know about [topic]?
/wiki compare X and Y based on what we have
/wiki give me a deep analysis of [topic]
/wiki write me a presentation on [topic]
/wiki show me a knowledge map
```

Three depth tiers are auto-detected:

- **Quick:** Index-level overview, answered inline
- **Standard:** Reads relevant articles, writes output to Vault/output/
- **Deep:** Multi-agent research across the full wiki, fills gaps with web search

### Health Checks

```
/wiki how healthy is the wiki?
/wiki find gaps and fix them
/wiki what's missing?
/wiki clean up
```

Audits structural integrity (orphans, broken links), content quality (thin articles, stale sources), temporal health (topics needing refresh), and suggests evolution paths.

### Saving Answers

```
/wiki save that last answer
/wiki that should be a permanent article
/wiki keep that
```

Valuable query outputs become permanent wiki articles with proper frontmatter, cross-links, and index entries.

## If No Argument Given

If you just type `/wiki` with no argument:

1. Check if `Vault/INDEX.json` exists
2. If yes: show status (topics, articles, sources, recent activity, any issues needing attention)
3. If no: offer to initialize the vault structure

## First Time Setup

If the vault doesn't exist yet:

1. Copy the `vault-template/` contents to `Vault/` in your project root
2. Or just say `/wiki set up the knowledge base` and the AI creates the structure for you
3. Start dropping source files into `Vault/raw/`
4. Say `/wiki process the new stuff`

## The Compounding Loop

```
Drop sources -> Ingest -> Compile -> Query -> File back -> Repeat
     raw/         wiki/     wiki/     output/    wiki/
```

Each cycle makes the wiki smarter. Summaries feed concept articles. Concept articles improve query answers. Good answers file back as permanent knowledge. The knowledge base grows in quality, not just volume.
