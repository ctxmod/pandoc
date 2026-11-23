# Build Error Troubleshooting

Troubleshooting workflow for LaTeX compilation errors including diagnostic commands, error classification, and isolation techniques.

Procedure for diagnosing and resolving LaTeX compilation failures. Build errors fall into two severity classes: fatal errors that halt PDF generation, and warnings that produce output with quality degradation. This procedure covers both latexmk-driven and pandoc-driven build pipelines.

## Prerequisites

- A build failure or warning from `just pdf` or `just docx`
- Access to the project's `.log` file (generated alongside `.aux` files)
- latexmk and pandoc installed per project setup

## Diagnostic Workflow

### 1. Capture the Full Error

Run the build with nonstop mode to collect all errors in a single pass rather than halting at the first:

```bash
latexmk -pdf -interaction=nonstopmode main.tex 2>&1 | tee build.log
```

For pandoc pipelines, enable verbose output:

```bash
pandoc main.md sections/*.md --filter pandoc-crossref --citeproc \
    --bibliography=bibliography/refs.bib --verbose -o main.pdf 2>&1 | tee build.log
```

### 2. Classify the Error

Locate the error in `build.log` or the `.log` file. LaTeX errors follow the pattern `! ErrorType` with a file path and line number on the preceding line.

| Error Class | Pattern in Log | Severity | Typical Cause |
|-------------|---------------|----------|---------------|
| Missing package | `! LaTeX Error: File 'pkg.sty' not found` | Fatal | Package not installed or misspelled |
| Undefined reference | `LaTeX Warning: Reference 'fig:x' undefined` | Warning | Label missing or misspelled |
| Undefined citation | `LaTeX Warning: Citation 'key' undefined` | Warning | BibTeX key absent from refs.bib |
| Missing file | `! LaTeX Error: File 'img' not found` | Fatal | Wrong path or missing `\graphicspath` |
| Syntax error | `! Missing $ inserted` or `! Undefined control sequence` | Fatal | Unmatched delimiters or typo in command |
| BibTeX/citeproc | `I found no \citation commands` | Warning | `--citeproc` flag missing or empty bibliography |
| Pandoc filter | `Error running filter pandoc-crossref` | Fatal | Filter not installed or incompatible version |
| Missing citation key | `[WARNING] Citeproc: citation key not found` | Warning | Key absent from `refs.bib` or misspelled |

### 3. Inspect the Log File

The `.log` file contains the complete compilation trace. Search for `!` at the start of a line to find fatal errors, and `Warning` for non-fatal issues:

```bash
grep -n "^!" main.log
grep -n "Warning" main.log
```

For each error, the log reports the source file and line number. Cross-reference these against your source files to locate the problem.

### 4. Isolate the Error Source

When the error source is unclear, use `\input{}` commenting to narrow the scope. Comment out sections in the root document until the build succeeds, then re-enable sections one at a time:

```latex
\input{sections/01-introduction}
% \input{sections/02-methods}      % Temporarily disabled
% \input{sections/03-results}      % Temporarily disabled
\input{sections/04-discussion}
```

For pandoc Markdown projects, apply the same technique by dropping section files from the recipe's input list, or from the `input-files` list of a defaults file. To see the LaTeX pandoc generates before any engine runs, write it out with `pandoc ... -s -o main.tex` and compile that file directly; an error then points at a line you can read.

## Common Resolutions

**Missing packages**: Install via your TeX distribution's package manager (`tlmgr install pkgname`) or add to the document preamble. Verify the package name matches the `.sty` filename.

**Undefined references**: Rebuild twice. LaTeX resolves forward references on subsequent passes. If the warning persists, verify the label exists and matches the reference exactly, including prefix (`fig:`, `tbl:`, `eq:`).

**BibTeX/citeproc failures**: Confirm `bibliography/refs.bib` exists and contains the cited keys. For citeproc, verify the `--citeproc` and `--bibliography` flags are both present. For BibTeX, run `bibtex main` separately to see detailed errors.

**Pandoc filter errors**: Verify pandoc-crossref is installed (`pandoc-crossref --version`) and compatible with the installed pandoc version. Version mismatches between pandoc and pandoc-crossref cause silent filter failures.

**Syntax errors**: Check for unmatched `$`, `{`, `}`, or `\begin`/`\end` pairs. LaTeX reports the line where it detected the problem, which may differ from where the actual mismatch occurs. Search backward from the reported line.

## Severity Classification

| Severity | Build Outcome | Action |
|----------|---------------|--------|
| Fatal | PDF not generated | Resolve before continuing |
| Warning | PDF generated with issues | Resolve before submission |
| Info | PDF generated correctly | Review at next opportunity |

Warnings accumulate. A document with unresolved reference warnings produces incorrect cross-references in the final PDF, so treat persistent warnings as pre-submission blockers.

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/project_setup.md" />
- `design/reference/kiss.md` in `github.com/ctxmod/engineering/context` (profile `principles`)

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />

## Sources

- https://pandoc.org/MANUAL.html — filter and citeproc ordering, `input-files` in defaults files, `--verbose` (pandoc 3.14, 2026-11-10); accessed 2026-11-23.
