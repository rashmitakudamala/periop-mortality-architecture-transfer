# Perioperative Mortality Architecture-Transfer Framework

## Overview

This repository contains the analysis code for a study testing whether a
domain-structured ensemble architecture, previously validated for
postoperative delirium prediction, transports to 30-day perioperative
mortality prediction with discrimination statistically indistinguishable
from a monolithic model trained on an identical feature set. Predictors
are organized into three domains (patient-related, surgery-related, and
anesthetic-related), domain-specific LightGBM base learners are trained
with patient-level out-of-fold prediction, and their outputs are combined
through a standardized, logit-transformed logistic regression
meta-learner. The study also quantifies each domain's standalone
contribution to mortality discrimination and compares the resulting
domain hierarchy against the surgery-dominant hierarchy previously
reported for postoperative delirium using the same architecture.

No patient-level data is included in this repository, and none can be —
both notebooks expect a local data file (governed by an institutional data
use agreement) that is not and will not be part of this repo. They will
not run without that file present locally.

## Repository Structure

```
├── Notebooks/
│ ├── 01_single_architecture_comparator.ipynb (#Background work, not used in the manuscript)
│ └── 02_domain_ensemble_metalearner.ipynb
├── LICENSE
└── README.md
```

Aggregate tables and figures (cohort characteristics, performance metrics,
SHAP importance, calibration and risk-surface plots) are produced by
running the notebooks; they are not checked into this repository. One file
produced during execution must never be committed under any circumstances:
`test_predictions_2020.parquet` contains real per-patient study identifiers
linked to each patient's actual outcome and individual predicted risk
score. The included `.gitignore` excludes it automatically.

## Notebooks

### 01 - Single-Architecture Comparator (background work, not used in the current manuscript)

- Data loading, deduplication audit, and cohort construction
- Feature engineering on a separate, earlier feature extract (88 predictors,
  no domain stratification)
- Missing-data reporting (pre-imputation) and case-mix drift assessment
- LightGBM training, Platt calibration, and bootstrap confidence intervals
- SHAP feature importance, decision curve analysis, subgroup discrimination
- Stratified performance by clinical vulnerability decile, with a
  floor-model (ASA + age + Charlson) comparison and net reclassification
  improvement by component

An earlier draft of the manuscript compared this model against the
domain-stratified ensemble in notebook 02 as a proxy for a prior
single-architecture baseline. That comparison was removed from the current
manuscript because the two models were trained on different source files
with non-overlapping feature sets, making the comparison invalid. This
notebook is retained for provenance but does not feed any table or figure
in the current manuscript.

Requires a local `df_model_1.csv`.

### 02 - Domain Ensemble and Meta-Learner (primary analysis)

- Domain definition (patient-related, surgery-related, anesthetic-related)
- Duration-normalized hemodynamic burden variable construction
- Domain-specific LightGBM base learners with patient-level out-of-fold
  prediction
- Logistic regression meta-learner (L2 regularization) on standardized,
  logit-transformed domain probabilities
- Head-to-head comparison against a monolithic full-feature model trained
  on the identical, union-of-domains predictor set, with a paired
  bootstrap test on the AUROC/PR-AUC difference
- SHAP interpretability by domain, used to identify each domain's
  highest-contributing predictors
- Vulnerability-stratified hemodynamic risk-surface heatmaps and
  subgroup/fairness analysis (exploratory; not part of the manuscript's
  primary reported comparisons)

Produces Tables 1 and 2 of the manuscript. Requires a local
`Standardized_data_all_4_20_26.xlsx`.

## Modelling Approach

Both notebooks use patient-level grouped five-fold cross-validation and a
temporal train/validation/test split (2010-2018 / 2019 / 2020) to prevent
data leakage and assess prospective generalizability. The domain-ensemble
meta-learner is trained on out-of-fold domain predictions generated within
the training set, with domain models refit on the full training set for
validation/test inference, preventing the leakage that would otherwise
arise from training the meta-learner on in-sample domain predictions. The
primary architecture comparison uses a paired bootstrap (same
patient-level resample scores both models each iteration) rather than
separately-estimated confidence intervals.

## Requirements

- Python 3.9+
- scikit-learn
- LightGBM
- SHAP
- pandas
- numpy
- statsmodels
- matplotlib

Install dependencies:

```pip install scikit-learn lightgbm shap pandas numpy statsmodels matplotlib```


## Data Availability

This study used a single-institution perioperative registry. Due to data
use agreements and patient privacy protections, the raw data cannot be
shared publicly.

## Citation

If you use this framework, please cite the accompanying manuscript and the
domain-structured ensemble architecture this work builds on:

* Kudamala R, Barboi C. Architecture transfer of a domain-structured
ensemble for perioperative mortality prediction. Manuscript submitted for
publication, AMIA Amplify 2027.

* Shukla S, Barboi C. A domain-structured ensemble framework for
perioperative outcome prediction using electronic health record data. 2026
IEEE 14th International Conference on Healthcare Informatics (ICHI).
Minneapolis, MN, USA; 2026:377-385. doi:10.1109/ICHI69079.2026.00056.
