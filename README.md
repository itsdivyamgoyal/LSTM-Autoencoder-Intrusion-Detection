# Network Intrusion Detection Using LSTM Autoencoder

This implements an supervised anomaly detection system for detecting network and vehicular intrusions using an LSTM-based autoencoder. The model is trained only on normal traffic and flags anomalies based on how poorly it reconstructs unseen sequences — the idea being that attack traffic looks "unfamiliar" to a model that has only ever seen normal data.

The project was run on Kaggle with GPU acceleration (2x Tesla T4).

---

## What it does

The pipeline works in two phases:

**Phase 1 — Car Hacking Dataset (CAN bus traffic)**
CAN bus messages from a vehicle are loaded, windowed by time, and features like unique message IDs and byte sizes are extracted per window. The model is trained on purely normal windows and tested on windows containing DoS, Fuzzy, Gear spoofing, and RPM spoofing attacks.

**Phase 2 — UNSW-NB15 Dataset (Network traffic)**
A standard network intrusion benchmark. Features like duration, bytes transferred, and packet counts are extracted, and the same autoencoder architecture is applied.

In both cases, a normalized Gaussian likelihood transform is applied to the features before passing them into the LSTM autoencoder. Reconstruction error at test time is used as the anomaly score.

---

## Datasets

The datasets are not included in this repository due to their size. You need to add them manually on Kaggle.

**Car Hacking Dataset**
- Source: https://www.kaggle.com/datasets/pranavjha24/car-hacking-dataset
- Files used: `DoS_dataset.csv`, `Fuzzy_dataset.csv`, `gear_dataset.csv`, `RPM_dataset.csv`
- Total size: ~800 MB

**UNSW-NB15 Dataset**
- Source: https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15
- Files used: `UNSW_NB15_training-set.csv`, `UNSW_NB15_testing-set.csv`
- Total size: ~45 MB

---

## Model Architecture

The model is a sequence-to-sequence LSTM autoencoder:

- Input: window of length 10, 1 feature (normalized likelihood score)
- Encoder: single LSTM layer with 50 hidden units (100 for UNSW)
- Decoder: RepeatVector + LSTM + TimeDistributed Dense
- Loss: Mean Squared Error
- Optimizer: Adam with learning rate scheduling and early stopping

Detection threshold is selected by grid search over 500 candidates, optimized for accuracy, F1, or geometric mean of TPR and TNR.

---

## Results

| Dataset      | Accuracy | Precision | Recall | F1     |
|--------------|----------|-----------|--------|--------|
| Car Hacking  | 98.04%   | 100.00%   | 66.94% | 80.19% |
| UNSW-NB15    | 99.99%   | 99.99%    | 99.99% | 100.00%|

The paper this is based on reports AUC of 0.974 (CAN) and 0.966 (UNSW). Our CAN AUC came out lower (0.75) — likely because the test split ended up with very few pure-attack windows due to how the HCRL dataset mixes normal and attack traffic within the same file.

---

## Requirements

See `requirements.txt` for the full list. The notebook was run with:

- Python 3.10
- TensorFlow 2.19.0
- NumPy 2.0.2
- Pandas 2.3.3

---

## Project Structure

```
.
├── LSTM-Autoencoder.ipynb
├── README.md
└── requirements.txt
```

---

## References

This project is based on the methodology described in:

> "Anomaly-Based Intrusion Detection Using LSTM Autoencoders" — applied to CAN bus and network traffic datasets.

The windowing strategy, likelihood transform (Algorithm 1), and autoencoder design (Algorithm 2) follow the paper's approach with some modifications to the labeling logic to handle mixed-label windows.
