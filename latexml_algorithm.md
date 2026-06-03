1. Configure environment
    * Set current directory in Terminal so that article tex is at root.
    * Create `epub` and `html` subdirectories.
    * Strip \DocumentMetadata from top of article before conversion (LaTeXML doesn't like it).
    * Remove all extraneous files from previous compiles (can cause issues if references compiled using biblatex prior, for example).
2. If the article includes references or academic citations, convert the bib file to XML first.
```{shell}
latexml --destination=html/references.bib.xml references/references.bib
latexml --destination=epub/references.bib.xml references/references.bib
```
3. Convert the article to HTML5
```{shell}
latexmlc article.tex \
  --preload=preamble/latexml-stubs.sty \
  --path=preamble \
  --path=figures \
  --path=tables \
  --path=references \
  --bibliography=html/references.bib.xml \
  --format=html5 \
  --destination=html/article.html
```
4. Convert the article to ePub 
```{shell}
latexmlc article.tex \
  --preload=preamble/latexml-stubs.sty \
  --path=preamble \
  --path=figures \
  --path=tables \
  --path=references \
  --bibliography=epub/references.bib.xml \
  --format=epub \
  --destination=epub/article.epub
```
5. Add back `\DocumentMetadata` and recompile the tagged PDF version of the article.

