# Pandoc Integration

Write in Pandoc Markdown with embedded LaTeX only for unsupported features.

Write primarily in Pandoc Markdown. Embed raw LaTeX only for features that Pandoc Markdown cannot express. This separation keeps content semantic and portable while preserving access to LaTeX typographic capabilities when necessary.

## Why Pandoc-First

Pandoc converts a single Markdown source to PDF (via LaTeX), DOCX, and HTML. A pandoc-first workflow produces documents that serve multiple output formats without rewriting content. Collaborators who do not use LaTeX can read and edit Markdown directly. Version control diffs remain meaningful because Markdown lacks the syntactic noise of raw LaTeX environments.

Layout-specific packages -- geometry, fancyhdr, setspace -- affect only PDF rendering. Pandoc ignores them during DOCX and HTML conversion. This means layout configuration belongs in the YAML metadata or a separate preamble file, not interleaved with content. Content stays clean; presentation stays separate.

- `design/reference/kiss.md` in `github.com/ctxmod/engineering/context` (profile `principles`)
- `basics/reference/clarity.md` in `github.com/ctxmod/writing_context/context` (profile `core`)

## Markdown as Primary Source

Standard Pandoc Markdown handles headings, lists, inline code, block quotes, footnotes, and math (`$...$` for inline, `$$...$$` for display). YAML frontmatter at the top of the root document configures title, authors, date, bibliography path, CSL style, and crossref settings.

Sections live as individual `.md` files in `sections/`. The root `main.md` orchestrates them. Each section file contains pure Markdown without preamble or document class declarations.

## Cross-References

The pandoc-crossref filter generates numbered cross-references for figures, tables, equations, and sections. Labels use the syntax `\{#fig:label}`, `\{#tbl:label}`, `\{#eq:label}`, and `\{#sec:label}`. In-text references use `@fig:label`, `@tbl:label`, and so on.

Configure crossref behavior in YAML frontmatter:

```yaml
crossref:
  chapters: false
  figPrefix: ["Fig.", "Figs."]
  tblPrefix: ["Table", "Tables"]
  eqnPrefix: ["Eq.", "Eqs."]
  secPrefix: ["Section", "Sections"]
  linkReferences: true
```

Build with the crossref filter before citeproc to ensure correct processing order:

```bash
pandoc --filter pandoc-crossref --citeproc input.md -o output.pdf
```

## When to Use Raw LaTeX

Embed raw LaTeX for theorem and proof environments, algorithm pseudocode (`algorithm2e`, `algorithmicx`), multi-column layouts within a section, and journal-specific macros required by the submission template. Wrap these blocks in Pandoc's raw attribute syntax:

````markdown
```{=latex}
\begin{theorem}
Content here.
\end{theorem}
```
````

Avoid defining custom macros. Pandoc's macro expander handles simple `\newcommand` definitions but fails on complex primitives involving `\expandafter`, `\csname`, conditional branching, or counter manipulation. When a macro would simplify repeated LaTeX, test that `pandoc --to=latex` expands it correctly before committing to its use.

## Trade-Offs

| Concern | Pandoc-first | LaTeX-first |
|---------|-------------|-------------|
| Output formats | PDF, DOCX, HTML from one source | PDF only without rewriting |
| Collaboration | Low barrier (Markdown literacy) | Requires LaTeX knowledge |
| Typographic control | Limited to LaTeX preamble + raw blocks | Full control throughout |
| Custom environments | Raw blocks, restricted macro use | Unrestricted |
| Version control | Clean diffs, readable content | Noisy diffs from LaTeX syntax |

Pandoc-first suits multi-format academic workflows where portability and collaboration outweigh fine typographic control. When a document targets only PDF and requires extensive custom environments, a LaTeX-first approach may be more appropriate. Most academic papers benefit from pandoc-first because revision and review cycles involve DOCX exchange with co-authors and reviewers.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::references@core/rules/floats.md" />

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/modularity.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::references@core/reference/cross_references.md" />
- `composition/reference/imrad_structure.md` in `github.com/ctxmod/writing_academic/context` (profile `scholarly_prose`)
- `design/reference/kiss.md` in `github.com/ctxmod/engineering/context` (profile `principles`)
- `basics/reference/clarity.md` in `github.com/ctxmod/writing_context/context` (profile `core`)
