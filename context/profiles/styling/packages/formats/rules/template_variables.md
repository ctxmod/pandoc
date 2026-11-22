---
paths:
  - "content/*.md"
  - "content/*/*.md"
  - "content/*/*/*.md"
  - "content/*/*/*/*.md"
  - "papers/*.md"
  - "papers/*/*.md"
  - "papers/*/*/*.md"
  - "papers/*/*/*/*.md"
  - "*.md"
  - "*/*.md"
  - "*/*/*.md"
  - "*/*/*/*.md"
  - "*/*/*/*/*.md"
---

# Template Variables in YAML Metadata

Define all template-configurable values as YAML frontmatter variables. Prohibit hardcoding values in document content or build commands when those values control layout, styling, or conditional document features. Pandoc templates interpolate frontmatter variables (`$var$`), so one metadata block governs the document across all output formats and defaults files can supply project-wide settings that individual documents override selectively. Command-line `-V`/`--metadata` variables override frontmatter silently, creating inconsistency between what the document declares and what the build produces.

Every document should declare applicable variables from the four standard categories — document metadata (`title`, `author`, `date`, `abstract`, `keywords`), layout control (`frontpage`, `secondpage`, `toc`, `toc-depth`, `numbersections`), styling configuration (`fontsize`, `geometry`, `linestretch`, `mainfont`, `sansfont`, `monofont`), and conditional features (`fancyhdr-enabled`, `wallpaper-path`, `colorlinks`, `linkcolor`, `citecolor`). Build-time overrides are acceptable only for CI/CD values not known at authoring time (build numbers, commit hashes, generated timestamps), documented in the project Justfile so they remain discoverable.

- the full variable tables with types and purposes, and the correct/incorrect examples — <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::formats@styling/reference/template_variables.md" />
- why layout configuration belongs in the YAML metadata block rather than interleaved with content — <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/reference/pandoc_integration.md" />
- the build recipes these variables feed — <ctxmod_link path="ctx:github.com/ctxmod/pandoc/context::shared::project@core/rules/build.md" />
