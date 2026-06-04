# Accessible LaTeX Articles

A LaTeX article template and shared preamble for producing **accessible academic articles** in three formats from a single source: a tagged **PDF/UA-2** PDF, an **HTML5** page, and an **ePub** book.

The template is the one I use for my own working papers and course notes; it is shared here so others can fork it as a starting point.

## What's in the repo

| File | Purpose |
| --- | --- |
| `kotrous_article_template.tex` | Article skeleton — title page, abstract, bibliography wiring, header/footer setup, PDF metadata for tagging, and an `\ifdraftmode` toggle for line-numbered review drafts. |
| `kotrous_article_preamble.tex` | Shared preamble: fonts, math, tables, figures, bibliography, colors, links, and a `\ifnotlatexml` switch that swaps incompatible packages when the source is being converted by LaTeXML. |
| `latexml-stubs.sty` | One-line stub (`\def\latexml{}`) preloaded by LaTeXML to flip the `\ifnotlatexml` switch to `false`. |
| `examples/` | Three articles compiled to PDF, HTML5, and ePub using this template (`consumption-notes`, `OLGdepreciation`, `prudence`). |

## Requirements

- **LuaLaTeX** — required engine. The preamble uses `fontspec` and `unicode-math`, and the PDF/UA-2 `\DocumentMetadata` tagging interface is best supported on LuaLaTeX.
- **Biber** — bibliography backend used by `biblatex`.
- **NewComputerModernMath** — OpenType math font.
- **Source Serif 4**, **Source Sans 3**, **Source Code Pro** — text and monospace fonts.
- **LaTeXML** — only needed if you want to produce HTML5 and ePub outputs.

### Installation hints

**TeX Live 2024 or later** is required. The `\DocumentMetadata{…}` interface and the automatic tagging machinery used by the template reached stability in the 2024 release of the LaTeX kernel, so earlier installs will not produce valid PDF/UA-2 output. A standard TeX Live install bundles **LuaLaTeX, Biber, `latexmk`, and NewComputerModernMath** — you do not need to install those separately.

**LaTeXML** is a Perl program with platform-specific install paths:

- macOS: `brew install latexml`
- Debian / Ubuntu: `sudo apt install latexml`
- Otherwise: install via CPAN (`cpanm LaTeXML`) per the [LaTeXML install guide](https://math.nist.gov/~BMiller/LaTeXML/get.html).

**Source fonts.** The Adobe Source family is published as open source on GitHub:

- Source Serif 4 — <https://github.com/adobe-fonts/source-serif>
- Source Sans 3 — <https://github.com/adobe-fonts/source-sans>
- Source Code Pro — <https://github.com/adobe-fonts/source-code-pro>

Download the latest release's OTF bundle and install it system-wide (drag into Font Book on macOS, copy to `~/.fonts/` on Linux, install via Settings on Windows). The same fonts are also available through Google Fonts and Adobe Fonts if you prefer a font-manager workflow.

### Verifying your install

```shell
lualatex --version | head -1     # expect "This is LuaHBTeX, Version ..."
biber --version                  # expect "biber version: 2.x"
latexml --VERSION                # expect "latexml (LaTeXML version 0.8.x)"  (only if building HTML/ePub)
fc-list | grep -iE "Source (Serif|Sans|Code)|NewComputerModern"
```

The final `fc-list` line should print several entries for each of the four font families; an empty result means the fonts are not visible to your TeX install and `fontspec` will fail.

## Quick start

1. Fork or clone the repo.
2. Copy `kotrous_article_template.tex` and `kotrous_article_preamble.tex` into a new article directory (or work from this directory directly).
3. Place `.bib` files in a `references/` subdirectory next to your article (the template uses `\addbibresource{references/*.bib}`).
4. Edit the template:
   - Fill in `\title{}`, `\hypersetup{pdftitle=…}`, `\author`, `\affil`, and `\date{}`.
   - Replace the abstract and JEL codes.
   - Drop your content where the `% content here` comment is.
5. Compile with LuaLaTeX + Biber, e.g.:
   ```shell
   latexmk -lualatex your-article.tex
   ```

The template includes an `\ifdraftmode` switch (set to `\draftmodetrue` by default) that loads `lineno` and enables line numbers for review drafts. Change the line to `\draftmodefalse` (or comment it out) before compiling the final/submission PDF — `lineno` will not be loaded at all in that build.

## Producing a tagged (accessible) PDF

The template's `\DocumentMetadata{...}` block declares PDF/UA-2 conformance, English language, automatic tagging, and the `mathml-SE` math setup. Compile with LuaLaTeX as normal and the output PDF will carry the accessibility tags.

You can verify the resulting PDF with a tool such as [PAC](https://pdfua.foundation/en/pdf-accessibility-checker-pac/) or Adobe Acrobat's accessibility checker.

## Producing HTML5 and ePub via LaTeXML

The preamble guards LuaLaTeX-only packages (`fontspec`, `unicode-math`, `biblatex`, `siunitx`, `microtype`) behind `\ifnotlatexml … \fi` and provides equivalent fallbacks (`natbib`, `amsfonts`/`dsfont`) when LaTeXML is processing the source.

The commands below assume the article, `kotrous_article_preamble.tex`, `latexml-stubs.sty`, and a `references/` subdirectory all sit in the same working directory (the simplest forking layout). The `examples/` articles use a different layout — they live in `examples/<name>/` while the preamble and stub stay at the repo root — so for those, replace `--preload=latexml-stubs.sty` with `--preload=../../latexml-stubs.sty` and `--path=.` with `--path=../..`.

The conversion workflow:

1. **Prepare the working directory.**
   - Run the conversion from the directory containing the article `.tex`.
   - Create `html/` and `epub/` subdirectories for the outputs.
   - Temporarily remove the `\DocumentMetadata{…}` block from the top of the article — LaTeXML does not understand it.
   - Delete stale build artifacts from previous PDF compiles (e.g. `.bbl`, `.aux`) to avoid bibliography conflicts.
   - The `\ifdraftmode` toggle does **not** need to be flipped — LaTeXML ships with a `lineno` stub that silently ignores line numbering, so the HTML5 and ePub outputs are identical whether draft mode is on or off.

2. **Convert the bibliography to XML** (only if the article cites references; replace `references.bib` with the actual filename, e.g. `olg.bib`):
   ```shell
   latexml --destination=html/references.bib.xml references/references.bib
   latexml --destination=epub/references.bib.xml references/references.bib
   ```

3. **Convert the article to HTML5:**
   ```shell
   latexmlc article.tex \
     --preload=latexml-stubs.sty \
     --path=. \
     --path=references \
     --bibliography=html/references.bib.xml \
     --format=html5 \
     --destination=html/article.html
   ```

4. **Convert the article to ePub:**
   ```shell
   latexmlc article.tex \
     --preload=latexml-stubs.sty \
     --path=. \
     --path=references \
     --bibliography=epub/references.bib.xml \
     --format=epub \
     --destination=epub/article.epub
   ```

5. **Restore `\DocumentMetadata{…}`** and recompile the tagged PDF.

The `--preload=latexml-stubs.sty` flag is what defines `\latexml` so the preamble's `\ifnotlatexml` switch routes to the LaTeXML-safe code path. Add further `--path=<dir>` entries if your article `\input`s figures, tables, or other helpers from additional subdirectories.

## What the preamble provides

- **Typography** — Source Serif 4 / Source Sans 3 / Source Code Pro via `fontspec`; 1.3 line spread; `microtype` for character protrusion and font expansion; `csquotes` with a custom `pullquote` environment.
- **Math** — `mathtools` plus `unicode-math` with NewComputerModernMath; `\indicator` macro for the indicator function (with an `\mathbb{1}` / `\mathds{1}` swap depending on engine).
- **Bibliography** — `biblatex` (author-year, Biber) for PDF; `natbib` with `\textcite`/`\parencite` shims for LaTeXML.
- **Tables** — `booktabs`, `longtable`, `threeparttable`, `lscape`, a centered fixed-width column type `P{…}`, and `siunitx` configured for decimal-aligned numeric columns.
- **Figures** — `graphicx`, `subcaption`, plus `tikz` and `pgfplots`.
- **Color** — `xcolor` with an eight-color colorblind-friendly palette (`cb1`–`cb8`) ready to use in plots and figures.
- **Links** — `hyperref` with a muted blue (`#2a7f9e`) for URLs and citations, black for internal links.
- **Lists** — `enumitem` for tidy enumerate/itemize customization.
- **Author block** — `authblk` for multi-affiliation author lines.

## Examples

The `examples/` directory contains three articles I have compiled with this template. Each has the source `.tex`, two tagged PDFs (the final version and a line-numbered draft build), an HTML5 build under `html/`, and an ePub build under `epub/`.

## License

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/). See [`LICENSE`](LICENSE) for the full text.
