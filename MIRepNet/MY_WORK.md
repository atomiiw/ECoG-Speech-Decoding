# MIRepNet Fine Tuning (my contribution)

The `README.md` in this folder is the original MIRepNet repository README by Wu et al., 2025. This file documents my contribution.

## What I did

Adapted MIRepNet (originally trained for 4 class motor imagery classification) to 9 class phoneme decoding on the Cogan Lab ECoG dataset.

- Froze all conv and transformer layers
- Replaced the 4 class motor imagery head with a new 9 class phoneme classification head
- Fine tuned only the new head on Cogan Lab data
- Evaluated cross patient and same patient generalization

My notebooks:
- `MIRepNet_Finetuning.ipynb`: fine tuning pipeline
- `produce_dataset.ipynb`: dataset preparation for the Cogan ECoG format

## Results

| Setup | Accuracy |
|---|---|
| Pretrained model directly (no fine tuning) | 8.33% (random ≈ 11%) |
| Train across all 8 patients, test on held out patients (cross patient) | ~12% (chance level) |
| Train on one patient, test on training trials (memorization check) | 100% |
| Train on one patient, test on unseen trials from same patient | **55%** |

**Interpretation:** the foundation model generalizes within a patient given enough fine tuning, but does not transfer cross patient on this dataset. Overfitting limits the same patient performance ceiling.

## Attribution

Everything in this folder except the two notebooks above (`MIRepNet_Finetuning.ipynb`, `produce_dataset.ipynb`) is from the original MIRepNet repository ([staraink/MIRepNet](https://github.com/staraink/MIRepNet/tree/main)) by Wu et al.

Paper: Wu et al., "MIRepNet: An EEG Foundation Model for Motor Imagery", arXiv 2507.20254 (July 2025).
Pretrained weights (`weight/MIRepNet.pth`) are theirs.

Data files (`.pkl`, `.npy`) for the Cogan ECoG dataset are not included for IRB reasons.
