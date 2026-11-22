# Semantic Classes

Map semantic div/span classes to format-specific styling through Lua filters.

Lua filters translate semantic class annotations in Markdown source into format-specific styling for PDF, DOCX, and HTML output. Authors mark content with meaning (`.highlight`, `.abstract`, `.warning`) rather than appearance. Filters handle the translation to each output format during pandoc conversion, keeping source documents portable and presentation-independent.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />

## Lua Filter Fundamentals

A pandoc Lua filter is a Lua script containing functions named after AST element types. Pandoc calls each function when it encounters a matching element during conversion. The function receives the element, optionally transforms it, and returns either the modified element, `nil` (no change), an empty list (deletion), or a list of elements (splice).

Two element types carry classes from Markdown source:

| Element | Markdown Syntax | AST Type |
|---------|----------------|----------|
| Block-level | `::: {.warning}\ncontent\n:::` | `Div` |
| Inline | `[text]{.highlight}` | `Span` |

The `FORMAT` global variable identifies the target output format. Filters use `FORMAT:match('docx')`, `FORMAT:match('html')`, or `FORMAT:match('latex')` to branch into format-specific logic. This conditional structure enables a single filter file to serve all three output formats.

## Class-to-Style Mapping

The core pattern reads the first class from a Div or Span, then applies format-appropriate styling.

**DOCX**: Pandoc supports a `custom-style` attribute that maps directly to Word styles defined in the reference document. Setting `el.attributes['custom-style'] = 'StyleName'` applies the named paragraph or character style from `reference.docx`.

**HTML**: Classes pass through to the output element natively. A Span with class `.highlight` produces `<span class="highlight">` in HTML, styled by the accompanying CSS stylesheet. No filter transformation needed for HTML unless the class requires attribute injection.

**LaTeX/PDF**: No native class mechanism exists. Filters emit raw LaTeX commands via `pandoc.RawInline('latex', ...)` or `pandoc.RawBlock('latex', ...)` to achieve the desired visual effect.

A Div handler for the `.warning` class illustrates the pattern:

```lua
function Div(el)
  if el.classes:includes('warning') then
    if FORMAT:match('docx') then
      el.attributes['custom-style'] = 'Warning'
      return el
    elseif FORMAT:match('latex') then
      return pandoc.RawBlock('latex',
        '\\begin{tcolorbox}[colback=orange!10, colframe=orange!80]\n' ..
        pandoc.utils.stringify(el.content) ..
        '\n\\end{tcolorbox}')
    end
    -- HTML: class passes through, CSS handles styling
  end
end
```

## Filter Structure

Organize filters as a returned table of element handlers. Pandoc 3.x supports both the implicit global-function style and the explicit table return. The table return provides ordering control when multiple element types interact:

```lua
return {
  { Span = handle_inline_classes },
  { Div = handle_block_classes },
  { Meta = inject_metadata }
}
```

Pandoc executes filter passes in list order. Place metadata injection (`Meta`) after element transformations when computed values depend on document content. Place it before when elements need metadata for conditional logic.

## Format-Conditional Guards

Wrap format-specific logic in guard clauses at the function level rather than nesting conditionals deeply:

```lua
function Span(el)
  local class = el.classes[1]
  if not class then return nil end

  if FORMAT:match('docx') then
    return map_docx_span(el, class)
  elseif FORMAT:match('latex') then
    return map_latex_span(el, class)
  end
  -- HTML and other formats: no transformation needed
end
```

This structure keeps each format's logic isolated and testable. When a format requires no transformation (HTML classes pass through natively), the function returns `nil` implicitly.

## Standard Semantic Classes

Define a consistent vocabulary of semantic classes that all three format pipelines recognize:

| Class | Purpose | DOCX Style | LaTeX Command | CSS Rule |
|-------|---------|------------|---------------|----------|
| `highlight` | Emphasized content | Highlight (character) | `\colorbox{yellow}{...}` | `background-color: yellow` |
| `warning` | Cautionary blocks | Warning (paragraph) | `tcolorbox` environment | `border-left: 4px solid orange` |
| `abstract` | Paper abstract | Abstract (paragraph) | `abstract` environment | `font-style: italic; margin: 2em 0` |
| `caption` | Figure/table captions | Caption (paragraph) | `\caption{...}` | `font-size: 0.9em; color: #555` |

Extend this vocabulary per project. New classes require three additions: a DOCX style in `reference.docx`, a LaTeX handler in the Lua filter, and a CSS rule in `default.css`.

## Debugging and Testing

Test filters against sample Markdown input with explicit format targets:

```bash
pandoc sample.md --lua-filter=filters/semantic-classes.lua -t docx -o test.docx
pandoc sample.md --lua-filter=filters/semantic-classes.lua -t html -o test.html
pandoc sample.md --lua-filter=filters/semantic-classes.lua -t latex -o test.tex
```

Insert `print(pandoc.utils.stringify(el))` or `io.stderr:write(el.t .. '\n')` inside filter functions to trace element processing. The `pandoc.utils.type(el)` function confirms element types when debugging unexpected behavior. Remove debug statements before committing filters to version control.

## Rule Companion: Standard Vocabulary, Build Pipeline Requirement, and Examples

*Companion to the `semantic_classes` rule: its full rule set, worked examples and rationale.*

### Rationale

Direct formatting couples content to a single output format. A manually bolded warning block renders correctly in HTML but carries no semantic meaning for DOCX or LaTeX pipelines. Semantic class annotations separate meaning from presentation, enabling a single Lua filter to translate intent into format-appropriate styling during pandoc conversion. This maintains source portability across the tri-modal output pipeline (PDF, DOCX, HTML) and keeps formatting decisions centralized in templates and filters rather than scattered through content files.

### Standard Vocabulary

All projects using this profile recognize these semantic classes:

| Class | Markdown Syntax | Purpose |
|-------|----------------|---------|
| `highlight` | `[text]{.highlight}` | Emphasized inline content |
| `warning` | `::: {.warning}\ncontent\n:::` | Cautionary block |
| `abstract` | `::: {.abstract}\ncontent\n:::` | Paper abstract |
| `caption` | `[text]{.caption}` | Figure or table caption annotation |

Extend this vocabulary per project. Each new class requires three additions:

1. A Word style in `reference.docx` matching the class name (capitalized)
2. A CSS rule in `default.css` targeting `.classname`
3. A LaTeX handler in `semantic-classes.lua` for the corresponding `FORMAT:match('latex')` branch

### Build Pipeline Requirement

Every build recipe that processes Markdown containing semantic classes must include the Lua filter flag:

```bash
pandoc input.md --lua-filter=filters/semantic-classes.lua -o output.FORMAT
```

Omitting the filter causes class annotations to pass through as raw attributes, producing visible markup artifacts in output documents.

### Examples

**Correct**: Semantic annotation with filter in build
```markdown
The results [exceeded expectations]{.highlight} across all trials.

::: {.warning}
Sample sizes below 30 may produce unreliable estimates.
:::
```

Build command:
```bash
pandoc paper.md --lua-filter=filters/semantic-classes.lua --reference-doc=templates/reference.docx -o paper.docx
```

**Incorrect**: Direct formatting instead of semantic class
```markdown
The results **exceeded expectations** across all trials.

> **Warning:** Sample sizes below 30 may produce unreliable estimates.
```

This approach produces bold text in all formats but loses semantic meaning. DOCX output lacks the Warning paragraph style. HTML output lacks the `.warning` class for CSS targeting. PDF output renders a generic blockquote instead of a styled warning box.

**Incorrect**: Semantic annotation without filter in build
```bash
pandoc paper.md --reference-doc=templates/reference.docx -o paper.docx
```

Output contains literal `{.highlight}` text or unstyled spans because no filter translates the annotations.

### Exceptions

Plain text output (`-t plain`, `-t markdown`) does not require the semantic-classes filter, since these formats strip or preserve class annotations without visual rendering. Draft previews using `pandoc -t html` without `--css` may also omit the filter when rapid iteration matters more than styled output.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/rules/semantic_classes.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/lua_filters.md" />

## Related

- `filters/semantic-classes.lua` (a project asset)
