# Pandoc 3.x: Versions and Features the Pipeline Relies On

The current pandoc release is 3.14 (2026-11-10); pandoc releases every
few weeks, and this module's recipes assume pandoc 3.x. This page
records the version facts that change how a pandoc project is built,
so a project can state its minimum version and check it.

## Pin the toolchain together

pandoc-crossref is compiled against a range of pandoc releases:
version 0.3.26 accepts pandoc 3.12 up to, but not including, 3.15. A
pandoc upgrade outside the filter's range breaks every recipe that
runs `--filter pandoc-crossref`, so upgrade the two together and
record both versions in the project's README or lockfile. Lua filters
can assert their own floor at load time:

```lua
PANDOC_VERSION:must_be_at_least '3.1.2'
```

## Features by version

| Since | Feature | What it changes for a project |
|-------|---------|-------------------------------|
| 2.8 | Defaults files (`-d`) | A YAML file holds inputs, filters, and options once; recipes pass it |
| 2.11 | Built-in citeproc (`--citeproc`) | No separate pandoc-citeproc binary; citeproc runs in command-line order with filters |
| 3.0 | `Figure` block in the document model | Captioned images, and tables placed as figures, are `Figure` elements; a Lua filter that looks for a paragraph holding one image misses them |
| 3.1.2 | Typst writer, `--pdf-engine=typst` | A LaTeX-free PDF route: `pandoc main.md -o main.pdf --pdf-engine=typst` |
| 3.1.3 | Typst reader | Typst sources can be converted to other formats |
| 3.1.12 | Djot reader and writer | Djot is a lightweight markup alternative with a stricter grammar than Markdown |
| 3.9 | `pdfstandard` variable | Tagged PDF/UA and PDF/A output through LuaLaTeX; see the PDF quality reference |
| 3.10 | `--typst-input` | Passes values to a Typst document's `sys.inputs` |

## Choosing a PDF engine

The LaTeX route stays this module's default for PDF: latexmk compiles,
and journal templates are LaTeX. `--pdf-engine` also accepts
`lualatex`, `xelatex`, `tectonic`, `context`, HTML-based engines
(`weasyprint`, `pagedjs-cli`, `prince`, `wkhtmltopdf`), `groff`, and
`typst`. Typst typically compiles much faster than LaTeX and needs no TeX
distribution, which makes it a reasonable engine for drafts and
internal reports. It is not a drop-in replacement: raw LaTeX blocks,
LaTeX packages in `header-includes`, and the LaTeX template variables
do not apply to Typst output, and a venue that requires a LaTeX class
still needs the LaTeX route. A project that adopts Typst for drafts
keeps the `pdf` recipe on latexmk and adds a separate recipe for the
Typst build.

## Sources

- https://pandoc.org/releases.html — release list; pandoc 3.14 dated
  2026-11-10; accessed 2026-11-23.
- https://github.com/jgm/pandoc/blob/main/changelog.md — `Figure`
  block (3.0, 2023-01-18), Typst writer and `--pdf-engine=typst`
  (3.1.2, 2023-03-26), Typst reader (3.1.3, 2023-06-07), djot (3.1.12,
  2024-02-14), `pdfstandard` (3.9, 2026-02-03), `--typst-input` (3.10,
  2026-06-03); accessed 2026-11-23.
- https://pandoc.org/MANUAL.html — `--pdf-engine` values and per-format
  defaults, defaults files (pandoc 3.14); accessed 2026-11-23.
- https://pandoc.org/lua-filters.html — `PANDOC_VERSION:must_be_at_least`,
  the `Figure` element; accessed 2026-11-23.
- https://github.com/lierdakil/pandoc-crossref — release 0.3.26, built
  against pandoc >=3.12 && <3.15; accessed 2026-11-23.
