---
paths:
  - "content/*.md"
  - "content/*/*.md"
  - "content/*/*/*.md"
  - "content/*/*/*/*.md"
  - "papers/*.md"
  - "papers/*/*.md"
  - "papers/*/*/*.md"
  - "papers/*/*/*/*.md"
  - "*.md"
  - "*/*.md"
  - "*/*/*.md"
  - "*/*/*/*.md"
  - "*/*/*/*/*.md"
---

# Semantic Class Annotations

Use semantic class annotations (`{.highlight}`, `{.warning}`, `{.abstract}`, `{.caption}`) in Markdown source instead of direct formatting, and include the filter in every build recipe that processes Markdown containing semantic classes via `--lua-filter=filters/semantic-classes.lua`. For every semantic class in use, define the corresponding style in `reference.docx` (DOCX), `default.css` (HTML), and the LaTeX handlers (PDF). Direct formatting couples content to a single output format — bold text in all formats loses the DOCX Warning paragraph style, the HTML `.warning` class for CSS targeting, and the styled PDF warning box — and omitting the filter leaves class annotations passing through as raw attributes, producing visible markup artifacts in output documents.

Inline annotations are `[text]{.class}`; block annotations are `::: {.class}` fenced divs. The standard vocabulary is `highlight` (emphasized inline content), `warning` (cautionary block), `abstract` (paper abstract), and `caption` (figure or table caption annotation). Extend it per project: each new class requires a Word style in `reference.docx` matching the class name (capitalized), a CSS rule in `default.css` targeting `.classname`, and a LaTeX handler in `semantic-classes.lua` for the corresponding `FORMAT:match('latex')` branch.

Plain text output (`-t plain`, `-t markdown`) does not require the semantic-classes filter, since these formats strip or preserve class annotations without visual rendering. Draft previews using `pandoc -t html` without `--css` may also omit the filter when rapid iteration matters more than styled output.

- the class-to-style mapping per output format, the Lua filter mechanics, and worked correct/incorrect examples — <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/semantic_classes.md" />
- developing, testing, and debugging the filter itself — <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/lua_filters.md" />

Related:

- `filters/semantic-classes.lua` (a project asset)
