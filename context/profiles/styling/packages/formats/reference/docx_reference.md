# DOCX Reference Template Creation

Create and customize a Word reference.docx template for pandoc DOCX output styling.

Procedure for generating, customizing, and testing a Word reference.docx that controls pandoc DOCX output styling. Pandoc applies styles from the reference document to generated output, so all visual formatting lives in the template rather than in CLI flags or Markdown source.

## Prerequisites

- pandoc installed (2.11+ for bundled citeproc)
- Microsoft Word or compatible editor (LibreOffice Writer works for basic modifications)
- Paper directory structure created per project setup procedure

## Steps

### 1. Generate the Baseline Template

Extract pandoc's default reference document to establish the complete set of expected style names:

```bash
pandoc -o reference.docx --print-default-data-file reference.docx
```

This baseline contains sample text demonstrating every style pandoc maps during DOCX generation. Always start from this file rather than creating a blank document. A blank document lacks pandoc's expected style names, which causes output to fall back to Word defaults.

### 2. Modify Built-In Styles

Open `reference.docx` in Word and modify styles through the Styles pane (Home tab > Styles section). Right-click a style, select "Modify", and adjust properties.

**Priority styles to configure**:

| Style | Typical Adjustments |
|-------|-------------------|
| Normal | Font family, size (11-12pt), line spacing (1.15-1.5), paragraph spacing |
| Title | Font family, size (24-28pt), alignment, spacing after |
| Heading 1 | Font, size (18-22pt), bold, spacing before/after, page break before |
| Heading 2 | Font, size (14-16pt), bold, spacing before/after |
| Heading 3-6 | Progressive size reduction, consistent font family |
| First Paragraph | Match Normal but without first-line indent |
| Body Text | Match Normal with first-line indent if desired |
| Code Block | Monospace font (Consolas, Courier New), reduced size, background shading |
| Block Text / Quote | Indented margins, italic or reduced size |

Save modifications to the reference.docx file itself. Changes apply only to this template and do not affect global Word styles.

### 3. Define Custom Styles

Create project-specific styles for content that pandoc does not map by default. In Word: Home > Styles > New Style. Name each style descriptively ("Abstract", "Keywords", "Caption").

Apply custom styles in Markdown source using the `custom-style` attribute:

```markdown
::: {custom-style="Abstract"}
This paragraph renders with the Abstract style in DOCX output.
:::

Text with [emphasized]{custom-style="Emphasized Text"} inline styling.
```

### 4. Configure Headers and Footers

Double-click the header or footer area in Word to enter editing mode. Pandoc does not generate headers/footers programmatically; they must be baked into the reference template.

**Dynamic content via Word fields**:
- `{ PAGE }` -- current page number
- `{ STYLEREF "Heading 1" }` -- running head from the current chapter title
- `{ TITLE }` -- document title (populated from pandoc YAML metadata)
- `{ AUTHOR }` -- author name (populated from pandoc YAML metadata)

**Different first page**: Enable "Different First Page" in Header & Footer Tools (Design tab). This creates independent first-page headers and footers, useful for title pages that omit running heads or page numbers.

**Odd/even pages**: Enable "Different Odd & Even Pages" for book-style layouts with alternating headers.

### 5. Add Header Background Images

To place a background image (logo, watermark) in the header:

1. Enter header editing mode (double-click header area)
2. Insert > Pictures > select the image file
3. Right-click the image > Wrap Text > Behind Text
4. Format tab > Position > Fix Position on Page (prevents the image from shifting with text)
5. Resize and position to cover the desired area
6. Exit header mode

Images placed in the header repeat on every page by default. When "Different First Page" is enabled, add a separate image (or none) to the first-page header. This supports designs where a full-bleed cover image appears only on page one while subsequent pages carry a smaller logo.

### 6. Set Up Section Breaks

Section breaks enable different page layouts, headers, and footers within the same document. In the reference template, insert section breaks where format transitions occur:

- **Next Page**: Page Layout > Breaks > Next Page (new section on a new page)
- **Continuous**: Page Layout > Breaks > Continuous (new section without a page break)

Pandoc's `--top-level-division=chapter` flag inserts section breaks before first-level headings. For finer control, use horizontal rules (`---`) in Markdown combined with a Lua filter that converts them to DOCX section breaks.

### 7. Test with Pandoc

Build a sample document using the customized reference template:

```bash
pandoc sample.md -o output.docx --reference-doc=reference.docx
```

Verify the following in the generated output:
- Heading styles render with correct font, size, and spacing
- Normal text uses the configured paragraph formatting
- Custom styles apply via `custom-style` attributes
- Headers and footers display correctly on first and subsequent pages
- Background images appear in the expected position with Behind Text wrapping
- Section breaks produce the intended page layout transitions
- Word fields (`PAGE`, `TITLE`, `AUTHOR`) populate from document metadata

If styles do not apply, confirm the style names in `reference.docx` match pandoc's expected names exactly. Renamed or deleted styles revert to Word defaults.

## Version Control

DOCX files are binary ZIP archives containing XML, which limits Git diff and merge capabilities. Two approaches maintain traceability:

**Git textconv** (recommended): Configure `.gitattributes` so `git diff` renders DOCX changes as readable Markdown:
```gitattributes
*.docx diff=pandoc
```
```gitconfig
[diff "pandoc"]
  textconv = pandoc --to=markdown
```

**Commit discipline**: Commit `reference.docx` alongside a brief commit message describing which styles changed. Avoid bundling template changes with content changes in the same commit.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/rules/semantic_classes.md" />

## Related

- `templates/reference.docx` (a project asset)
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />
