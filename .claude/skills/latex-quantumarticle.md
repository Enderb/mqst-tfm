# LaTeX / quantumarticle conventions

## Document class options in use

```latex
\documentclass[a4paper,onecolumn,11pt,unpublished]{quantumarticle}
```

`unpublished` suppresses the journal header; `onecolumn` gives full-width text (hence `\columnwidth = \textwidth`).

## Custom commands defined in `quantumarticle.cls`

| Command | Purpose |
|---|---|
| `\supervisor{Name}` | Thesis supervisor line below author |
| `\address{...}` | Alias for `\affiliation` |
| `\ead{email}` | Author email (goes to footnote area) |

These are NOT standard LaTeX commands — they exist only in this class.

## Known class constraints

**No `\\` in `\title{}`**. The class saves the title in a single-line `\hbox` to measure its width and pick `\Huge` vs `\huge` font size. Inside that box `\raggedright` redefines `\\` to `\@centercr`, which calls `\par`, which corrupts list state and produces `! LaTeX Error: Something's wrong--perhaps a missing \item` at the first blank line after `\maketitle`. Long titles wrap automatically in the displayed output — the `\\` is never needed.

**Blacklisted font packages**: the class loads `lmodern`, `textcomp`, and related packages itself via `\AtBeginDocument`. Adding them manually in the preamble causes "option clash" or "already defined" errors.

**Abstract and `\maketitle` relationship**: the `abstract` environment calls `\@maketitle` internally. If `\maketitle` is called explicitly before `\begin{abstract}`, the class detects this (via `\global\let\@maketitle\relax`) and skips the second call. This is the pattern used in `main.tex`.

## Packages in use (preamble)

```latex
\usepackage[numbers,sort&compress]{natbib}  % citation style
\usepackage[nottoc]{tocbibind}              % bibliography in TOC
\usepackage{amsmath,amssymb,amsthm,bm,amsfonts,mathrsfs}
\usepackage{braket}          % \ket{}, \bra{}, \braket{}
\usepackage{siunitx}         % \SI{9.85}{\giga S/s}, \SIrange{}{}{}
\usepackage{algorithm}
\usepackage[noend]{algpseudocode}
\usepackage{minted}          % requires -shell-escape
\usepackage[babel]{microtype}
\usepackage{wrapfig,subcaption,xcolor}
```

## Math conventions in the thesis

- Braket notation: `\ket{0}`, `\bra{1}`, `\braket{i|g\hat{n}_q|j}`
- Bold vectors: `\mathbf{x}`, `\boldsymbol{\mu}`, `\boldsymbol{\Sigma}`
- SI units: always use `siunitx` macros, never type units manually
- Dispersive shift: `\chi` (not `\delta`)
- Readout fidelity: `\mathcal{F}`

## `\todo{}` macro

Defined in `main.tex`:
```latex
\newcommand{\todo}[1]{{\bfseries\color{red}[TODO: #1]}}
```

Renders as bold red text in the PDF. Remove when the section is complete.

## Bibliography

Style: `unsrt` (citation-order). Key format: `AuthorYear` (e.g. `Stefanazzi2022`, `Blais2021`). All entries are in `references.bib`.

## Compiling with minted

`minted` shells out to `pygments` for syntax highlighting. Always compile with:
```
pdflatex -shell-escape main.tex
```
Without `-shell-escape` the compile fails immediately with a write18 error. The `.vscode/settings.json` recipe already includes this flag.
