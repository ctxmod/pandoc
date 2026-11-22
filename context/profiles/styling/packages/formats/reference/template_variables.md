# Template Variables

Standard YAML template variables for pandoc documents across metadata, layout, styling, and conditional-feature categories.

*Companion to the `template_variables` rule. The rule retains the normative core; this file carries the variable tables and worked examples.*

Define all template-configurable values as YAML frontmatter variables. Prohibit hardcoding values in document content or build commands when those values control layout, styling, or conditional document features.

## Rationale

Pandoc templates interpolate variables from YAML metadata using `$var$` syntax. When values like title, author, or layout flags live in frontmatter, a single metadata block governs the entire document across all output formats. Hardcoding these values in content or build flags scatters configuration, making documents harder to maintain and inconsistent across PDF, DOCX, and HTML pipelines. Centralizing variables in YAML also enables defaults files to provide project-wide settings that individual documents override selectively.

## Standard Variables

Every document using this profile should declare applicable variables from the following categories.

### Document Metadata

| Variable | Type | Purpose |
|----------|------|---------|
| `title` | string | Document title rendered in all formats |
| `author` | string or list | Author name(s) for title block |
| `date` | string | Publication or revision date |
| `abstract` | string | Abstract text for title page or header |
| `keywords` | list | Subject keywords for metadata embedding |

### Layout Control

| Variable | Type | Purpose |
|----------|------|---------|
| `frontpage` | boolean or string | Enable custom front page; path to background image if string |
| `secondpage` | boolean or string | Enable distinct second page layout |
| `toc` | boolean | Generate table of contents |
| `toc-depth` | integer | Heading depth for table of contents |
| `numbersections` | boolean | Enable section numbering |

### Styling Configuration

| Variable | Type | Purpose |
|----------|------|---------|
| `fontsize` | string | Base font size (e.g., `11pt`, `12pt`) |
| `geometry` | string or list | Page margins and dimensions |
| `linestretch` | number | Line spacing multiplier |
| `mainfont` | string | Primary document font (requires xelatex/lualatex) |
| `sansfont` | string | Sans-serif font for headings |
| `monofont` | string | Monospace font for code blocks |

### Conditional Features

| Variable | Type | Purpose |
|----------|------|---------|
| `fancyhdr-enabled` | boolean | Activate custom headers and footers via fancyhdr |
| `wallpaper-path` | string | Background image path for eso-pic integration |
| `colorlinks` | boolean | Enable colored hyperlinks |
| `linkcolor` | string | Color for internal links |
| `citecolor` | string | Color for citation links |

## Examples

**Correct**: All configurable values in YAML frontmatter
```yaml
---
title: "Analysis of Survey Results"
author:
  - "First Author"
  - "Second Author"
date: "2026-02-23"
abstract: "This paper examines..."
frontpage: true
secondpage: false
toc: true
toc-depth: 3
fontsize: 11pt
geometry: "margin=2.5cm"
fancyhdr-enabled: true
---
```

**Incorrect**: Title hardcoded in content instead of metadata
```markdown
---
author: "First Author"
---

# Analysis of Survey Results

\thispagestyle{fancy}
\setlength{\parindent}{0pt}
```

This approach embeds the title as a heading rather than a metadata variable, preventing pandoc templates from placing it in title blocks, headers, or PDF properties. The `\thispagestyle` call hardcodes a layout decision that the `fancyhdr-enabled` variable should control.

**Incorrect**: Build flags substituting for frontmatter variables
```bash
pandoc paper.md -V fontsize=12pt -V geometry="margin=1in" -o paper.pdf
```

Command-line variables override frontmatter silently, creating inconsistency between what the document declares and what the build produces. Declare these values in YAML so the document remains self-describing.

## Exceptions

Build-time overrides via `-V` or `--metadata` are acceptable for CI/CD pipelines that need to inject values not known at authoring time (e.g., build numbers, commit hashes, generated timestamps). Document these overrides in the project Justfile so they remain discoverable.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/rules/template_variables.md" />

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />
