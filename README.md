# Network Intrusion Detection with NSL-KDD

A comparative study of supervised, unsupervised, and rule-based approaches to network intrusion detection, built on the NSL-KDD benchmark dataset.

The project trains and evaluates multiple detection strategies side by side — classical supervised classifiers, unsupervised anomaly detection, and a hand-crafted rule-based scan detector — then compares them on precision, recall, F1, and ROC-AUC to reason about the trade-offs each approach makes for real-world intrusion detection.

## Overview

Intrusion detection systems face a core tension: labelled attack data is often scarce or stale against novel attacks, but purely unsupervised methods tend to underperform on known attack signatures. This notebook explores that tension directly by implementing three different detection paradigms against the same dataset and comparing where each one wins and loses.

| Approach | Method | Labels used? |
|---|---|---|
| Supervised classification | Logistic Regression, Random Forest, Gradient Boosting | Yes |
| Unsupervised anomaly detection | Isolation Forest (trained on normal traffic only) | No |
| Rule-based detection | Threshold rule on connection/service features | No |

## Dataset

**[NSL-KDD](https://www.unb.ca/cic/datasets/nsl.html)** — an improved version of the KDD Cup 1999 dataset with duplicate records removed, widely used as a benchmark for network intrusion detection research. Each record represents a network connection labelled as either normal traffic or one of several attack categories (DoS, probe, R2L, U2R).

## What's covered

**Data Preparation**
- Loading and cleaning NSL-KDD, exploratory analysis of class distribution and feature correlations
- Categorical encoding, train/validation/test splitting with stratification
- Feature scaling with leakage-safe train-only fitting

**Supervised Classification**
- Logistic Regression, Random Forest, and Gradient Boosting classifiers
- Feature importance analysis across tree-based models
- Class imbalance handling via `class_weight="balanced"`

**Unsupervised Anomaly Detection**
- Isolation Forest trained exclusively on normal traffic, evaluated on its ability to flag unseen attack patterns without label supervision

**Rule-Based Scan Detection**
- A hand-crafted threshold rule (`srv_count > 10 AND dst_host_diff_srv_rate > 0.5`) targeting port scan / IP sweep behavior, evaluated on precision/recall like the ML models

**Comparison and Discussion**
- Consolidated precision/recall/F1/AUC comparison across all methods
- Discussion of when to prefer anomaly detection over supervised classification, what each method misses, and how the three approaches could be layered into a single detection pipeline

## Key findings

- Tree-based models (Random Forest, Gradient Boosting) substantially outperform Logistic Regression on recall — critical in intrusion detection, where a missed attack is more costly than a false alarm.
- `src_bytes`, `dst_bytes`, `serror_rate`, and `same_srv_rate` consistently rank as the strongest predictors across both tree-based models, consistent with known DoS/probe traffic signatures.
- Isolation Forest trades recall for the ability to generalize to attack patterns it was never trained on — a meaningful advantage against zero-day-style traffic that supervised models can't achieve.
- Rule-based detection is fast and fully transparent but brittle — effective only against the specific pattern it was designed for.
- The strongest practical design is layered: rule-based detection for known signatures, a supervised classifier for the bulk of traffic, and anomaly detection as a safety net for anything neither has seen before.

## Tech stack

`Python` · `pandas` · `NumPy` · `scikit-learn` · `matplotlib` · `seaborn`

## Running it

```bash
pip install pandas numpy scikit-learn matplotlib seaborn jupyter
jupyter notebook network-intrusion-detection-nslkdd.ipynb
```

The notebook expects the NSL-KDD training and test CSVs (`KDDTrain+.txt` / `KDDTest+.txt`) — available from the [NSL-KDD dataset page](https://www.unb.ca/cic/datasets/nsl.html).

## License

MIT
