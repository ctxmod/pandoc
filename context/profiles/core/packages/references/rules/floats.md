---
paths:
  - "content/papers/*.tex"
  - "content/papers/*/*.tex"
  - "content/papers/*/*/*.tex"
  - "content/papers/*.md"
  - "content/papers/*/*.md"
  - "content/papers/*/*/*.md"
  - "content/papers/*.bib"
  - "content/papers/*/*.bib"
  - "content/papers/*/*/*.bib"
---

# Float Management

Configure `\graphicspath{\{figures/}}` in the preamble, use booktabs for all tables, and apply pandoc-crossref for automated cross-references. Figures must use PNG or JPG formats for Word compatibility.

## Rationale

Centralized graphics paths prevent absolute path references that break across machines and build environments. The booktabs package produces professional table rules without vertical lines, matching academic publishing standards. Pandoc-crossref generates numbered references automatically, eliminating manual numbering that drifts during revision. PNG and JPG formats ensure pandoc can produce both PDF and DOCX output from the same source, while PDF figures fail during Word conversion.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />

## Figure Conventions

Configure the graphics path once in the document preamble:

```latex
\graphicspath{{figures/}}
```

Omit file extensions in `\includegraphics` calls so LaTeX selects the best available format:

```latex
\includegraphics[width=\textwidth]{fig-results-overview}
```

Prefix filenames with `fig-` and a descriptive slug (e.g., `fig-results-overview.png`, `fig-architecture-layers.jpg`). Store all images in the `figures/` directory. Use PNG for raster graphics and JPG for photographs. Avoid PDF figures, nested figure environments such as `subfigure`, and `minipage` wrappers around figures. These constructs break pandoc's conversion to non-LaTeX formats.

In Pandoc Markdown, use the standard image syntax with a crossref label:

```markdown
![Caption text](fig-results-overview.png){#fig:results-overview width=80%}
```

## Table Conventions

Use Pandoc pipe tables in Markdown source with a caption and crossref label:

```markdown
| Method | Precision | Recall |
|--------|-----------|--------|
| Ours   | 0.94      | 0.91   |

Table: Comparison of methods {#tbl:comparison}
```

Always load booktabs in the LaTeX preamble for professional table formatting:

```latex
\usepackage{booktabs}
```

Extract tables exceeding five rows or requiring advanced formatting to standalone `.tex` files in `tables/` and include them via `\input{tables/table-name}`. Simple tables (fewer than five rows) may remain inline in Markdown source.

## Cross-References

Use pandoc-crossref for all figure, table, and equation references. Label elements with `\{#fig:label}`, `\{#tbl:label}`, or `\{#eq:label}` syntax. Reference them in text with `@fig:label`, `@tbl:label`, or `@eq:label`.

Build with the crossref filter enabled:

```bash
pandoc --filter pandoc-crossref --citeproc input.md -o output.pdf
```

## Examples

**Correct**: Relative path via graphicspath, no extension
```latex
\graphicspath{{figures/}}
\includegraphics[width=0.8\textwidth]{fig-accuracy-curve}
```

**Incorrect**: Absolute path with extension
```latex
\includegraphics[width=0.8\textwidth]{/home/user/project/figures/fig-accuracy-curve.pdf}
```

**Correct**: Pandoc pipe table with crossref label
```markdown
| Metric | Value |
|--------|-------|
| F1     | 0.92  |

Table: Performance results {#tbl:performance}
```

**Incorrect**: Raw LaTeX table without booktabs
```latex
\begin{tabular}{|l|r|}
\hline
Metric & Value \\
\hline
F1 & 0.92 \\
\hline
\end{tabular}
```

## Exceptions

Inline simple tables (fewer than five rows) directly in Markdown source without extracting to `tables/`. When targeting only LaTeX output with no DOCX requirement, PDF vector figures are acceptable.
