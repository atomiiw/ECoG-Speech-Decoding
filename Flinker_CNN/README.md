# Flinker Spatiotemporal CNN Fine Tuning

Adapted the spatiotemporal CNN from Flinker Lab's neural speech decoding framework to 9 class phoneme classification on Cogan Lab ECoG data.

## Architecture (Flinker et al., 2024)

- **3D CNN + Residual Blocks** extracts spatiotemporal features across the electrode grid and time
- **Projection** produces a compact latent representation

Originally trained for brain to speech reconstruction. I repurposed the encoder for phoneme classification.

## What I did

- Froze the entire encoder
- Fine tuned a new classification head only
- Trained on Cogan Lab phoneme data (different paradigm and electrode layout than the original training data)

My notebooks in this folder:
- `ecog_decoder_finetune.ipynb`: feature extraction + classification head comparison
- `local_training.ipynb`: local training experiments (a2a, e2a configs)
- `colab_training.ipynb`: Colab training runs

## Results

Tested on patient HB02 (144 trials, 9 phoneme classes, 57 train / 58 val / 29 test, 64 electrodes after subsampling). Random baseline 11.1%.

| Method | CV Score | Train Acc | Val Acc |
|---|---|---|---|
| Raw ECoG + PCA + SVM | 17.73% | 36.84% | 12.07% |
| Raw ECoG + PCA + LDA | 10.61% | 42.11% | 13.79% |
| Encoder + PCA + SVM | 26.82% | 71.93% | 24.14% |
| **Encoder + PCA + LDA** | 23.33% | 42.11% | **27.59%** |

**Key finding:** Flinker's pretrained encoder approximately doubles validation accuracy over raw ECoG features (12 to 14% → 24 to 28%). Best configuration is Encoder + PCA + LDA at 27.59% val acc.

## Attribution

Architecture, training infrastructure, and pretrained weights are from the official Flinker Lab repository, [flinkerlab/neural_speech_decoding](https://github.com/flinkerlab/neural_speech_decoding).

Paper: Chen et al., "A neural speech decoding framework leveraging deep learning and speech synthesis", *Nature Machine Intelligence* (2024). DOI: 10.1038/s42256-024-00824-8.

To reproduce, clone the original repo and place these notebooks in its root directory:

```bash
git clone https://github.com/flinkerlab/neural_speech_decoding
cd neural_speech_decoding
# copy the .ipynb files from this folder here
```

Data files (`.pkl`) for the Cogan ECoG dataset are not included for IRB reasons.
