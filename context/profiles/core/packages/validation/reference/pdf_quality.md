# PDF Quality Validation

Post-build validation workflow for PDF output covering font embedding, image resolution, hyperlinks, cross-references, and bibliography formatting.

Procedure for verifying PDF output quality after a successful build. A clean build does not guarantee a publishable document -- fonts may be missing, images may render at low resolution, hyperlinks may be broken, and cross-references may display as raw labels. This procedure catches quality issues that compilation alone cannot detect.

## Prerequisites

- A successfully generated PDF from `just pdf` or `just docx`
- `pdffonts` and `pdfinfo` available (part of poppler-utils)
- The source project follows the structure defined in the build and float rules

## Automated Checks

### Font Embedding

Unembedded fonts cause rendering failures on systems that lack the font. Verify all fonts are embedded:

```bash
pdffonts output.pdf
```

Every row in the output must show `yes` in the `emb` column. pdfLaTeX, LuaLaTeX, and XeLaTeX embed the fonts they use by default, so an unembedded font usually enters through an included figure PDF; regenerate that figure with fonts embedded. Type 3 bitmap fonts indicate a configuration problem -- these produce poor scaling. Under pdfLaTeX they appear when T1 encoding is used without an outline font: load `lmodern` (pandoc's default LaTeX template loads it for pdfLaTeX) or another outline font package:

```latex
\usepackage[T1]{fontenc}
\usepackage{lmodern}
```

Under LuaLaTeX or XeLaTeX, set OpenType fonts through the `mainfont`, `sansfont`, and `monofont` metadata variables. `-dEmbedAllFonts` is a Ghostscript option and has no effect on a LaTeX engine.

### Document Metadata

Confirm the PDF carries correct metadata using:

```bash
pdfinfo output.pdf
```

Verify `Title`, `Author`, and `CreationDate` fields match the document's YAML frontmatter. Missing metadata indicates the YAML block is not propagating to the PDF engine. Check that `title`, `author`, and `date` fields are present in the root document's frontmatter.

### Image Resolution

Low-resolution figures appear blurred in print. Raster images (PNG, JPG) should meet a minimum of 300 DPI for print and 150 DPI for screen-only distribution. Verify image dimensions against their rendered size in the PDF:

```bash
pdfimages -list output.pdf
```

Each image entry reports pixel dimensions and rendered size. Divide pixel width by rendered width in inches to compute effective DPI. Images below the target threshold need regeneration at higher resolution from the source data.

## Visual Inspection Checklist

Automated tools cannot catch layout and formatting problems. Open the PDF and verify each item:

- [ ] **Cross-references**: All `@fig:`, `@tbl:`, `@eq:`, and `@sec:` labels resolve to numbered references, not raw label text
- [ ] **Hyperlinks**: Internal cross-reference links navigate to the correct figure, table, or section when clicked
- [ ] **Bibliography**: All citations render in the specified CSL style with consistent formatting
- [ ] **Missing citations**: No citation key followed by `?` (citeproc's marker for a key absent from the bibliography, reported as `[WARNING] Citeproc: citation <key> not found`) and no bold `¿fig:label?` cross-reference markers appear in the text
- [ ] **Figure placement**: Figures appear near their first reference, captions are present and numbered
- [ ] **Table formatting**: Booktabs rules render correctly, no vertical lines, column alignment is consistent
- [ ] **Margins**: Content respects the document class margin requirements with no overflow into gutters
- [ ] **Orphans and widows**: No single lines stranded at the top or bottom of a page
- [ ] **Page numbers**: Sequential, starting from the correct page, matching the header/footer style

## Accessible PDF

A PDF is accessible only when it is tagged: tags carry the reading order, headings, lists, table structure, and image alternative text that a screen reader needs. Since pandoc 3.9, the `pdfstandard` variable requests a standard through LaTeX's `\DocumentMetadata` interface, adding tagging where the standard requires it:

```yaml
pdfstandard:
  - ua-2
  - a-4f
```

This needs LuaLaTeX from TeX Live 2025 with a LaTeX kernel of 2025-06-01 or newer (`--pdf-engine=lualatex`). Give every figure alternative text that differs from its caption where the caption does not describe the image: `![Caption.](figures/fig-x.png){#fig:x alt="What the image shows"}`. Pandoc cannot verify conformance, so check the result with a PDF/UA validator such as veraPDF before claiming it.

## DOCX Comparison

When the project produces both PDF and DOCX output (via `just pdf` and `just docx`), verify structural correspondence between formats:

1. Confirm section headings match in count and hierarchy
2. Verify all figures appear in both outputs with captions
3. Check that table content and row counts match
4. Confirm bibliography entries are identical in both formats
5. Note any cross-reference differences (pandoc-crossref resolves differently for DOCX)

Structural divergence between PDF and DOCX indicates format-specific LaTeX constructs that pandoc cannot convert. Replace these with Pandoc Markdown equivalents per the pandoc integration principle.

## Troubleshooting

**Low-resolution figures**: Regenerate from source data at 300+ DPI. If the source is vector-based, export as PNG at the required resolution rather than relying on rasterization during PDF compilation.

**Unembedded fonts**: Switch to a font package that supports embedding (e.g., `lmodern`, `libertinus`). Verify with `pdffonts` after rebuilding.

**Broken cross-references**: Rebuild twice to resolve forward references. If labels still appear as raw text, verify pandoc-crossref filter is in the build pipeline and label syntax matches `{#fig:label}` format exactly.

**Missing bibliography entries**: Verify the cited key exists in `bibliography/refs.bib`, including case, and read the build log for citeproc's `citation <key> not found` warning. citeproc resolves citations on every pandoc run, so stale auxiliary files matter only for raw BibTeX or biblatex builds; for those, run `just clean` followed by a fresh build.

the academic quality-assurance validation procedure (agentic module, `workflows` profile, `validation` package)

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::references@core/rules/floats.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />

## Related

- the academic quality-assurance validation procedure (agentic module, `workflows` profile, `validation` package)
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::validation@core/reference/build_errors.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />

## Sources

- https://pandoc.org/MANUAL.html — sections "Accessible PDFs and PDF archiving standards" (`pdfstandard`, LuaLaTeX and kernel requirements), `implicit_figures` (the `alt` attribute), and the default LaTeX font setup (pandoc 3.14, 2026-11-10); accessed 2026-11-23.
- https://github.com/jgm/pandoc/blob/main/changelog.md — pandoc 3.9 (2026-02-03) added the `pdfstandard` variable; accessed 2026-11-23.
- https://github.com/jgm/citeproc — the `citation <key> not found` warning and the `key?` placeholder citeproc renders for a missing entry; accessed 2026-11-23.
