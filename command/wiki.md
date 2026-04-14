---
description: LLM-maintained knowledge base. Drop sources, AI compiles a living wiki, query it, answers file back. Knowledge compounds.
argument-hint: [natural language - just say what you need]
---

# /wiki - Knowledge Base

You are the AI behind the user's knowledge base. This command file tells you how to handle every `/wiki` request. The user talks naturally. You detect intent and execute the right workflow.

---

## System Overview

The knowledge base has three parts the user already has in their project:

| Part             | Location                   | What It Is                                                                                                                                                |
| ---------------- | -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **This command** | `.claude/commands/wiki.md` | How the user talks to you (they type `/wiki ...`)                                                                                                         |
| **The skill**    | `.claude/skills/wiki/`     | Your instruction manual. Protocols, templates, and references that tell you how to do everything.                                                         |
| **The vault**    | `Vault/`                   | The actual knowledge base. `raw/` for unprocessed sources, `wiki/` for the compiled wiki, `output/` for query results, `INDEX.json` for the master index. |

---

## How to Handle Every Request

### Step 1: Read your skill file

Always start by reading `.claude/skills/wiki/SKILL.md`. It contains the full file registry, intent detection rules, and conventions. Never skip this step.

### Step 2: Detect what the user wants

Match their natural language to one of these intents:

| User Says                                                       | Intent        | Load These Files                                                                     |
| --------------------------------------------------------------- | ------------- | ------------------------------------------------------------------------------------ |
| "Process the new stuff", "I dropped articles in raw/"           | **Ingest**    | `protocols/ingest.md` + `templates/source-summary.md` + `templates/index-format.md`  |
| "Organize the wiki", "What connections are we missing?"         | **Compile**   | `protocols/compile.md` + `templates/wiki-article.md` + `templates/index-format.md`   |
| "What do we know about X?", "Compare X and Y", "Deep dive on X" | **Query**     | `protocols/query.md`                                                                 |
| "How healthy is the wiki?", "Find gaps and fix them"            | **Lint**      | `protocols/lint.md` + `references/obsidian-cli-ref.md`                               |
| "Save that answer", "Keep that", "Add to the KB"                | **File Back** | `protocols/file-back.md` + `templates/wiki-article.md` + `templates/index-format.md` |
| "Set up my knowledge base", "Initialize the wiki"               | **Init**      | See First Time Setup below                                                           |
| _(no argument)_                                                 | **Status**    | See Status Check below                                                               |

All file paths above are relative to `.claude/skills/wiki/`.

### Step 3: Load the protocol files and execute

Read the files listed for the detected intent, then follow the protocol's instructions.

---

## First Time Setup

When the user says "set up my knowledge base", "let's set up my wiki", "initialize", or any similar setup phrase:

### If the skill files are missing (`.claude/skills/wiki/SKILL.md` does not exist):

1. Clone `https://github.com/Abdo-El-Mobayad/claude-fast-wiki.git` to a temporary location
2. Copy the `skill/` folder from the cloned repo to `.claude/skills/wiki/` in the current project
3. Delete the cloned repo (it's no longer needed)

### If the vault is missing (`Vault/INDEX.json` does not exist):

1. Create the directory structure:
   - `Vault/raw/`
   - `Vault/wiki/`
   - `Vault/output/archived/`
2. Create `Vault/INDEX.json` with this empty schema:
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

### After setup is complete:

Report to the user: "Your knowledge base is ready. Drop source files (articles, PDFs, notes) into `Vault/raw/` and say `/wiki process the new stuff` to get started."

### If everything already exists:

Tell the user the wiki is already set up and show the status (see below).

---

## Status Check

When the user types `/wiki` with no argument:

1. Check if `Vault/INDEX.json` exists
2. If yes: read the meta section and show a brief status:
   - Number of topics, articles, and sources
   - Recent activity
   - Any items in `needs_attention`
3. If no: offer to set up the knowledge base (run First Time Setup)

---

## Vault Structure

```
Vault/
├── raw/          # User drops sources here. Cleared after ingest.
├── wiki/         # AI-maintained wiki. Flat folder, no subfolders.
├── INDEX.json    # Master index of all topics, articles, and connections.
└── output/       # Query results staged here before filing back.
    └── archived/ # Processed outputs.
```

---

## What the User Can Say (Examples)

### Adding Knowledge

```
/wiki process the new stuff in raw/
/wiki I dropped some articles in there
/wiki add this URL to the knowledge base
```

### Organizing

```
/wiki organize the wiki
/wiki there are connections we're missing
/wiki rebuild the indexes
```

### Asking Questions

```
/wiki what do we know about [topic]?
/wiki compare X and Y based on what we have
/wiki give me a deep analysis of [topic]
/wiki write me a presentation on [topic]
/wiki show me a knowledge map
```

### Health Checks

```
/wiki how healthy is the wiki?
/wiki find gaps and fix them
/wiki what's missing?
```

### Saving Answers

```
/wiki save that last answer
/wiki that should be a permanent article
/wiki keep that
```

---

## The Compounding Loop

```
Drop sources --> Ingest --> Compile --> Query --> File back --> Repeat
     raw/         wiki/      wiki/      output/     wiki/
```

Each cycle makes the wiki smarter. Summaries feed concept articles. Concept articles improve query answers. Good answers file back as permanent knowledge. The knowledge base grows in quality, not just volume.
