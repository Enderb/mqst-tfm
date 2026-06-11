# Run the analysis notebooks

The notebooks must be run from the `code/` directory so that relative paths (`./ssro_h5/`, `./figures/`) resolve correctly.

```bash
cd code

# IQ classifier pipeline (LDA / QDA / MLP on integrated IQ)
jupyter nbconvert --to notebook --execute SSRO_readout_ML.ipynb --output SSRO_readout_ML.ipynb

# Full ML discrimination + CNN on synthetic traces
jupyter nbconvert --to notebook --execute SSRO_ML_discrimination.ipynb --output SSRO_ML_discrimination.ipynb

# Synthetic raw-trace generation + 1D CNN
jupyter nbconvert --to notebook --execute SSRO_synthetic_CNN.ipynb --output SSRO_synthetic_CNN.ipynb

# Raw waveform capture (requires live ZCU216 connection)
jupyter nbconvert --to notebook --execute ssro_raw_waveform.ipynb --output ssro_raw_waveform.ipynb
```

`ssro_raw_waveform.ipynb` requires a live ZCU216 board (`QickSoc()` call will fail without one). The other three notebooks work offline with the HDF5 data in `ssro_h5/`.

Generated figures land in `code/figures/`. To use them in the thesis, copy them to `figures/`:

```bash
cp code/figures/fig*.pdf figures/
```
