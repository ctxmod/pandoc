---
description: >-
  Tri-modal styling for pandoc output: semantic classes, template
  variables, the DOCX reference document, Lua filters, and
  cross-format consistency across PDF, DOCX and HTML.
binds_to_depth: main...
---

# Pandoc Styling

Adds tri-modal styling infrastructure to the core pandoc profile. Projects applying this profile gain format-specific templates, a Lua filter pipeline, and cross-format consistency procedures that work alongside the core profile's build automation, bibliography management, and modular content structure.

## Tri-Modal Architecture

Three output formats share a single Markdown source. Each format receives styling through a dedicated mechanism.

| Format | Styling Mechanism | Template Asset |
|--------|-------------------|----------------|
| PDF | LaTeX templates with pandoc variables | Default pandoc LaTeX template, customized via YAML metadata |
| DOCX | Word reference document defining named styles | `templates/reference.docx` |
| HTML | CSS stylesheet with semantic class mappings | `templates/default.css` |

Lua filters bridge these mechanisms. A filter like `semantic-classes.lua` reads format-agnostic class annotations in Markdown source (e.g., `[text]{.highlight}`) and emits format-appropriate styling instructions: `\colorbox` for LaTeX, custom style attributes for DOCX, inline CSS for HTML. This keeps the source format-neutral while producing styled output across all three targets.

The template and filter files named here (`templates/reference.docx`, `templates/default.css`, `filters/semantic-classes.lua`) are assets of the consuming project, not of this module.

## Core Profile Principles

This profile inherits and extends the core pandoc profile's principles.

- **Pandoc integration** (<ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />): All styling works through pandoc's native extension points -- templates, reference documents, CSS, and Lua filters -- rather than post-processing or external tools.
- **Modularity** (<ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/modularity.md" />): Templates, filters, and procedures are independent units. A project can use `reference.docx` for DOCX styling without adopting `default.css` for HTML.
- **Justfile builds** (<ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />): Styling assets integrate into existing build recipes via flags (`--reference-doc`, `--css`, `--lua-filter`).

## Capabilities

### Templates

- DOCX reference document (`templates/reference.docx`) defining heading, body, abstract, caption, and table styles
- CSS stylesheet (`templates/default.css`) for HTML output with academic typography and semantic class support
- PDF styling through pandoc's LaTeX template system and YAML metadata variables

### Filters

- Semantic class filter (`filters/semantic-classes.lua`) mapping format-agnostic annotations to format-specific styling
- Filter pipeline integration through `--lua-filter` build flags

### Procedures

- DOCX reference document creation and customization workflow (<ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/docx_reference.md" />)
- HTML stylesheet development and resource embedding
- Lua filter development, testing, and organization (<ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/lua_filters.md" />)
- Cross-format visual consistency validation (<ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/cross_format_consistency.md" />)
