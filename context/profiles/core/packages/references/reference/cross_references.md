# Cross-Reference Configuration

Configure pandoc-crossref for automated figure, table, equation, and section references.

Configure pandoc-crossref to generate numbered references for figures, tables, equations, and sections. This procedure covers filter installation, Justfile integration, label syntax, and YAML metadata configuration.

## Prerequisites

- Pandoc installed (version 2.11 or later)
- Just command runner available
- Paper directory initialized per project-setup procedure

## Steps

### 1. Install pandoc-crossref

Install the pandoc-crossref filter matching your pandoc version. Version compatibility matters: a mismatched filter produces silent failures or crashes.

```bash
# macOS
brew install pandoc-crossref

# Verify installation
pandoc-crossref --version
```

Confirm the reported pandoc API version matches your installed pandoc version. Mismatches between filter and pandoc versions cause build failures without clear error messages.

### 2. Configure the Filter in Justfile

Add `--filter pandoc-crossref` to the pandoc invocation. The crossref filter must run before `--citeproc` because pandoc-crossref processes labels that citeproc would otherwise misinterpret as citation keys.

```just
pdf target:
    pandoc --filter pandoc-crossref \
           --citeproc \
           --pdf-engine=latexmk \
           --bibliography=bibliography/refs.bib \
           content/papers/{{target}}/main.md \
           -o content/papers/{{target}}/main.pdf
```

Filter ordering is significant. Placing `--citeproc` before `--filter pandoc-crossref` causes equation and figure labels prefixed with `@` to be consumed by the citation processor, producing "reference not found" warnings and missing cross-references in the output.

### 3. Add Labels to Elements

Apply labels using the `\{#prefix:label}` attribute syntax after each element.

**Figures**:

```markdown
![Accuracy over training epochs](fig-accuracy-curve.png){#fig:accuracy-curve width=80%}
```

**Tables**:

```markdown
| Method | Precision | Recall |
|--------|-----------|--------|
| Ours   | 0.94      | 0.91   |

Table: Comparison of detection methods {#tbl:detection-comparison}
```

**Equations**:

```markdown
$$E = mc^2$$ {#eq:mass-energy}
```

The equation label must appear on the same line as the closing `$$` delimiter, separated by a space. Placing the label on a subsequent line causes pandoc-crossref to ignore it.

**Sections**:

```markdown
## Methodology {#sec:methodology}
```

### 4. Reference Labels in Text

Use `@prefix:label` to insert numbered references. Pandoc-crossref resolves these to the formatted reference string during compilation.

```markdown
As shown in @fig:accuracy-curve, the model converges after 50 epochs.
@tbl:detection-comparison summarizes the results.
Applying @eq:mass-energy to the measured values yields consistent predictions.
The experimental protocol follows @sec:methodology.
```

Multiple references use semicolons: `@fig:accuracy-curve;@fig:loss-curve` renders as "Figs. 1, 2" with the default prefix configuration.

### 5. Configure YAML Metadata

Add crossref settings to the document's YAML frontmatter to control prefix strings, numbering, and hyperlinking.

```yaml
---
crossref:
  chapters: false
  figPrefix: ["Fig.", "Figs."]
  eqnPrefix: ["Eq.", "Eqs."]
  tblPrefix: ["Table", "Tables"]
  secPrefix: ["Section", "Sections"]
  numberSections: true
  linkReferences: true
---
```

Set `chapters: true` only for multi-chapter documents such as theses. For single papers, `chapters: false` produces flat sequential numbering (Fig. 1, Fig. 2) rather than chapter-relative numbering (Fig. 1.1, Fig. 1.2).

### 6. Verify References Resolve

Build the PDF and check for unresolved references:

```bash
just pdf {paper-slug}
```

Search the output for `**???**` markers, which indicate labels pandoc-crossref could not resolve. Common causes: misspelled label names, missing `#` in the attribute, or a label defined after its first reference in a single-pass build.

## Troubleshooting

| Symptom | Cause | Resolution |
|---------|-------|------------|
| `**???**` in output | Unresolved label | Verify label spelling matches reference exactly |
| Equation label ignored | Label on separate line | Move `\{#eq:label}` to same line as closing `$$` |
| References show as citations | Filter order wrong | Place `--filter pandoc-crossref` before `--citeproc` |
| Duplicate numbering | Duplicate label names | Ensure each `\{#prefix:label}` is unique across the document |
| "Pandoc-crossref not found" | Filter not installed | Install via package manager, verify PATH |

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::references@core/rules/floats.md" />
