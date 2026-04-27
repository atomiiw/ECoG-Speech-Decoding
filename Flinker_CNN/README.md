# Flinker Spatiotemporal CNN Fine Tuning

Adapted the spatiotemporal CNN from Flinker Lab's neural speech decoding framework to 9 class phoneme classification on Cogan Lab ECoG data.

## Architecture (Flinker et al., 2024)

- **3D CNN + Residual Blocks** extracts spatiotemporal features across the electrode grid and time
- **Projection** produces a compact latent representation

Originally trained for brain to speech reconstruction. I repurposed the encoder for phoneme classification.

## What I did

This was a **cross dataset transfer** experiment:

1. **Stage 1 (encoder pretraining on Flinker data):** Trained Flinker's full pipeline on patient HB02 from their HUPA dataset.
   - `local_training.ipynb`: stage 1 (a2a, audio to audio, 60 epochs) and stage 2 (e2a, ECoG to audio, 60 epochs) on local GPU
   - `colab_training.ipynb`: same pipeline on Colab A100
   - Output: pretrained encoder weights `model_epoch59.pth` (or `model_epoch90_HB02.pth`)

2. **Stage 2 (transfer to Cogan data for phoneme classification):** Froze the pretrained encoder, used it as a feature extractor on Cogan Lab Subject 14, fit classical ML heads on top.
   - `ecog_decoder_finetune.ipynb`: 4 method comparison (raw ECoG vs encoder features × SVM vs LDA)

## Results

Tested on Cogan Subject 14 (144 trials, 9 phoneme classes, ~57 train / 58 val / 29 test). Encoder pretrained on Flinker HB02. Random baseline 11.1%.

| Method | CV Score | Train Acc | Val Acc |
|---|---|---|---|
| Raw ECoG + PCA + SVM | 17.73% | 36.84% | 12.07% |
| Raw ECoG + PCA + LDA | 10.61% | 42.11% | 13.79% |
| Encoder + PCA + SVM | 26.82% | 71.93% | 24.14% |
| **Encoder + PCA + LDA** | 23.33% | 42.11% | **27.59%** |

**Key finding:** even with cross dataset transfer (Flinker HB02 encoder applied to Cogan S14 phonemes), Flinker's pretrained encoder approximately doubles validation accuracy over raw ECoG features (12 to 14% → 24 to 28%). The encoder features carry generalizable structure across patients and recording paradigms. Best configuration is Encoder + PCA + LDA at 27.59% val acc.

There is no "Flinker model on Flinker data for phoneme classification" number to report. Flinker's original paper evaluates audio reconstruction quality, not phoneme classification. The training notebooks above only produced encoder weights; their training loss curves are not directly comparable to the classification accuracies above.

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
