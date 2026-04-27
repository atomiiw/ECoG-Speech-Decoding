# ECoG Speech Decoding

Independent study at **Pearson Lab, Duke University** (Junior Fall 2025), advised by Prof. John Pearson.

I compared 4 model architectures for decoding speech from electrocorticography (ECoG) signals into phoneme sequences. The work was a survey of the current ECoG decoding landscape, identifying overfitting and cross patient generalization as the dominant bottlenecks.

## Poster

![Towards Unconstrained Speech Decoding from ECoG](poster/poster.png)

[Download PDF](poster/Towards_Unconstrained_Speech_Decoding_from_ECoG.pdf) · presented at Duke Independent Study Poster Session, December 2025

## Dataset

Cogan Lab (Duke) micro ECoG recordings during production of nonsense syllables.
8 patients, 144 trials per patient, 3 phonemes per trial, 9 possible phonemes, 200 time points (±500 ms around phoneme onset).

The patient data files (`.pkl`, `.npy`) are not included in this repo for IRB and privacy reasons.

## Models compared

| Model | What I did | Result (random baseline 11%) |
|---|---|---|
| **PCA + SVM** (linear baseline) | Built and trained the entire pipeline from scratch | Best per patient × position: **52% acc**. Range **7 to 52%** across 8 patients × 3 phoneme positions. Middle phoneme position consistently highest, suggesting better alignment to the analysis window. |
| **Seq2Seq RNN** (Cogan et al., 2023) | Used the existing architecture, ran a 40 run hyperparameter sweep (2 learning rates × 2 hidden sizes × 10 folds) | Best test acc **60%**, test range **21 to 60%**. Train acc reaches **100%** (heavy memorization), val 17 to 46%. The large train to test gap shows overfitting and limited data, not architecture, are the bottleneck. |
| **MIRepNet** (Wu et al., 2025) | Froze conv and transformer layers, fine tuned a new 9 class classification head | Same patient unseen trials: **55% acc**. Cross patient (train on 7, test on 1): **~12%**, equal to chance. The foundation model transfers within a patient but collapses across patients on this dataset. |
| **Spatiotemporal CNN** (Flinker et al., 2024) | (1) Trained the encoder on Flinker's HB02 patient by running their full pipeline as published (a2a + e2a, 60 epochs each). (2) Removed Flinker's audio synthesis decoder. (3) Replaced it with a phoneme classification head (PCA + LDA / SVM). (4) Froze the encoder, applied the new head to Cogan S14 phoneme data. | Best val acc **27.59%** (Encoder + PCA + LDA) vs **12 to 14%** with raw ECoG under the same head. **CNN encoder features ~2x raw**, isolating the encoder as the source of the gain. Confirms the CNN learns structure that transfers across patients and datasets. |

## Repo layout

```
.
├── poster/                    Independent study poster
├── data_exploration/          Cogan Lab .pkl exploration and data prep notebooks
├── PCA_SVM/                   From scratch linear baseline
├── Seq2Seq_RNN/               Hyperparameter sweep on Cogan 2023 architecture
├── MIRepNet/                  Fine tuning Wu 2025 EEG foundation model
└── Flinker_CNN/               Fine tuning Flinker 2024 spatiotemporal CNN
```

Each folder has its own README explaining what is mine versus what is third party, and pointing to the original source repository for the third party code.

## Findings

The highest reported accuracies in the literature (86% word accuracy in Willett 2023, 68 to 90% in Chang 2025) rely on privileged conditions: thousands of repeated sentences from one participant, stable electrodes over months, limited vocabularies, strong language model correction. In less constrained settings, accuracy drops to 30 to 50% phoneme accuracy and 10 to 58% word error rate. **High performance in unconstrained, real world scenarios remains unsolved.**

My results on the Cogan Lab dataset (a less constrained setting) confirm this. Even foundation model fine tuning (MIRepNet) only reaches 55% on same patient unseen trials and collapses to chance level (12%) cross patient.

## Next directions

- Fine tune a Mamba based model for single phoneme neural speech decoding (Mamba has not been applied to neural speech decoding)
- Switch to the Stavisky Lab (Stanford) dataset with 15 months of recordings from one patient
- Build and train a hybrid CNN + Mamba architecture
- Extend to continuous speech decoding on minutes long recordings with automatic speech detection

## About me

[Atom Wang (Maidou Wang)](https://github.com/atomiiw), BSE Electrical and Computer Engineering at Duke (2023 to 2027).
