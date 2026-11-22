---
paths:
  - "content/papers/*.md"
  - "content/papers/*/*.md"
  - "content/papers/*/*/*.md"
  - "content/papers/*.tex"
  - "content/papers/*/*.tex"
  - "content/papers/*/*/*.tex"
  - "content/papers/justfile"
  - "content/papers/*/justfile"
  - "content/papers/*/*/justfile"
  - "content/papers/Justfile"
  - "content/papers/*/Justfile"
  - "content/papers/*/*/Justfile"
---

# Project Structure

All LaTeX papers use the `content/papers/{paper-slug}/` directory layout with required subdirectories for sections, figures, tables, bibliography, and research artifacts.

## Rationale

Modular directory organization separates content by type, enabling independent version control of sections, parallel collaboration on figures and text, and selective compilation during drafting. The standardized layout also aligns with ACM artifact evaluation expectations, where reviewers locate materials by convention rather than documentation.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/modularity.md" />
- `composition/reference/imrad_structure.md` in `github.com/ctxmod/writing_academic/context` (profile `scholarly_prose`)

## Required Structure

```
content/papers/{paper-slug}/
  main.md                    # Root document with YAML frontmatter
  justfile                   # Build automation (pdf, watch, clean)
  sections/                  # Modular content files
    01-introduction.md
    02-related-work.md
    03-methods.md
    04-results.md
    05-discussion.md
  figures/                   # All images (PDF for vector, PNG for raster)
  tables/                    # Complex standalone table .tex files
  bibliography/
    refs.bib                 # Single bibliography source
  artifacts/                 # Research artifacts
    summaries/               # Research summaries
    analysis/                # Analysis outputs
    findings/                # Consolidated findings
```

## Examples

**Correct**: Modular layout with separated concerns
```
content/papers/cache-coherence/
  main.md
  justfile
  sections/
    01-introduction.md
    02-background.md
    03-design.md
    04-evaluation.md
    05-conclusion.md
  figures/
    fig-architecture.pdf
    fig-throughput.png
  tables/
    tbl-comparison.tex
  bibliography/
    refs.bib
  artifacts/
    summaries/
    analysis/
    findings/
```

**Incorrect**: Flat structure mixing content types
```
cache-coherence/
  paper.tex
  architecture.pdf
  throughput.png
  comparison.tex
  refs.bib
  notes.txt
```

This flat layout conflates figures with source files, lacks section modularity, and provides no artifact organization for research materials.

## Exceptions

Short documents under 10 pages may omit the `sections/` subdirectory and place all content directly in `main.md`. The `artifacts/` subdirectory hierarchy may be omitted when no research artifacts are produced. The `tables/` directory may be omitted when all tables are inline Pandoc pipe tables.
