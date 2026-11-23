# Lua Filter Development

Develop, test, and organize pandoc Lua filters for format-agnostic semantic styling.

Procedure for writing pandoc Lua filters that transform semantic class annotations into format-specific styling for PDF, DOCX, and HTML output. Filters operate on the pandoc AST during conversion, enabling transformations that templates and reference documents cannot express.

See also: [Pandoc Lua Filters documentation](https://pandoc.org/lua-filters.html)

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/semantic_classes.md" />

## Prerequisites

- pandoc 3.x installed (Lua filters exist since 2.0; `traverse` and `pandoc.utils.type` since 2.17; the `Figure` element since 3.0). A filter that depends on a version states it with `PANDOC_VERSION:must_be_at_least '3.0'`
- Text editor with Lua syntax highlighting
- Sample Markdown file containing semantic class annotations (`{.highlight}`, `{.warning}`)

## Steps

### 1. Define the Filter Structure

Create a new `.lua` file in the `filters/` directory. Name it after its purpose: `semantic-classes.lua`, `metadata-inject.lua`, `section-breaks.lua`.

A filter is a Lua table mapping AST element type names to handler functions. Return the table explicitly for ordering control:

```lua
return {
  { Span = handle_inline_classes },
  { Div = handle_block_classes },
  { Meta = inject_metadata }
}
```

Pandoc executes passes in list order. Place `Meta` handlers before element handlers when elements depend on metadata values. Place them after when metadata depends on document content.

### 2. Write Element Handler Functions

Each handler receives a single AST element and returns one of four values: the modified element (replace), `nil` (no change), an empty list (delete), or a list of elements (splice).

Two element types carry classes from Markdown source:

| Markdown Syntax | AST Type | Handler Function |
|----------------|----------|-----------------|
| `::: {.warning}\ncontent\n:::` | `Div` | `function Div(el)` |
| `[text]{.highlight}` | `Span` | `function Span(el)` |

Additional handlers (`RawBlock`, `CodeBlock`, `Meta`, `Header`, `Figure`) serve specialized transformations such as raw format injection, code annotation, and metadata computation.

Since pandoc 3.0, a captioned image on its own line is a `Figure` block wrapping the image, not a paragraph holding one image. A filter written for pandoc 2 that matches `Para` with a single `Image` no longer sees figures; give it a `Figure` handler, and reach the caption through `el.caption` and the image through `el.content`.

### 3. Add Format-Conditional Logic

The `FORMAT` global variable identifies the target output format. Use `FORMAT:match()` to branch into format-specific paths:

```lua
local function handle_inline_classes(el)
  local class = el.classes[1]
  if not class then return nil end

  if FORMAT:match('docx') then
    el.attributes['custom-style'] = class_to_docx_style(class)
    return el
  elseif FORMAT:match('latex') then
    return emit_latex_inline(el, class)
  end
  -- HTML: classes pass through natively, styled by CSS
end
```

Guard on format at the function level rather than nesting conditionals deeply. `FORMAT:match('latex')` does not match `beamer`, so a filter that should also style slides tests both. Typst output (`FORMAT == 'typst'`) needs its own branch emitting `pandoc.RawInline('typst', ...)` when a project builds Typst drafts. When a format needs no transformation (HTML classes map directly to CSS selectors), return `nil` implicitly.

### 4. Map Classes to Format-Specific Output

**DOCX**: Set the `custom-style` attribute to a Word style name defined in `reference.docx`:

```lua
el.attributes['custom-style'] = 'Warning'
```

**LaTeX/PDF**: Emit raw LaTeX commands via `pandoc.RawInline` or `pandoc.RawBlock`:

```lua
return pandoc.RawBlock('latex',
  '\\begin{tcolorbox}[colback=orange!10]\n' ..
  pandoc.utils.stringify(el.content) ..
  '\n\\end{tcolorbox}')
```

**OpenXML (advanced DOCX)**: Emit raw OpenXML for effects beyond Word styles:

```lua
return pandoc.RawBlock('openxml',
  '<w:p><w:pPr><w:sectPr>' ..
  '<w:type w:val="nextPage"/>' ..
  '</w:sectPr></w:pPr></w:p>')
```

### 5. Test the Filter

Run pandoc with `--lua-filter` against each target format using a sample Markdown file:

```bash
pandoc sample.md --lua-filter=filters/semantic-classes.lua -t docx -o test.docx
pandoc sample.md --lua-filter=filters/semantic-classes.lua -t html -o test.html
pandoc sample.md --lua-filter=filters/semantic-classes.lua -t latex -o test.tex
```

Verify that semantic classes produce the expected visual output in each format. For DOCX, confirm custom styles appear in the Styles pane. For HTML, inspect generated class attributes. For LaTeX, check that raw commands compile without errors.

### 6. Debug Filter Behavior

Insert diagnostic output to trace element processing:

```lua
io.stderr:write('Processing: ' .. pandoc.utils.type(el) .. ' ' ..
  pandoc.utils.stringify(el) .. '\n')
```

Use `pandoc.utils.type(el)` to confirm element types when handlers receive unexpected input. Use `pandoc.utils.stringify(el)` to inspect text content. Remove debug statements before committing.

### 7. Organize the Filters Directory

Place filters in `filters/` within the project root, named by responsibility:

| Filter | Purpose |
|--------|---------|
| `semantic-classes.lua` | Map semantic annotations to format styles |
| `metadata-inject.lua` | Compute and inject template variables |
| `section-breaks.lua` | Convert horizontal rules to section breaks |

Apply multiple filters in sequence via repeated `--lua-filter` flags. Execution order follows flag order on the command line. Filters with dependencies (metadata injection before class mapping) must appear in the correct sequence.

## Troubleshooting

| Symptom | Cause | Resolution |
|---------|-------|------------|
| Filter has no effect | Handler function name does not match AST type | Verify function name matches exactly (`Div`, `Span`, `Meta`) |
| Custom style missing in DOCX | Style not defined in reference.docx | Add the named style to the reference template |
| LaTeX compilation error | Malformed raw LaTeX string | Test raw output with `pandoc -t latex` before PDF build |
| "module 'pandoc' not found" | Running Lua outside pandoc | Filters execute only within pandoc, not standalone |
| Handler called for wrong elements | Missing class guard | Add `if not el.classes[1] then return nil end` |

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />

## Related

- `filters/semantic-classes.lua` (a project asset)

## Sources

- https://pandoc.org/lua-filters.html — traversal order (`traverse`, since 2.17), the `Figure` element and constructor, `PANDOC_VERSION:must_be_at_least`, `pandoc.utils.type` (pandoc 3.14); accessed 2026-11-23.
- https://github.com/jgm/pandoc/blob/main/changelog.md — pandoc 3.0 (2023-01-18) adds the `Figure` block through pandoc-types 1.23; accessed 2026-11-23.
