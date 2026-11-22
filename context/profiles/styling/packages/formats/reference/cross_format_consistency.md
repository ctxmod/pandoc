# Cross-Format Consistency Validation

Validate styling consistency across PDF, DOCX, and HTML output formats.

Procedure for verifying that PDF, DOCX, and HTML output from the same Markdown source produce equivalent presentation. A successful build in each format does not guarantee consistency -- typography may diverge, semantic classes may render in one format but not others, and cross-references may resolve differently. This procedure catches divergence that per-format checks miss.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::validation@core/reference/pdf_quality.md" />

## Prerequisites

- All three formats generated from the same source: `just pdf`, `just docx`, `just html`
- Source contains at least one instance of each semantic class in use (`.highlight`, `.warning`, `.abstract`)
- `reference.docx` and `default.css` configured per their respective procedures

## Comparison Workflow

### 1. Generate All Formats

Build all outputs from the same source to eliminate drift:

```bash
just pdf && just docx && just html
```

Resolve any build failure before proceeding. The build-errors procedure covers diagnostic steps.

### 2. Typography Styles

Open all three outputs side by side. Verify:

- [ ] Heading levels H1-H3 render with distinct sizes in all formats
- [ ] Body text font, size, and line spacing produce comparable density
- [ ] Code blocks use monospace font with consistent indentation
- [ ] Block quotes are visually distinguished from body text

PDF typography derives from LaTeX packages, DOCX from `reference.docx` styles, HTML from `default.css`. When hierarchy diverges, adjust the format-specific template.

### 3. Headers, Footers, and Page Numbers

- [ ] PDF page numbers appear in the configured position
- [ ] DOCX headers and footers from `reference.docx` render on all pages
- [ ] DOCX "Different First Page" omits running heads on the title page (if enabled)
- [ ] HTML navigation elements render via CSS or template structure

### 4. Cross-Reference Resolution

- [ ] Every `@fig:`, `@tbl:`, `@eq:`, `@sec:` reference resolves to a numbered label in all formats
- [ ] PDF hyperlinks navigate to the correct target
- [ ] HTML cross-references render as clickable anchors

### 5. Semantic Class Rendering

Confirm each class produces visible styling in all three outputs:

- [ ] **Highlight**: PDF colorbox, DOCX character style, HTML CSS background
- [ ] **Warning**: PDF tcolorbox, DOCX paragraph style, HTML CSS border
- [ ] **Abstract**: PDF abstract environment, DOCX Abstract style, HTML CSS italic/margin

When a class renders in one format but not another, the Lua filter likely lacks a `FORMAT:match()` branch for that combination. Check `semantic-classes.lua`.

### 6. Background Images

Background images require independent configuration per format:

- [ ] DOCX: Image appears on expected pages with "Behind Text" wrapping and fixed position
- [ ] PDF: LaTeX-specific handling (e.g., `eso-pic` package) matches the DOCX design intent
- [ ] HTML: CSS `background-image` rules in `default.css` produce equivalent placement

### 7. DOCX Section Breaks

- [ ] Next Page breaks start new sections on fresh pages
- [ ] Continuous breaks apply column/margin changes without page breaks
- [ ] Each section carries the correct header/footer configuration
- [ ] Page numbering restarts or continues per section properties

### 8. Google Docs Import

When DOCX files will be shared via Google Docs, upload and open:

1. Upload `output.docx` to Google Drive, right-click > Open with > Google Docs
2. Verify heading hierarchy, standard fonts, and table alignment survive import
3. Expect custom styles to reset to Google defaults and headers/footers to misalign

Google Docs degrades custom styles, non-standard fonts, and complex layouts. Use standard styles and common fonts in `reference.docx` to maximize fidelity. Distribute PDF for collaborators requiring exact formatting.

## Remediation

| Divergence | Fix |
|-----------|-----|
| Font mismatch | Modify style in `reference.docx` or `default.css` to align with PDF |
| Missing semantic style | Add `FORMAT:match()` branch to `semantic-classes.lua` |
| Header/footer absent | Configure in `reference.docx` or add LaTeX package/CSS rule |
| Cross-reference unresolved | Verify pandoc-crossref filter in the build command |
| Section break missing | Add in `reference.docx` or emit raw OpenXML via Lua filter |
| Google Docs import loss | Switch to standard fonts and built-in Word styles |

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/docx_reference.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/lua_filters.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/rules/semantic_classes.md" />

## Related

- `templates/reference.docx` (a project asset)
- `templates/default.css` (a project asset)
- `filters/semantic-classes.lua` (a project asset)
