---
paths:
  - "content/papers/*.bib"
  - "content/papers/*/*.bib"
  - "content/papers/*/*/*.bib"
  - "content/papers/*.md"
  - "content/papers/*/*.md"
  - "content/papers/*/*/*.md"
  - "content/papers/*.tex"
  - "content/papers/*/*.tex"
  - "content/papers/*/*/*.tex"
  - "content/papers/*.csl"
  - "content/papers/*/*.csl"
  - "content/papers/*/*/*.csl"
---

# Bibliography Management

Maintain a single `bibliography/refs.bib` file per paper. Use citeproc for bibliography processing and a `.csl` file for citation style formatting. Avoid biblatex or natbib formatting packages in the LaTeX preamble.

## Rationale

A single bibliography file prevents duplicate entries and conflicting citation keys across scattered `.bib` files. Citeproc processes citations independently of the LaTeX engine, producing consistent output whether the target format is PDF, HTML, or DOCX. CSL (Citation Style Language) files define formatting rules declaratively, allowing style changes by swapping a single file rather than reconfiguring LaTeX packages. This separation keeps bibliography management portable across pandoc output formats.

- `citation_practice/reference/citation_practice.md` in `github.com/ctxmod/writing_academic/context` (profile `scholarly_prose`)
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::references@core/reference/bibliography.md" />

## Workflow

1. Store all references in `bibliography/refs.bib` at the paper root
2. Select a `.csl` file matching the target venue style (download from the [Zotero Style Repository](https://www.zotero.org/styles))
3. Configure YAML frontmatter in the root document:
   ```yaml
   bibliography: bibliography/refs.bib
   csl: ieee.csl
   ```
4. Use standard `\cite{}` commands or pandoc citation syntax (`@key`, `[@key]`) in source files
5. Build with `pandoc --citeproc` to process citations and generate the bibliography

## Examples

**Correct**: Citeproc with CSL formatting
```yaml
# YAML frontmatter
bibliography: bibliography/refs.bib
csl: acm-sig-proceedings.csl
```

```bash
pandoc --citeproc --bibliography=bibliography/refs.bib \
  --csl=acm-sig-proceedings.csl main.md -o main.pdf
```

**Incorrect**: Mixing citeproc with biblatex engine flags
```bash
# Do not combine --citeproc with --biblatex
pandoc --citeproc --biblatex main.md -o main.pdf
```

**Incorrect**: Multiple scattered bibliography files
```
paper/
  intro/refs.bib        # Fragmented bibliography
  methods/refs.bib      # Risks duplicate keys
  results/refs.bib      # Inconsistent entries
```

**Correct**: Single centralized bibliography
```
paper/
  bibliography/
    refs.bib            # All references in one file
  sections/
    intro.md
    methods.md
    results.md
```

## Reference Manager Integration

Export from Zotero, JabRef, or Mendeley to BibTeX format. Configure automatic export to `bibliography/refs.bib` when possible. Verify exported entries contain complete metadata: author, title, year, venue, and DOI or URL.

## Exceptions

Use `cite-method: biblatex` in YAML frontmatter only when the target venue requires raw BibLaTeX processing that citeproc cannot reproduce, such as journal-specific `.bbx` style files with no CSL equivalent.
