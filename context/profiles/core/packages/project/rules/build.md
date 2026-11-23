---
paths:
  - "justfile"
  - "*/justfile"
  - "*/*/justfile"
  - "*/*/*/justfile"
  - "*/*/*/*/justfile"
  - "Justfile"
  - "*/Justfile"
  - "*/*/Justfile"
  - "*/*/*/Justfile"
  - "*/*/*/*/Justfile"
  - "*.tex"
  - "*/*.tex"
  - "*/*/*.tex"
  - "*/*/*/*.tex"
  - "*/*/*/*/*.tex"
  - "*.md"
  - "*/*.md"
  - "*/*/*.md"
  - "*/*/*/*.md"
  - "*/*/*/*/*.md"
---

# Build Automation

All pandoc projects include a Justfile with `pdf`, `watch`, `clean`, `docx`, and `html` recipes. latexmk compiles PDFs — it runs the multiple compilation passes that resolve auxiliary-file dependencies automatically — and pandoc with citeproc and pandoc-crossref produces DOCX and HTML. The five recipe names are mandatory; tool flags may vary by document class or journal requirements. A project producing web output must not lack its `html` recipe; projects that only produce PDF and DOCX may omit it.

Never a Makefile, and never manual `pdflatex` invocations that skip bibliography and cross-reference resolution passes. The default recipes work without styling infrastructure; `--template`, `--reference-doc`, `--css`, and `--lua-filter` flags are optional additions for projects that apply the pandoc-styling sub-profile — never add them without the corresponding template files in the project (install the sub-profile first).

- KISS — `design/reference/kiss.md` in `github.com/ctxmod/engineering/context` (profile `principles`)
- the required-recipe and optional styling-flag tables, the complete default and styling-extended Justfiles, rationale, and worked correct/incorrect examples — <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/project_setup.md" />
- the pandoc and pandoc-crossref version facts the recipes rely on, and the PDF engine choices beside latexmk — <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_versions.md" />
- build-failure diagnosis for the latexmk and pandoc pipelines — <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::validation@core/reference/build_errors.md" />
