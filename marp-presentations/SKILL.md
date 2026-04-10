---
name: marp-presentations
description: Create beautiful, professional slide decks from Markdown using Marp (Markdown Presentation Ecosystem). Use when the user wants to create, edit, or modify presentations, slide decks, or speaker notes in Markdown format. Triggers on keywords like "presentation", "slides", "slide deck", "marp", "speaker notes", or when working with .md files that have marp frontmatter.
---

# Marp Presentations

Create beautiful, professional slide decks from Markdown using Marp (Markdown Presentation Ecosystem).

## Quick Start

Marp is a Markdown presentation framework. Create slides using standard Markdown syntax with a few special directives.

```markdown
---
marp: true
title: My Presentation
description: A brief description
size: 16:9
paginate: true
---

# Slide Title

Content goes here

---

## Second Slide

- Bullet points
- More content
```

## Frontmatter Configuration

Every Marp presentation requires at least `marp: true` in the frontmatter.

### Common Frontmatter Options

| Option | Description | Example |
|--------|-------------|---------|
| `marp` | Required. Enables Marp mode | `marp: true` |
| `title` | Presentation title | `title: Git Worktrees` |
| `description` | Short description | `description: Stop stashing...` |
| `size` | Slide aspect ratio | `size: 16:9` or `size: 4:3` |
| `paginate` | Show page numbers | `paginate: true` |
| `theme` | Built-in theme | `theme: default` |
| `style` | Custom CSS | See Custom Styling below |
| `backgroundColor` | Slide background | `backgroundColor: #0d1117` |

## Custom Styling

Use the `style` frontmatter field for custom CSS. See `assets/github-dark-theme.css` for a complete production-ready theme.

### Quick CSS Classes

```markdown
<!-- _class: title-slide -->
# Title Slide

<!-- _class: section-divider -->
# Section Header
```

### Common Custom Classes

| Class | Purpose |
|-------|---------|
| `title-slide` | Centered title with gradient |
| `section-divider` | Section header slide |
| `columns` | Two-column layout |
| `card` | Bordered content box |
| `highlight-box` | Accent-colored callout |
| `danger-box` | Warning/alert callout |
| `success-box` | Positive confirmation |

### CSS Template Pattern

```markdown
style: |
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
  
  :root {
    --bg-dark: #0d1117;
    --accent: #58a6ff;
    --text: #e6edf3;
  }
  
  section {
    background: var(--bg-dark);
    color: var(--text);
    font-family: 'Inter', sans-serif;
  }
```

## Slide Directives

Control individual slide behavior with HTML comments:

```markdown
<!-- _class: title-slide -->
# This slide uses the title-slide class

<!-- _backgroundColor: #000 -->
# Custom background

<!-- _paginate: false -->
# No page number on this slide
```

## Content Patterns

### Two-Column Layout

```markdown
<div class="columns">

<div>

### Left Column

- Point 1
- Point 2

</div>

<div>

### Right Column

- Point A
- Point B

</div>

</div>
```

### Code Blocks

Use triple backticks with language for syntax highlighting:

```markdown
```bash
git status
```
```

### Tables

Standard Markdown tables work:

```markdown
| Feature | Status |
|---------|--------|
| Auth | ✅ Done |
| API | 🚧 WIP |
```

### Blockquotes

```markdown
> Important note or quote
```

## Slide Types

### 1. Title Slide
```markdown
<!-- _class: title-slide -->

<div class="tag">Subtitle or command</div>

# Main Title

## Subtitle

<p class="subtitle">Tagline or description</p>
```

### 2. Content Slide
```markdown
## Slide Title

Regular content with:
- Bullet points
- **Bold text**
- `inline code`
```

### 3. Section Divider
```markdown
<!-- _class: section-divider -->

# 🎯 Section Title
```

## Speaker Notes

Add presenter notes that don't appear on slides:

```markdown
<!--
Speaker notes here.
These won't be visible in the presentation.
-->
```

Or create a separate speaker notes file with the same CSS and `---` delimiters for each slide's notes.

## Resources

### assets/github-dark-theme.css
Complete production-ready theme matching the examples in the workspace. Copy this CSS into your `style:` frontmatter for a consistent, professional dark theme.

## Best Practices

1. **Keep slides focused** - One main idea per slide
2. **Use section dividers** - Break content into logical sections
3. **Consistent styling** - Use the same theme throughout
4. **Code legibility** - Keep code blocks short; split long examples across slides
5. **Visual hierarchy** - Use H1 for titles, H2 for sections, H3 for subsections
6. **Preview locally** - Use Marp VS Code extension or CLI to preview

## Export Formats

Marp can export to:
- **HTML** - Self-contained presentation
- **PDF** - Print-ready document
- **PPTX** - PowerPoint format

Use Marp CLI or VS Code extension to export.
