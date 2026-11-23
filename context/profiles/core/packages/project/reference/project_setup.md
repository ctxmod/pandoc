# Project Setup

Step-by-step workflow for initializing a new LaTeX paper project with pandoc, latexmk, and Justfile.

Procedure for initializing a LaTeX paper project with the pandoc-first toolchain: pandoc with citeproc for bibliography processing, pandoc-crossref for automated numbering, latexmk for PDF compilation, and Just for build automation. The result is a modular directory structure ready for academic writing.

## Prerequisites

Gather before starting:

- **Paper slug**: Lowercase, hyphenated identifier (e.g., `cache-coherence`).
- **Tools installed**: pandoc, latexmk, just, pandoc-crossref filter.
- **Citation style**: CSL file for the target venue (download from the [Zotero Style Repository](https://www.zotero.org/styles)).

## Setup Workflow

### 1. Create the Directory Structure

Build the standard paper layout with all required subdirectories:

```bash
mkdir -p content/papers/{paper-slug}/{sections,figures,tables,bibliography,artifacts/{summaries,analysis,findings}}
```

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/modularity.md" />

### 2. Create the Root Document

Create `content/papers/{paper-slug}/main.md` with YAML frontmatter configuring pandoc, citeproc, and crossref:

```yaml
---
title: "Paper Title"
author: "Author Name"
date: 2026-01-01
bibliography: bibliography/refs.bib
csl: acm-sig-proceedings.csl
header-includes:
  - \usepackage{booktabs}
  - \graphicspath{{figures/}}
# pandoc-crossref reads its options as top-level metadata keys
chapters: false
figPrefix: ["Fig.", "Figs."]
tblPrefix: ["Table", "Tables"]
eqnPrefix: ["Eq.", "Eqs."]
secPrefix: ["Section", "Sections"]
linkReferences: true
---
```

### 3. Set Up the Bibliography

Create an empty `bibliography/refs.bib` and add an initial entry to verify the pipeline:

```bibtex
@article{knuth1984,
  author  = {Donald E. Knuth},
  title   = {Literate Programming},
  journal = {The Computer Journal},
  year    = {1984},
  volume  = {27},
  number  = {2},
  pages   = {97--111}
}
```

Test the citation in `main.md` with `[@knuth1984]` to confirm citeproc resolves the reference.

### 4. Create the Justfile

Add a `justfile` at the paper root with required recipes:

```just
default_target := "main"

pdf target=default_target:
    pandoc {{target}}.md sections/*.md \
        --standalone \
        --filter pandoc-crossref \
        --citeproc \
        --bibliography=bibliography/refs.bib \
        -o {{target}}.tex
    latexmk -pdf -interaction=nonstopmode -halt-on-error {{target}}.tex

watch target=default_target:
    latexmk -pdf -pvc -interaction=nonstopmode {{target}}.tex

docx target=default_target:
    pandoc {{target}}.md sections/*.md \
        --filter pandoc-crossref \
        --citeproc \
        --bibliography=bibliography/refs.bib \
        -o {{target}}.docx

clean:
    latexmk -C
    rm -f *.bbl *.run.xml *.nav *.snm *.vrb
```

Three details in these recipes carry the pipeline:

- **Section files are inputs.** Pandoc Markdown has no include directive, so the root document cannot pull in `sections/` by itself. The recipes pass `{{target}}.md sections/*.md`; pandoc concatenates the files in order, and the numeric prefixes (`01-`, `02-`) fix that order. A short document without `sections/` drops the glob.
- **The PDF goes through LaTeX source.** The `pdf` recipe has pandoc write a standalone `{{target}}.tex` with citations and cross-references resolved, then latexmk compiles it with as many passes as the auxiliary files need. `pandoc --pdf-engine=latexmk -o {{target}}.pdf` is the one-command equivalent. `watch` recompiles when the generated `.tex` changes; after editing Markdown, run `just pdf` again or drive it from a file watcher.
- **pandoc-crossref runs before citeproc.** Pandoc applies `--filter`, `--lua-filter`, and `--citeproc` in command-line order, and both use the `@label` syntax, so `--filter pandoc-crossref` must precede `--citeproc` in every recipe.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />

### 5. Initialize Version Control

Create `.gitignore` with entries for LaTeX auxiliary files:

```
*.aux
*.bbl
*.blg
*.log
*.nav
*.out
*.snm
*.toc
*.vrb
*.run.xml
*.fdb_latexmk
*.fls
*.synctex.gz
main.tex
```

Initialize the repository and commit the project skeleton.

### 6. Validate the Setup

Build an initial output and verify the toolchain:

```bash
just docx main
```

Confirm that the PDF or DOCX renders without errors, the test citation resolves, and cross-reference prefixes display correctly.

## Troubleshooting

**pandoc-crossref filter not found**: Install via `brew install pandoc-crossref` (macOS), `cabal install pandoc-crossref` (Haskell), or download a prebuilt binary from the GitHub releases page. Verify the filter version matches the installed pandoc version.

**Bibliography not found**: Pandoc resolves the `bibliography` path relative to the working directory of the pandoc process, then searches the `--resource-path`; it does not resolve it relative to the document's own location. Run the build from the paper root, where the Justfile sits, and use `bibliography: bibliography/refs.bib`.

**graphicspath has no effect in pandoc**: The `\graphicspath` directive applies only during LaTeX compilation. For pandoc Markdown images, use relative paths from the document root (e.g., `![Caption](figures/fig-overview.png)`).

## Rule Companion: Rationale, Recipes, Styling Flags, Example Justfiles, Examples, and Exceptions

*Companion to the `build` rule: its full rule set, worked examples and rationale.*

### Rationale

Manual compilation fails to scale for documents with bibliographies, cross-references, and multi-file structures. latexmk handles the required multiple compilation passes automatically, resolving auxiliary file dependencies without manual intervention. A Justfile provides a single entry point for all build operations, keeping commands discoverable and reproducible across environments.

### Required Recipes

| Recipe | Purpose | Primary Tool |
|--------|---------|--------------|
| `pdf TARGET` | Compile PDF from source | latexmk |
| `watch TARGET` | Auto-rebuild on file changes | latexmk -pvc |
| `clean` | Remove auxiliary files | rm |
| `docx TARGET` | Convert to Word format | pandoc |
| `html TARGET` | Convert to standalone HTML | pandoc |

### Optional Styling Flags

Projects that apply the pandoc-styling sub-profile gain access to format-specific templates and filters. The flags below are optional and do not affect the default recipes.

| Flag | Recipe | Purpose | When to Use |
|------|--------|---------|-------------|
| `--template=custom-pdf.tex` | pdf | Custom LaTeX template for PDF typography and layout | Project requires non-default PDF styling (custom fonts, headers, margins) |
| `--reference-doc=templates/reference.docx` | docx | Word reference document defining styles for DOCX output | Project needs consistent Word formatting (heading styles, page layout, custom styles) |
| `--css=templates/default.css` | html | CSS stylesheet for HTML typography and semantic styling | Project produces HTML output with custom visual design |
| `--lua-filter=filters/semantic-classes.lua` | all | Cross-format semantic class transformations | Project uses semantic markup (highlight, warning, abstract) that maps to format-specific styling |

### Example Justfile

The default Justfile works without any styling infrastructure. Add optional flags when the project applies the pandoc-styling sub-profile.

```just
# Default target
default_target := "main"

# Write LaTeX with pandoc, then compile the PDF with latexmk
pdf target=default_target:
    pandoc {{target}}.md sections/*.md \
        --standalone \
        --filter pandoc-crossref \
        --citeproc \
        --bibliography=bibliography/refs.bib \
        -o {{target}}.tex
    latexmk -pdf -interaction=nonstopmode -halt-on-error {{target}}.tex

# Recompile the generated LaTeX whenever it changes
watch target=default_target:
    latexmk -pdf -pvc -interaction=nonstopmode {{target}}.tex

# Convert to DOCX using pandoc with crossref, then citeproc
docx target=default_target:
    pandoc {{target}}.md sections/*.md \
        --filter pandoc-crossref \
        --citeproc \
        --bibliography=bibliography/refs.bib \
        -o {{target}}.docx

# Convert to standalone HTML using pandoc with crossref, then citeproc
html target=default_target:
    pandoc {{target}}.md sections/*.md \
        --filter pandoc-crossref \
        --citeproc \
        --bibliography=bibliography/refs.bib \
        --embed-resources \
        --standalone \
        -o {{target}}.html

# Remove auxiliary files
clean:
    latexmk -C
    rm -f *.bbl *.run.xml *.nav *.snm *.vrb
```

### Justfile with Styling Flags

Projects using the pandoc-styling sub-profile extend the default recipes with template, reference document, CSS, and Lua filter flags.

```just
# Default target
default_target := "main"

# Write LaTeX with the custom template and semantic class filter, then compile with latexmk
pdf target=default_target:
    pandoc {{target}}.md sections/*.md \
        --standalone \
        --filter pandoc-crossref \
        --citeproc \
        --bibliography=bibliography/refs.bib \
        --template=custom-pdf.tex \
        --lua-filter=filters/semantic-classes.lua \
        -o {{target}}.tex
    latexmk -pdf -interaction=nonstopmode -halt-on-error {{target}}.tex

# Recompile the generated LaTeX whenever it changes
watch target=default_target:
    latexmk -pdf -pvc -interaction=nonstopmode {{target}}.tex

# Convert to DOCX with reference document and semantic class filter
docx target=default_target:
    pandoc {{target}}.md sections/*.md \
        --filter pandoc-crossref \
        --citeproc \
        --bibliography=bibliography/refs.bib \
        --reference-doc=templates/reference.docx \
        --lua-filter=filters/semantic-classes.lua \
        -o {{target}}.docx

# Convert to standalone HTML with CSS and semantic class filter
html target=default_target:
    pandoc {{target}}.md sections/*.md \
        --filter pandoc-crossref \
        --citeproc \
        --bibliography=bibliography/refs.bib \
        --css=templates/default.css \
        --embed-resources \
        --standalone \
        --lua-filter=filters/semantic-classes.lua \
        -o {{target}}.html

# Build PDF from Markdown in one pandoc call with the custom template
pdf-pandoc target=default_target:
    pandoc {{target}}.md sections/*.md \
        --filter pandoc-crossref \
        --citeproc \
        --bibliography=bibliography/refs.bib \
        --pdf-engine=pdflatex \
        --template=custom-pdf.tex \
        --lua-filter=filters/semantic-classes.lua \
        -o {{target}}.pdf

# Remove auxiliary files
clean:
    latexmk -C
    rm -f *.bbl *.run.xml *.nav *.snm *.vrb
```

### Shared Options in a Defaults File

When the recipes repeat the same inputs and flags, a pandoc defaults file can hold them once, and each recipe passes it with `-d`. The Justfile stays the single entry point; the defaults file only removes repetition:

```yaml
# defaults/common.yaml
input-files:
  - main.md
  - sections/01-introduction.md
  - sections/02-related-work.md
filters:
  - pandoc-crossref
  - citeproc
bibliography: bibliography/refs.bib
```

```just
docx:
    pandoc -d defaults/common.yaml -o main.docx
```

The `filters` list runs in order, and `citeproc` in it stands for `--citeproc`, so crossref-before-citeproc is written once. Listing `input-files` explicitly also makes temporarily dropping a section a one-line edit.

### Examples

**Correct**: Single Justfile at the paper root with all five recipes defined, each using the appropriate tool for its output format.

**Correct**: Base project without styling sub-profile uses default recipes (no template, reference-doc, CSS, or Lua filter flags).

**Correct**: Project applying pandoc-styling adds `--reference-doc`, `--css`, and `--lua-filter` flags to respective recipes while keeping the same recipe names.

**Incorrect**: Makefile instead of Justfile, or manual `pdflatex` invocations that skip bibliography and cross-reference resolution passes.

**Incorrect**: Missing `html` recipe in a project that produces web output, forcing manual pandoc invocations with inconsistent flags.

**Incorrect**: Adding `--reference-doc` or `--css` flags without the corresponding template files in the project. Install the pandoc-styling sub-profile first.

### Exceptions

None. All pandoc projects require automated builds through a Justfile. The specific tool flags may vary by document class or journal requirements, but the five recipe names remain mandatory for projects producing those output formats. Projects that only produce PDF and DOCX may omit the `html` recipe.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/structure.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />

## Sources

- https://pandoc.org/MANUAL.html — multiple input files are concatenated; `--pdf-engine` values including `latexmk`; filters, Lua filters, and citeproc run in command-line order; defaults files (`input-files`, `filters` with `citeproc`, `bibliography`); `--resource-path` (pandoc 3.14, 2026-11-10); accessed 2026-11-23.
- https://lierdakil.github.io/pandoc-crossref/ — pandoc-crossref options as top-level metadata keys and the requirement to run it before citeproc; accessed 2026-11-23.
