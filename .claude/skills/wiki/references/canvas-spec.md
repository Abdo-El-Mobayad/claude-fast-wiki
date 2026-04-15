# JSON Canvas Specification

Use this when generating `.canvas` files for visual knowledge maps. Canvas files render natively in Obsidian.

**Spec version:** 1.0
**File extension:** `.canvas`
**Location:** Save to `Vault/output/[name].canvas`

---

## Top-Level Structure

```json
{
  "nodes": [],
  "edges": []
}
```

## Node Types

### Text Node

```json
{
  "id": "unique-id",
  "type": "text",
  "x": 0,
  "y": 0,
  "width": 300,
  "height": 100,
  "text": "# Heading\nMarkdown content here",
  "color": "4"
}
```

### File Node (links to a vault file)

```json
{
  "id": "unique-id",
  "type": "file",
  "x": 400,
  "y": 0,
  "width": 250,
  "height": 60,
  "file": "wiki/article-slug.md"
}
```

### Link Node (external URL)

```json
{
  "id": "unique-id",
  "type": "link",
  "x": 400,
  "y": 200,
  "width": 250,
  "height": 60,
  "url": "https://example.com"
}
```

### Group Node (visual container)

```json
{
  "id": "unique-id",
  "type": "group",
  "x": -20,
  "y": -20,
  "width": 700,
  "height": 300,
  "label": "Group Label",
  "color": "5"
}
```

## Node Properties

| Property | Type    | Required   | Notes                           |
| -------- | ------- | ---------- | ------------------------------- |
| `id`     | string  | Yes        | Unique identifier               |
| `type`   | string  | Yes        | `text`, `file`, `link`, `group` |
| `x`      | integer | Yes        | Pixel x-coordinate              |
| `y`      | integer | Yes        | Pixel y-coordinate              |
| `width`  | integer | Yes        | Pixel width                     |
| `height` | integer | Yes        | Pixel height                    |
| `text`   | string  | text only  | Markdown content                |
| `file`   | string  | file only  | Vault-relative path             |
| `url`    | string  | link only  | Full URL                        |
| `label`  | string  | group only | Group label text                |
| `color`  | string  | No         | See color system below          |

## Color System

| Preset | Color  |
| ------ | ------ |
| `"1"`  | Red    |
| `"2"`  | Orange |
| `"3"`  | Yellow |
| `"4"`  | Green  |
| `"5"`  | Cyan   |
| `"6"`  | Purple |

Or use hex: `"#FF5733"`

## Edges

```json
{
  "id": "edge-unique-id",
  "fromNode": "source-node-id",
  "toNode": "target-node-id",
  "fromSide": "right",
  "toSide": "left",
  "fromEnd": "none",
  "toEnd": "arrow",
  "color": "1",
  "label": "relates to"
}
```

| Property   | Type   | Required | Notes                            |
| ---------- | ------ | -------- | -------------------------------- |
| `id`       | string | Yes      | Unique identifier                |
| `fromNode` | string | Yes      | Source node ID                   |
| `toNode`   | string | Yes      | Target node ID                   |
| `fromSide` | string | No       | `top`, `right`, `bottom`, `left` |
| `toSide`   | string | No       | `top`, `right`, `bottom`, `left` |
| `fromEnd`  | string | No       | `none` (default) or `arrow`      |
| `toEnd`    | string | No       | `none` or `arrow` (default)      |
| `color`    | string | No       | Same color system as nodes       |
| `label`    | string | No       | Edge label text                  |

## Z-Index

Node array order = ascending z-index. First element renders lowest (behind others).

---

## Knowledge Map Generation

When generating a knowledge map from the wiki:

### 1. Read INDEX.json

Get all topics, articles, and connections.

### 2. Layout Strategy

Use a grid or cluster layout:

```
Topic groups arranged in a grid:
  [Topic A group]    [Topic B group]
  [Topic C group]    [Topic D group]

Within each group:
  - Topic title node (text, colored by topic)
  - Article nodes (file, linking to actual .md files)
  - Edges from title to each article

Between groups:
  - Connection edges for cross-topic articles
```

### 3. Sizing Guidelines

| Element                 | Width                       | Height                      |
| ----------------------- | --------------------------- | --------------------------- |
| Topic title (text node) | 300                         | 80                          |
| Article (file node)     | 250                         | 50                          |
| Group container         | Fit contents + 40px padding | Fit contents + 40px padding |
| Spacing between nodes   | 20px vertical               | 60px between groups         |

### 4. Color Coding

Assign each topic a consistent color preset (1-6). Use the same color for the topic title node, the group, and edges within that topic. Use a neutral color for cross-topic connection edges.
