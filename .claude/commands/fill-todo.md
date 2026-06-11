# Find and fill TODO markers

List all open TODOs in the thesis:

```bash
grep -rn "\\\\todo{" chapters/ appendices/ main.tex
```

TODOs that require experimental data (cannot be filled without measurements):
- `chapters/07_ml.tex`: fidelity table values (F%, FPGA latency, DSP slices per classifier)
- `chapters/07_ml.tex`: IQ scatter plots with decision boundaries, ROC curve comparison
- `chapters/08_conclusions.tex`: ML experimental findings summary
- `chapters/05_readout.tex`: PFB implementation status and preliminary results
- `chapters/03_hardware.tex`: replace `figures/diag2` placeholder with real system topology diagram

When filling a TODO with real data, replace `\todo{...}` entirely. Use `\todo{...}` only for content still genuinely pending.
