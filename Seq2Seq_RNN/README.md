# Seq2Seq RNN

Hyperparameter sweep on the Seq2Seq RNN architecture from Cogan Lab (Duke), adapted for 9 class phoneme decoding on the Cogan ECoG dataset.

## Architecture (Cogan et al., 2023)

- **Temporal CNN** extracts local time patterns from raw ECoG
- **Encoder GRU** compresses the entire time window into one summary vector
- **Decoder GRU** generates three phoneme predictions sequentially

Classifies three consecutive phonemes from one continuous recording.

## What I did

I used the existing architecture and ran a 40 run hyperparameter sweep:
2 learning rates × 2 hidden sizes × 10 cross validation folds.

Notebooks in this folder:
- `train.ipynb` and `train_pipeline_simplified.ipynb`: training and sweep loops I built
- `aligned_decoding_train.ipynb` and `aligned_decoding_data_processing.ipynb`: my work inside the Cogan Lab cross patient code path
- `Zac_Model_Result.ipynb`: result analysis and comparison

## Results

| Split | Accuracy range |
|---|---|
| Train | 70 to 100% (memorization) |
| Validation | 17 to 46% (unstable, limited data) |
| Test | 21 to 60% |

The wide variance and large train versus test gap point to overfitting as the main bottleneck on this dataset size.

## Attribution

Architecture and codebase adapted from Cogan Lab's seq2seq decoding work and `cross_patient_speech_decoding` repo. Only the notebooks above are mine; the supporting infrastructure (`nn_models/`, `aligned_decoding/`, etc. in the original repo) belongs to Cogan Lab.

Data files (`.pkl`) are not included for IRB reasons.
