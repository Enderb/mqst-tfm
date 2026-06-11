# IQ-plane classifier pipeline

## Data flow

```
SSRO acquisition → ssro_h5/SSRO_0.h5 → load (2000 shots × 2 classes × [I,Q])
  → train/test split (stratified, typically 80/20)
  → optional: StandardScaler (mean=0, std=1 per feature)
  → fit classifier
  → evaluate: assignment fidelity F, confusion matrix, decision boundary plot
```

## Classifiers evaluated in this project

All from `sklearn` except the PyTorch CNN:

| Classifier | Class | Notes |
|---|---|---|
| LDA (baseline) | `LinearDiscriminantAnalysis` | Optimal for equal-covariance Gaussians; 2 DSP slices on FPGA |
| QDA | `QuadraticDiscriminantAnalysis` | Handles unequal covariances; 4 multiply-accumulates |
| MLP | `MLPClassifier(hidden_layer_sizes=(H,), activation='relu')` | Grid-searched over H ∈ {4,8,16,32} |
| 1D CNN | PyTorch `Conv1d` on raw traces | Requires `SSRO_raw.h5`; see `SSRO_synthetic_CNN.ipynb` |

**Finding**: No MLP architecture improves on LDA when the input is a single integrated (I, Q) pair per shot. The IQ blobs are well-described by a linear Gaussian model. CNNs on raw traces can potentially improve fidelity by exploiting time-domain structure (T₁ relaxation events).

## Assignment fidelity

```python
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_test, y_pred)
p10 = cm[0,1] / cm[0].sum()   # P(predict 1 | true 0)
p01 = cm[1,0] / cm[1].sum()   # P(predict 0 | true 1)
F   = 1 - 0.5*(p10 + p01)
print(f"Assignment fidelity: {F:.4f}")
```

## Decision boundary visualisation

```python
# meshgrid over IQ range
xx, yy = np.meshgrid(np.linspace(I_min, I_max, 300),
                     np.linspace(Q_min, Q_max, 300))
Z = clf.predict(np.c_[xx.ravel(), yy.ravel()]).reshape(xx.shape)
plt.contourf(xx, yy, Z, alpha=0.3, cmap='RdBu')
plt.scatter(X0[:,0], X0[:,1], s=2, label='|0⟩')
plt.scatter(X1[:,0], X1[:,1], s=2, label='|1⟩')
plt.savefig('figures/decision_boundaries.pdf', bbox_inches='tight')
```

## FPGA deployment path (planned)

1. Train classifier on laptop using `sklearn` / PyTorch.
2. Quantise weights to fixed-point (16-bit or 8-bit) compatible with ZCU216 DSP48 slices.
3. For LDA: a single dot-product and threshold → write coefficients into tProcessor data memory.
4. For MLP: use `hls4ml` to synthesise the network as an HLS IP block (see `SSRO_readout_ML.ipynb` §12).
5. Integrate the IP block so the tProcessor receives a binary `b ∈ {0,1}` within one clock cycle.

## Architecture grid search pattern

```python
from sklearn.model_selection import StratifiedKFold

results = {}
for hidden in [4, 8, 16, 32]:
    clf = MLPClassifier(hidden_layer_sizes=(hidden,), max_iter=1000)
    cv  = StratifiedKFold(n_splits=5)
    scores = cross_val_score(clf, X_scaled, y, cv=cv, scoring='accuracy')
    results[hidden] = scores.mean()
```
