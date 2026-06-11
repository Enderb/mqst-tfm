# Add a citation

1. Add the BibTeX entry to `references.bib` following the existing key style (`AuthorYear`, e.g. `Blais2021`).
2. Reference it in the text with `\cite{Key}` or `\cite[p.~X]{Key}`.
3. Run a full build so BibTeX picks up the new entry:

```bash
pdflatex -shell-escape -interaction=nonstopmode main.tex && bibtex main && pdflatex -shell-escape -interaction=nonstopmode main.tex
```

The bibliography style is `unsrt` (citation-order). Do not change it — the template requires it.
