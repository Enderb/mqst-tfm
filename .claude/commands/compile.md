# Compile thesis

Full build with citations:

```bash
pdflatex -shell-escape -interaction=nonstopmode main.tex && \
bibtex main && \
pdflatex -shell-escape -interaction=nonstopmode main.tex && \
pdflatex -shell-escape -interaction=nonstopmode main.tex
```

Quick draft (no bibliography update):

```bash
pdflatex -shell-escape -interaction=nonstopmode main.tex
```

Check for errors only:

```bash
pdflatex -shell-escape -interaction=nonstopmode main.tex 2>&1 | grep -E "^!|Error:"
```

Check page count of main text (excluding appendices):

```bash
grep -c "^\\\\section" chapters/*.tex
```
