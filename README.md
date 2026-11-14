# Detecting Rare Attacks in Highly Imbalanced Network Traffic

**Diagnosing minority attack detection in the Edge-IIoTset dataset through feature selection and Macro-F1 analysis of MITM and Fingerprinting attacks**

## Overview

When one type of event is very rare, a model can report high accuracy while missing almost every case that matters. This project looks at that problem in IoT and Industrial IoT (IIoT) network traffic, where some attack types make up less than 1% of the data.

I compared three machine learning models (Random Forest, Support Vector Machine and K-Nearest Neighbours) on their ability to detect the two rarest attack classes, **Man-in-the-Middle (MITM)** and **Fingerprinting**, across 15 traffic classes. I judged them on **Macro-F1** rather than accuracy, because Macro-F1 gives every class equal weight, including the rare ones.

The same challenge sits at the heart of fraud detection and transaction monitoring: suspicious activity is a tiny fraction of all activity, so a model has to be judged on how well it catches the rare cases, not on overall accuracy.

## Data

- **Dataset:** Edge-IIoTset (`ML-EdgeIIoT-dataset.csv` variant), Ferrag et al. (2022)
- **Source:** [Kaggle](https://www.kaggle.com/datasets/mohamedamineferrag/edgeiiotset-cyber-security-dataset-of-iot-iiot)
- **Size:** 157,800 rows, 63 raw columns, 15 classes (normal traffic plus 14 attack types)

The dataset is not included in this repository because of its size. Download it from Kaggle and place it in the same folder as the notebook.

## Method

1. **Data cleaning:** removed duplicate rows and checked for missing and infinite values.
2. **Feature selection:** reviewed columns one by one and removed 19 that had zero variance, were identifiers that would not generalise to new traffic (IP addresses, timestamps, raw payloads), or were dominated by placeholder values. This left 67 features.
3. **Leakage-safe split:** stratified 80/20 train/test split, with **SMOTE** oversampling applied only to the training data to rebalance the rarest classes.
4. **Modelling:** Random Forest, SVM and KNN with a fixed random seed (42) for reproducibility, plus a majority-class baseline for comparison.
5. **Evaluation:** Macro-F1, per-class precision, recall and F1, and PR-AUC for the two minority classes.
6. **Ablation study:** compared the selected 67-feature model against a 68-feature version to measure the real effect of feature selection.
7. **Error analysis:** traced every misclassified Fingerprinting record back to the model's most important features.

## Results

| Model | Macro-F1 | Accuracy |
|---|---|---|
| **Random Forest** | **0.904** | **92%** |
| K-Nearest Neighbours | 0.661 | 68% |
| Support Vector Machine | 0.519 | 54% |
| Baseline (always predicts "Normal") | 0.02 | 15% |

**Minority classes (Random Forest, held-out test set)**

| Class | F1 | PR-AUC |
|---|---|---|
| MITM | 1.00 | 1.00 |
| Fingerprinting | 0.72 | 0.863 |

Full per-class reports are in [`results/model_results.txt`](results/model_results.txt).

## Key findings

- **Rare does not mean hard to detect.** MITM was the rarest class but was detected perfectly, while Fingerprinting remained difficult.
- **The baseline shows why accuracy misleads.** A model that always predicts "Normal" scores 15% accuracy but a Macro-F1 of only 0.02. Random Forest's Macro-F1 was about 45 times higher.
- **Feature selection held up.** The selected 67-feature model performed almost the same as a 68-feature version (0.9039 vs 0.9064). The small difference came from a leakage-adjacent feature, which supports the selected feature set.
- **Errors were explainable.** Most of the 32 misclassified Fingerprinting records were confused with Ransomware. I traced this to the top Random Forest features (`tcp.ack`, `tcp.dstport`, `tcp.seq`), linking model behaviour to practical triage decisions.

## Limitations and next steps

- Default hyperparameters were used, with no grid search.
- SVM and KNN were trained on subsamples (30,000 to 45,000 rows) because of computing limits, which likely understates their performance.
- Results come from a single dataset and may not carry over to other networks.
- Next steps: test ADASYN as an alternative to SMOTE, add cross-validation and hyperparameter tuning, and try more automated feature selection.

## How to run

```bash
pip install -r requirements.txt
```

1. Download `ML-EdgeIIoT-dataset.csv` from Kaggle into this folder.
2. Open `edge_iiot_minority_attack_detection.ipynb` in Jupyter or VS Code.
3. Restart the kernel and run all cells in order. The notebook creates its own intermediate cleaned file as it runs.

**Tools:** Python, pandas, NumPy, scikit-learn, imbalanced-learn, Matplotlib, Seaborn

## About

Completed as part of my MSc in Cybersecurity at the National College of Ireland (module: AI and Machine Learning in Cybersecurity, 2026).

**Ahmed Lawal** | Financial crime investigator moving into AML, fraud analytics and transaction monitoring | [LinkedIn](https://www.linkedin.com/in/lawaloahmed/)
