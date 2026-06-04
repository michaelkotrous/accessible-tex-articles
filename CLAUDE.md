# Repository orientation

LaTeX article template + shared preamble for producing **accessible academic articles** in three formats from a single source: a tagged PDF/UA-2 PDF, an HTML5 page, and an ePub book. Maintained by Michael Kotrous (University of Georgia, Economics) for personal use; published publicly so others can fork it.

The README is the authoritative user-facing documentation. This file is a quick orientation for working *on* the repo.

## Layout

- `kotrous_article_template.tex` — article skeleton. Starts with `\DocumentMetadata{…}` (must remain the very first thing before `\documentclass` for tagging to engage), then sets up a `\ifdraftmode` toggle and `\input`s the preamble.
- `kotrous_article_preamble.tex` — the shared preamble. All package loads live here.
- `latexml-stubs.sty` — one-line stub (`\def\latexml{}`) preloaded by LaTeXML to flip the preamble's `\ifnotlatexml` switch.
- `examples/{consumption-notes,OLGdepreciation,prudence}/` — three real articles using the template. Each `\input`s `../../kotrous_article_preamble`. Each ships a final PDF, a `_draft.pdf` (line-numbered build), an HTML5 build under `html/`, and an ePub build under `epub/`.
- `README.md`, `LICENSE` (CC BY 4.0 full legal code), `.gitignore` (LaTeX build artifacts).

There is **no formal test suite**. Verify changes by recompiling at least one example to PDF and, if the preamble's LaTeXML guards were touched, also to HTML/ePub via the workflow in the README.

## Hard constraints

- **Engine: LuaLaTeX only.** The preamble uses `fontspec`, `unicode-math`, and `\DocumentMetadata` tagging. Do not switch to XeLaTeX or pdfLaTeX — neither will produce valid PDF/UA-2 output here.
- **TeX Live 2024+.** The `\DocumentMetadata` interface and the automatic tagging machinery stabilized in the 2024 kernel.
- **pgfplots `compat=1.18`** is pinned to match the currently installed pgfplots (1.18.2 in TeX Live 2026 basic). Bump only after verifying the installed version supports it.

## Two custom switches the preamble depends on

1. **`\ifnotlatexml`** (defined at the top of the preamble). Inverted-flag pattern: true when compiling with LuaLaTeX, false when LaTeXML is processing the source (because `latexml-stubs.sty` defines `\latexml`). Used to swap incompatible packages — biblatex/natbib, unicode-math/dsfont, hide siunitx and microtype during LaTeXML conversion.
2. **`\ifdraftmode`** (defined in the article, not the preamble). Toggles whether `lineno` is loaded and line numbers are emitted. Default `\draftmodetrue` in the master template; default `\draftmodefalse` in the three examples (since their published PDFs are final builds). Does *not* need to be flipped for LaTeXML conversion — LaTeXML ships its own `lineno` stub.

## Package load order invariants

The preamble has explicit `% ...must be loaded before...` comments around the load-order-sensitive packages, but to summarize:

- `csquotes` before `biblatex` (biblatex requirement).
- `xcolor` before `hyperref` (so the `url` color is defined before `hyperref` consumes it).
- `hyperref` before `biblatex` (biblatex documentation requirement; affects citation back-references).
- `fontspec` before `unicode-math`; `mathtools` before `unicode-math`.
- `pgfplots` loads `tikz` itself — do not also `\usepackage{tikz}`.
- `microtype` loaded last, guarded by `\ifnotlatexml`.

If you reorder packages, re-check these.

## LaTeXML conversion gotchas

- The `\DocumentMetadata{…}` block at the top of the article **must be temporarily removed** before running `latexmlc` — LaTeXML doesn't parse it and will error. Restore it before recompiling the PDF.
- Stale `.bbl`/`.aux` from prior biblatex/biber compiles can break LaTeXML's bibliography pass. Delete them first.
- `latexml-stubs.sty` lives at the repo root, **not** in a `preamble/` subdirectory. The `latexmlc` `--preload` argument and `--path` entries in the README reflect this: simple-fork layout uses `--preload=latexml-stubs.sty --path=.`; the `examples/<name>/` layout uses `--preload=../../latexml-stubs.sty --path=../..`.

## Commit / change conventions

- Repo has a single `main` branch and one author. No PR review process.
- Don't introduce a build/CI system unless the user asks — current verification is manual recompilation.
- Don't add `.md` files beyond README, LICENSE, and this CLAUDE.md unless explicitly requested.
- The user's email signature in the template (`michael.kotrous@uga.edu`) and affiliation (UGA Economics) are deliberate — do not generalize them to placeholders without being asked.
