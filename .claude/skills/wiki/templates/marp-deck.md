# Marp Slide Deck Reference

Use this when generating slide deck outputs. Marp converts markdown to presentations, viewable in Obsidian with the Marp Slides plugin.

---

## Basic Structure

```markdown
---
marp: true
theme: default
paginate: true
---

# Slide 1 Title

Content for the first slide.

---

# Slide 2 Title

- Bullet point 1
- Bullet point 2
- Bullet point 3

---

## Subsection Slide

More content here.
```

**Key rule:** Slides are separated by `---` (horizontal rule). The first `---` after frontmatter starts slide 1.

---

## Frontmatter Options

```yaml
---
marp: true # Required. Enables Marp rendering.
theme: default # default | gaia | uncover
paginate: true # Show page numbers
header: "Header Text" # Top of every slide
footer: "Footer Text" # Bottom of every slide
backgroundColor: "#fff" # Slide background color
color: "#333" # Text color
---
```

## Themes

| Theme     | Style                                 |
| --------- | ------------------------------------- |
| `default` | Clean, minimal, white background      |
| `gaia`    | Bold, centered, good for title slides |
| `uncover` | Modern, sidebar style                 |

## Images

```markdown
<!-- Standard -->

![](image.png)

<!-- Sized -->

![w:500](image.png)
![h:300](image.png)

<!-- Background image (fills slide) -->

![bg](image.png)

<!-- Split: image on right half -->

![bg right](image.png)

<!-- Split with ratio -->

![bg right:40%](image.png)
```

## Speaker Notes

```markdown
# Slide Title

Visible content.

<!-- This is a speaker note. Not shown on slide. -->
```

## Math (KaTeX)

```markdown
Inline: $E = mc^2$

Block:

$$
\sum_{i=1}^{n} x_i = x_1 + x_2 + \cdots + x_n
$$
```

---

## Deck Generation Guidelines

When generating a Marp deck from wiki content:

1. **Title slide:** Topic name + one-line description + date
2. **Overview slide:** Agenda or key topics (3-5 items)
3. **Content slides:** One concept per slide. Use the article's "Core Idea" section.
4. **Data slides:** Pull specific metrics and data points from summaries. Use tables or lists.
5. **Connections slide:** Show how concepts relate (use text diagrams or bullet relationships)
6. **Summary slide:** Key takeaways (3-5 bullets)
7. **Sources slide:** List source summaries referenced

Keep slides concise. 5-7 bullets max per slide. Use headers and subheaders for structure. Save to `Vault/output/[topic]-deck.md`.
