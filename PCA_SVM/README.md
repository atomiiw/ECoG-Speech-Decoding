# PCA + SVM Linear Baseline

A from scratch linear baseline for ECoG phoneme classification.

## Pipeline

1. **PCA** reduces high dimensional ECoG (128 channels × 200 time points) to components capturing most variance
2. **SVM (RBF kernel)** draws nonlinear decision boundaries between phoneme classes
3. Classifies one phoneme at a time from a single neural recording segment

## What I built

The entire pipeline (preprocessing, PCA fitting, SVM training, per patient evaluation) is mine.

## Results

Accuracy across 8 patients × 3 phoneme positions: **7 to 52%** (random baseline: 11%).

Position 2 consistently highest, suggesting the middle phoneme is better aligned to the analysis window.

See `PCA_SVM.ipynb`.
