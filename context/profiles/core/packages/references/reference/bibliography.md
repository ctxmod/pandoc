# Bibliography Management Procedure

Step-by-step citeproc workflow for bibliography setup, citation usage, and reference manager integration.

Configure and maintain bibliography processing using pandoc's citeproc filter. This procedure covers BibTeX file setup, CSL style selection, citation syntax in source documents, and integration with reference managers.

## Prerequisites

- pandoc installed with citeproc support (bundled since pandoc 2.11)
- Paper directory structure created per project setup procedure
- Target venue citation style identified (APA, IEEE, ACM, Chicago, etc.)

## Steps

### 1. Create the Bibliography File

Create `bibliography/refs.bib` at the paper root. Each entry requires a unique citation key, complete author list, title, year, and venue:

```bibtex
@inproceedings{smith2024attention,
  author    = {Smith, Alice and Jones, Bob},
  title     = {Attention Mechanisms in Sparse Networks},
  booktitle = {Proceedings of NeurIPS},
  year      = {2024},
  doi       = {10.1234/example.2024.001}
}
```

Use descriptive keys following `{firstauthor}{year}{keyword}` convention. Include DOI or URL for every entry to support verification.

### 2. Obtain a CSL Style File

Download the `.csl` file matching your target venue from the [Zotero Style Repository](https://www.zotero.org/styles). Place it at the paper root alongside `main.md`:

```
paper/
  main.md
  acm-sig-proceedings.csl
  bibliography/
    refs.bib
```

Common styles: `ieee.csl`, `apa.csl`, `acm-sig-proceedings.csl`, `chicago-author-date.csl`. If no venue-specific style exists, select the closest disciplinary match.

### 3. Configure YAML Frontmatter

Add bibliography and CSL paths to the root document frontmatter:

```yaml
---
title: "Paper Title"
bibliography: bibliography/refs.bib
csl: acm-sig-proceedings.csl
---
```

Pandoc resolves these paths relative to the working directory at build time.

### 4. Use Citation Syntax in Source Files

Pandoc citeproc supports two citation forms that map to narrative and parenthetical styles:

| Syntax | Rendering | Use When |
|--------|-----------|----------|
| `@smith2024attention` | Smith (2024) | Emphasizing the author |
| `[@smith2024attention]` | (Smith, 2024) | Emphasizing the finding |
| `[@smith2024; @jones2023]` | (Smith, 2024; Jones, 2023) | Grouping multiple sources |
| `[-@smith2024attention]` | (2024) | Suppressing author name |
| `[@smith2024attention, p. 42]` | (Smith, 2024, p. 42) | Adding locator |

These correspond to LaTeX citation variants: `@key` replaces `\citet{}`, `[@key]` replaces `\citep{}`, and `[-@key]` replaces `\citeyear{}`.

### 5. Build with Citeproc

Run pandoc with the `--citeproc` flag to process citations and generate the bibliography:

```bash
pandoc --citeproc \
  --bibliography=bibliography/refs.bib \
  --csl=acm-sig-proceedings.csl \
  main.md -o main.pdf
```

When bibliography and CSL paths appear in YAML frontmatter, the command-line flags are optional. The `--citeproc` flag remains required to activate the filter.

### 6. Verify Citation Output

After building, check the PDF for:
- All `@key` references resolve to formatted citations
- Bibliography section appears at document end
- Citation style matches venue requirements
- No "?" placeholders indicating missing keys

Missing citations indicate either a typo in the citation key or a missing entry in `refs.bib`. Compare keys exactly, including case sensitivity.

## Reference Manager Integration

**Zotero**: Install the Better BibTeX plugin. Configure automatic export to `bibliography/refs.bib` in BibTeX format. Better BibTeX generates stable citation keys and updates the `.bib` file when the Zotero library changes.

**JabRef**: Open `bibliography/refs.bib` directly. JabRef provides a GUI for editing BibTeX entries, validates required fields, and detects duplicate keys. Save directly to the same file path.

**Mendeley**: Export the relevant collection to BibTeX format. Copy the exported file to `bibliography/refs.bib`. Mendeley does not support automatic export to a specific path, so re-export after adding new references.

All managers should export complete metadata: author, title, year, venue, and DOI or URL.

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Citation shows as `@key` in output | `--citeproc` flag missing | Add `--citeproc` to build command |
| "?" in place of citation | Key mismatch or missing entry | Verify key spelling in `.bib` file |
| Bibliography missing | No `\printbibliography` or citeproc not invoked | Citeproc appends bibliography automatically; verify flag |
| Encoding errors | Non-UTF8 characters in `.bib` | Save `.bib` as UTF-8, replace special characters with LaTeX escapes |
| Duplicate entries | Multiple exports without deduplication | Use stable citation keys, deduplicate by DOI |

- `citations/reference/citation_practice.md` in `github.com/ctxmod/writing_academic/context` (profile `scholarly_prose`)
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />

## Related

- `citations/reference/citation_practice.md` in `github.com/ctxmod/writing_academic/context` (profile `scholarly_prose`)
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::references@core/rules/bibliography.md" />
