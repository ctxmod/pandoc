# Pandoc

Pandoc publishing doctrine for ctxmod (`github.com/ctxmod/pandoc/context`). Requires `github.com/ctxmod/writing/context` and `github.com/ctxmod/writing_academic/context` (citation style practice lives there; this module owns the pipeline).

Profiles:

- `core` (default): packages `project` (modularity, integration, setup, build and structure rules), `references` (bibliography, cross-references, floats), `validation` (build errors, PDF quality).
- `styling`: tri-format output (PDF, DOCX, HTML) — package `formats` (semantic classes, template variables, DOCX reference document, Lua filters, cross-format consistency); extends `core`.

Select a profile by its qualified name, e.g. `github.com/ctxmod/pandoc/context/profiles/styling`.
