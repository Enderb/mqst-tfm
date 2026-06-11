# Add a figure to the thesis

1. Place the image file in `figures/` (PDF preferred for vector graphics; PNG for raster).
2. Reference it with:

```latex
\begin{figure}[t]
  \centering
  \includegraphics[width=0.85\columnwidth]{figures/my_figure}
  \caption{Caption text.}
  \label{fig:my_figure}
\end{figure}
```

Note: `\columnwidth` equals `\textwidth` in one-column mode — both work.

For side-by-side subfigures use the `subcaption` package (already loaded):

```latex
\begin{figure}[t]
  \centering
  \begin{subfigure}{0.48\columnwidth}
    \includegraphics[width=\linewidth]{figures/fig_a}
    \caption{Left panel.}
    \label{fig:a}
  \end{subfigure}
  \hfill
  \begin{subfigure}{0.48\columnwidth}
    \includegraphics[width=\linewidth]{figures/fig_b}
    \caption{Right panel.}
    \label{fig:b}
  \end{subfigure}
  \caption{Overall caption.}
  \label{fig:both}
\end{figure}
```

Notebook-generated figures are saved to `code/figures/` — copy or symlink them to `figures/` before compiling the thesis.
