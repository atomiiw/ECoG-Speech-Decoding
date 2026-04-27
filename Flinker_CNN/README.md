# Flinker Spatiotemporal CNN Fine Tuning

Adapted the spatiotemporal CNN from Flinker Lab's neural speech decoding framework to 9 class phoneme classification on Cogan Lab ECoG data.

## Architecture (Flinker et al., 2024)

- **3D CNN + Residual Blocks** extracts spatiotemporal features across the electrode grid and time
- **Projection** produces a compact latent representation

Originally trained for brain to speech reconstruction. I repurposed the encoder for phoneme classification.

## What I did

The core question: **does the spatiotemporal CNN encoder from Flinker learn representations that transfer to a different dataset and task?**

To answer it I ran a cross dataset transfer experiment in two stages:

1. **Stage 1 (encoder pretraining on Flinker HB02):** Trained Flinker's full pipeline on patient HB02 from their HUPA dataset.
   - `local_training.ipynb`: stage 1 (a2a, audio to audio, 60 epochs) and stage 2 (e2a, ECoG to audio, 60 epochs) on local GPU
   - `colab_training.ipynb`: same pipeline on Colab A100
   - Output: pretrained encoder weights `model_epoch59.pth` (a.k.a. `model_epoch90_HB02.pth`)

2. **Stage 2 (transferability test on Cogan S14 phonemes):** Froze the encoder. Compared two feature inputs (encoder output vs raw ECoG) under the same classification head, to isolate the contribution of the CNN representation.
   - `ecog_decoder_finetune.ipynb`: 2x2 design (Raw / Encoder × PCA + SVM / PCA + LDA)

**Note on the head:** I originally defined a small NN classifier head (`SimplePhonemeClassifier`, 128 hidden + dropout) but did not implement the training loop in the end. I switched to PCA + SVM / LDA as the head because the dataset is small (144 trials) and a NN head would likely overfit before producing a useful comparison. The CNN encoder is what is being evaluated; the head is held the same across raw and encoder conditions to keep the comparison fair.

## Results

Tested on Cogan Subject 14 (144 trials, 9 phoneme classes, ~57 train / 58 val / 29 test). Encoder pretrained on Flinker HB02. Random baseline 11.1%.

| Input | Head | CV Score | Train Acc | Val Acc |
|---|---|---|---|---|
| Raw ECoG | PCA + SVM | 17.73% | 36.84% | 12.07% |
| Raw ECoG | PCA + LDA | 10.61% | 42.11% | 13.79% |
| **CNN encoder features** | PCA + SVM | 26.82% | 71.93% | 24.14% |
| **CNN encoder features** | PCA + LDA | 23.33% | 42.11% | **27.59%** |

**Key finding:** holding the classification head fixed, switching the input from raw ECoG to CNN encoder features approximately **doubles validation accuracy** (12 to 14% → 24 to 28%). This isolates the encoder as the source of the gain. The CNN learns structure that transfers across patients and across datasets (HB02 → S14), even though the original encoder was trained for audio reconstruction, not phoneme classification.

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
