# Document Modularity

Separate LaTeX content by type using subdirectories with a minimal root document for assembly.

Organize LaTeX documents by separating content into typed subdirectories orchestrated through a minimal root file. The root document contains only preamble configuration and input statements; all substantive content lives in dedicated files under `sections/`, `figures/`, `tables/`, and `bibliography/`.

- `design/reference/ssot.md` in `github.com/ctxmod/engineering/context` (profile `principles`)
- `design/reference/dry.md` in `github.com/ctxmod/engineering/context` (profile `principles`)

## Why Modularity

Monolithic LaTeX documents degrade as they grow. A single 3000-line file makes version control diffs unreadable, blocks parallel collaboration (two authors editing different sections cause merge conflicts), and forces full recompilation for minor edits. Splitting content by type solves all three problems: diffs stay scoped, authors work on independent files, and selective compilation becomes possible.

Modularity also enforces SSOT at the file level. Each section exists in exactly one file. Shared definitions (macros, acronyms, notation) live in a dedicated preamble or definitions file rather than scattered across sections. When notation changes, a single edit propagates everywhere.

## Input vs Include

LaTeX provides two mechanisms for assembling multi-file documents: `\input{}` and `\include{}`. They serve different purposes.

| Mechanism | Page Break | Aux Files | Use Case |
|-----------|-----------|-----------|----------|
| `\input{sections/intro}` | No | Shared | Sections within a paper |
| `\include{chapters/methods}` | Yes | Separate per file | Chapters in a thesis or book |

`\input{}` inserts content seamlessly, as if the text were typed directly at the insertion point. Use it for papers and shorter documents where section transitions should not force page breaks. `\include{}` inserts content with forced page breaks before and after, and generates separate `.aux` files per included file. This enables `\includeonly{}` for selective compilation of individual chapters -- useful for theses exceeding 100 pages.

The default choice is `\input{}`. Reserve `\include{}` for documents where chapter-level `.aux` separation and `\includeonly{}` provide measurable benefit.

## Root Document Structure

The root document (`main.md` or `main.tex`) stays minimal. It handles three concerns: document class and package configuration, shared definitions, and content assembly.

```latex
\documentclass[sigconf]{acmart}

% Packages and configuration
\usepackage{booktabs}
\graphicspath{{figures/}}

% Shared definitions
\input{definitions}

\begin{document}

\input{sections/01-introduction}
\input{sections/02-related-work}
\input{sections/03-methods}
\input{sections/04-results}
\input{sections/05-discussion}

\bibliographystyle{ACM-Reference-Format}
\bibliography{bibliography/refs}

\end{document}
```

No substantive prose appears in the root file. Adding content directly to the root file bypasses the modular structure and creates an implicit dependency that tools and collaborators cannot track.

## Directory Organization

Each subdirectory holds one content type:

| Directory | Contents | Naming Convention |
|-----------|----------|-------------------|
| `sections/` | One file per major section | `{NN}-{slug}.md` or `.tex` |
| `figures/` | All images and diagrams | `fig-{descriptive-name}.{pdf,png}` |
| `tables/` | Complex standalone tables | `tbl-{descriptive-name}.tex` |
| `bibliography/` | Single bibliography source | `refs.bib` |
| `artifacts/` | Research materials | Subdirectories: `summaries/`, `analysis/`, `findings/` |

Number section files (`01-`, `02-`) to preserve ordering in file listings. Use descriptive slugs after the number so the filename communicates content without opening the file. Configure `\graphicspath{{figures/}}` in the preamble so figure references omit the directory prefix.

## Scalability Guidance

Paper length determines the appropriate granularity:

- **Short papers (< 10 pages)**: All content in `main.md` with no `sections/` directory. Modularity overhead exceeds its benefit at this scale.
- **Standard papers (10-40 pages)**: Flat `sections/` directory with one file per major section. This covers most conference and journal submissions.
- **Theses and books (100+ pages)**: Nested `chapters/` directory with `\include{}` for chapter-level separation. Each chapter may contain its own `sections/` subdirectory for further decomposition.

The `artifacts/` directory follows ACM artifact evaluation conventions, organizing research materials into `summaries/`, `analysis/`, and `findings/` subdirectories. This structure supports reproducibility review by placing materials where evaluators expect them.

## Related

- `design/reference/ssot.md` in `github.com/ctxmod/engineering/context` (profile `principles`)
- `design/reference/dry.md` in `github.com/ctxmod/engineering/context` (profile `principles`)
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/project_setup.md" />
- the academic organization reference of the writing procedure (agentic module, `workflows` profile, `reporting` package)
