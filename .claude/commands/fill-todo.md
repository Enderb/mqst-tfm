# Find and fill TODO markers

List all open TODOs in the thesis:

```bash
grep -rn "\\\\todo{" chapters/ appendices/ main.tex
```

## Current TODO status (after June 2026 editing session)

Most TODOs have been resolved. The remaining open markers are:

| File | Location | Content |
|---|---|---|
| `chapters/03_hardware.tex` | system topology figure | Replace `figures/diag2` placeholder with real signal-chain diagram |
| `chapters/07_ml.tex` | QDA result | Add QDA fidelity (0.9062) and Mahalanobis-distance justification for LDA optimality |

## Items that were TODOs but are now resolved

- `chapters/07_ml.tex`: fidelity table — filled with real values (LDA 0.906, arch search 20 configs)
- `chapters/07_ml.tex`: IQ scatter plots and ROC curves — filled or explicitly deferred to figures wiring task
- `chapters/08_conclusions.tex`: ML experimental findings — filled with real results
- `chapters/05_readout.tex`: PFB implementation status — reframed as future work (no TODO)
- `chapters/06_feedback.tex`: π-pulse and active reset — reframed as designed/described (not a TODO)

## When filling a TODO

Replace `\todo{...}` entirely with real content. Only leave `\todo{...}` for content that is genuinely still pending. Do not leave empty `\todo{}` markers.

Use the confirmed numbers from CLAUDE.md (ML results section) when filling ch. 07 content. Do not invent or round numbers differently from what the notebooks produced.
