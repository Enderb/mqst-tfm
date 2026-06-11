# HDF5 SSRO data formats

## Integrated IQ data (`ssro_h5/SSRO_0.h5` … `SSRO_4.h5`)

These files contain averaged (accumulated) IQ readout results:

```python
with h5py.File('./ssro_h5/SSRO_0.h5', 'r') as f:
    results      = f['results'][:]       # shape (2, 2001, 2)
    state_labels = f['loops/state'][:]   # [b'off', b'on']
```

Array layout:
- `axis 0` — prepared state (0 = |0⟩ "off", 1 = |1⟩ "on")
- `axis 1` — shot index (2001 shots; drop index 0 as a potential acquisition artefact)
- `axis 2` — [I, Q] quadratures in ADU

Standard loading pattern used in all ML notebooks:

```python
X0 = results[0, 1:, :]   # |0⟩ IQ, shape (2000, 2)
X1 = results[1, 1:, :]   # |1⟩ IQ, shape (2000, 2)
X  = np.vstack([X0, X1])
y  = np.array([0]*len(X0) + [1]*len(X1))
```

## Raw ADC trace data (`ssro_h5/SSRO_raw.h5`)

Written by `SSRO_synthetic_CNN.ipynb` (synthetic) or `ssro_raw_waveform.ipynb` (real acquisition):

```python
with h5py.File('ssro_h5/SSRO_raw.h5', 'w') as f:
    f.create_dataset('raw_traces', data=raw_results)  # (2, N_shots, T, 2)
```

Array layout:
- `axis 0` — prepared state
- `axis 1` — shot index
- `axis 2` — time sample index (T samples; at effective 512 MSPS after 8× decimation, 1 µs ≈ 512 samples)
- `axis 3` — [I, Q]

Reading raw traces for CNN input:

```python
with h5py.File('ssro_h5/SSRO_raw.h5', 'r') as f:
    raw = f['raw_traces'][:]   # (2, N_shots, T, 2)

X0_raw = raw[0]   # (N_shots, T, 2)
X1_raw = raw[1]
```

## Synthetic trace generation (from `SSRO_synthetic_CNN.ipynb`)

The simulation anchors to statistics from the real integrated IQ data:

```python
# Parameters extracted from SSRO_0.h5
mu0, mu1    # class centroids in IQ space
cov0, cov1  # per-class IQ covariance matrices

# Free physical parameters
T         = 200    # readout window samples
f_IF      = ...    # intermediate frequency (Hz)
T1        = ...    # qubit relaxation time (µs)
```

Traces are generated as Gaussian noise + oscillating envelope, with optional T₁ relaxation events (the qubit flips mid-readout with probability `dt/T1` per sample). Self-consistency check: integrating synthetic traces should reproduce `SSRO_0.h5` blob statistics within ~5%.

## matplotlib style used across notebooks

```python
plt.rcParams.update({
    'figure.dpi': 120,
    'font.family': 'monospace',
    'axes.spines.top': False,
    'axes.spines.right': False,
})
```
