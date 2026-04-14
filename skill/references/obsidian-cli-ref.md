# Obsidian CLI Reference

Essential commands for wiki operations. Requires Obsidian 1.12+ with CLI enabled (Settings > General > CLI).

**Important:** The CLI is a remote control for a running Obsidian instance. If Obsidian is not running, it launches automatically. On Windows, do NOT run from admin terminals (silent failures).

---

## Syntax

```
obsidian [command] [param=value] [flag]
```

- Parameters: `key=value`, quotes for spaces: `key="value with spaces"`
- Multi-vault: `vault="VaultName"` as first parameter
- Output formats: `format=json|csv|tsv|md|paths|text|tree|yaml`
- Clipboard: `--copy` copies output to clipboard

---

## File Operations

```bash
# List all notes
obsidian files vault="Vault"

# Show folder tree
obsidian folders vault="Vault"

# Read a note
obsidian read file="path/to/note" vault="Vault"

# Create a note
obsidian create name="path/to/note" content="# Title\nContent" vault="Vault"

# Create from template
obsidian create name="path" template="TemplateName" vault="Vault"

# Append to a note
obsidian append file="path" content="New content" vault="Vault"

# Prepend (after frontmatter)
obsidian prepend file="path" content="New content" vault="Vault"

# Move (auto-updates wikilinks)
obsidian move file="source" to="destination/" vault="Vault"

# Delete (to trash)
obsidian delete file="path" vault="Vault"
```

## Search

```bash
# Full-text search
obsidian search query="search term" vault="Vault"

# Search with context (like grep -C)
obsidian search:context query="term" limit=10 vault="Vault"

# Search with JSON output (for scripting)
obsidian search query="term" format=json vault="Vault"
```

## Link Analysis

```bash
# Outgoing links from a note
obsidian links file="path" vault="Vault"

# Incoming links (backlinks) to a note
obsidian backlinks file="path" vault="Vault"

# Broken wikilinks (targets don't exist)
obsidian unresolved vault="Vault"

# Orphan notes (no incoming links)
obsidian orphans vault="Vault"
```

## Tags

```bash
# List all tags
obsidian tags vault="Vault"

# Tags sorted by frequency
obsidian tags sort=count counts vault="Vault"

# Files with a specific tag
obsidian tag tag="#tagname" vault="Vault"

# Bulk rename a tag
obsidian tags:rename old=oldname new=newname vault="Vault"
```

## Properties (Frontmatter)

```bash
# View properties
obsidian properties file="path" vault="Vault"

# Set a property
obsidian property:set file="path" name="key" value="val" vault="Vault"

# Remove a property
obsidian property:remove file="path" name="key" vault="Vault"
```

---

## Performance Benchmarks

On a 24,773 file / 5.1 GB vault:

| Operation           | CLI Time | grep Time | Speedup |
| ------------------- | -------- | --------- | ------- |
| Read file           | 0.73s    | N/A       | N/A     |
| Search with context | 0.27s    | 1.95s     | 6x      |
| Find orphan notes   | 0.26s    | 15.6s     | 54x     |
| Append operation    | 0.25s    | N/A       | N/A     |
| Tasks query         | 0.25s    | N/A       | N/A     |

**Always prefer CLI over grep/file-scanning when Obsidian is running.**

---

## When CLI Is Unavailable

Fall back to:

- Direct file reading via the Read tool
- Grep tool for content search
- Glob tool for file discovery
- Manual wikilink parsing for link analysis

The CLI is faster and more accurate (it uses Obsidian's internal indexes), but the system works without it.
