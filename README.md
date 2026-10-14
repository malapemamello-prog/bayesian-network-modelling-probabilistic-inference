# Bayesian Network Modelling and Probabilistic Inference for Heart Disease Prediction



---

## Overview

This project builds and compares two Bayesian networks (BNs) for binary heart-disease prediction on the Cleveland dataset: one **expert-designed** DAG and one **data-driven** DAG learned with `bnlearn`. The workflow covers data preparation, structure learning, parameter learning, probabilistic inference and held-out evaluation.

**Key result:** the expert DAG outperformed the data-driven DAG and a majority-class baseline on the held-out test set (accuracy 90.2%, ROC-AUC 0.961), so it was kept as the preferred model. This applies to this dataset and split only, and the model is not a clinically validated diagnostic tool.

---

## Data

| Item | Detail |
|---|---|
| Source | Cleveland heart-disease dataset (UCI Machine Learning Repository) |
| Size | 303 patients, 13 predictors |
| Target | Original 0 to 4 collapsed to binary: 0 = no disease, 1 = disease present |
| Class balance | 164 no disease (54.1%), 139 disease (45.9%) |
| Missing values | `ca` (4) and `thal` (2), filled with training-set modes (`ca`=0, `thal`=3) |
| Split | 242 train / 61 test, stratified |
| Discretisation | `age`, `trestbps`, `chol`, `thalach`, `oldpeak` into Low/Medium/High using training-set quantiles |

Splitting happened **before** imputation and discretisation, so nothing from the test set leaked into preprocessing.

---

## Methods

| Step | Approach |
|---|---|
| Expert DAG | 16 edges; four direct parents of the target (`age_cat`, `gender`, `trestbp_cat`, `chol_cat`), with diagnostic-evidence and exercise-response pathways |
| Data-driven DAG | Hill Climbing with BIC score, learned on training data; 11 edges, one direct target parent (`ca`) |
| Parameter learning | Bayesian estimation with BDeu smoothing, training data only |
| Inference | Evidence grid of 5,184 profiles per network, a one-variable sensitivity analysis on `ca`, and reverse inference with disease fixed to 1 |
| Evaluation | Accuracy, balanced accuracy, precision, recall, specificity, F1, ROC-AUC and PR-AUC against a majority-class baseline |

Arrows represent a probabilistic factorisation, not established medical causality.

---

## Results

### Structural comparison

| Comparison | Count |
|---|---|
| Expert edges | 16 |
| Learned edges | 11 |
| Common directed edges | 3 |
| Common adjacencies (direction ignored) | 5 |
| Reversed relationships | 2 |

### Inference

| Statistic | Expert DAG | Learned DAG |
|---|---|---|
| Profiles resolved | 5,184 | 5,184 |
| Max P(disease = 1) | 0.793 | 0.670 |
| Min P(disease = 1) | 0.192 | 0.307 |
| Posterior range | 0.601 | 0.363 |

- Mean absolute difference between the models' posteriors: **0.056**; Spearman rank correlation: **0.766**.
- Both networks rank profiles with `cp=4`, `thal=7` and `ca=1` highest.
- Moving `ca` from 0 to 1 gave the largest shift (expert 0.708 to 0.793; learned 0.561 to 0.670).

### Held-out performance (61 test cases)

| Model | Accuracy | Bal. Acc. | Precision | Recall | Specificity | F1 | ROC-AUC | PR-AUC |
|---|---|---|---|---|---|---|---|---|
| Majority baseline | 0.541 | 0.500 | 0.000 | 0.000 | 1.000 | 0.000 | 0.500 | 0.459 |
| **Expert DAG** | **0.902** | **0.901** | **0.893** | **0.893** | **0.909** | **0.893** | **0.961** | **0.953** |
| Learned DAG | 0.853 | 0.850 | 0.852 | 0.821 | 0.879 | 0.836 | 0.915 | 0.909 |

| Confusion matrix | TN | FP | FN | TP |
|---|---|---|---|---|
| Majority baseline | 33 | 0 | 28 | 0 |
| Expert DAG | 30 | 3 | 3 | 25 |
| Learned DAG | 29 | 4 | 5 | 23 |

### Performance vs interpretability

The expert DAG won on both counts: it scored best and its edges are justified. Some learned edges (for example `thal → gender` and `thalach_cat → age_cat`) are hard to defend as domain structure, so they are treated as empirical dependencies only.

---

## Limitations

| Limitation | Implication |
|---|---|
| Small sample (303 total, 61 test) | Results may be sensitive to the particular split |
| Three-state discretisation | Loses within-bin information |
| Simple mode imputation | More sophisticated missing-data methods were not explored |
| Expert DAG not clinically reviewed | Cannot be read as a validated causal graph |
| Hill Climbing can hit local optima | Near-equivalent structures may exist |
| No calibration assessment | Only discrimination and classification were evaluated |

**Recommended next steps:** repeated cross-validation or bootstrapping, calibration checks, clinical review of expert edges, sensitivity to discretisation and the BDeu prior, more transparent baselines, and external validation before any operational use.

---

## Contributors

| Name |
|---|
| Reatile Seekoei |
| Mamello Malape |


---

## References

- Detrano, R. et al. (1989). International application of a new probability algorithm for the diagnosis of coronary artery disease. *The American Journal of Cardiology*, 64(5), 304-310.
- Heckerman, D. (1995). *A Tutorial on Learning with Bayesian Networks*. Microsoft Research, MSR-TR-95-06.
- Koller, D. & Friedman, N. (2009). *Probabilistic Graphical Models*. MIT Press.
- Schwarz, G. (1978). Estimating the dimension of a model. *The Annals of Statistics*, 6(2), 461-464.
- Taskesen, E. (2024). Learning Bayesian Networks with the bnlearn Python package (v0.9.0).
- UCI Machine Learning Repository (1988). Heart Disease dataset (Cleveland). doi: 10.24432/C52P4X.
