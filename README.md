# Fraud Detection Case Study

A case-study analysis of transaction-level fraud data, answering two questions: **which
channel most needs fraud-detection attention**, and **which existing data fields are
most useful for a fraud-detection rule or model, and why**.

## Problem Statement

An existing fraud-scoring system (`Risk_Score`) is already in place, but fraud is
still slipping through in some areas. This project investigates:

1. Which transaction channel has the weakest fraud detection today, and where is the
   business exposed to the most missed-fraud dollars?
2. Of the fields already being captured, which ones are actually useful for a rule or
   model — and why, in terms a non-technical stakeholder can act on?

## Approach

- **Channel diagnostic** — per-channel AUC-ROC to isolate a genuine detection-capability
  gap from a simple threshold problem.
- **Cost-based threshold** — the alert cutoff is chosen by minimizing expected dollar
  cost (false alarms vs. missed fraud), not a generic statistical balance point, with a
  sensitivity check across a plausible cost range.
- **Feature selection** — Weight of Evidence / Information Value for each field's
  individual predictive power, followed by a logistic regression with SHAP values to
  check for redundancy and interaction effects.
- **Rule design** — the top features are translated into simple, explainable rules and
  benchmarked on precision, recall, false positive rate, and daily alert volume.
- **Robustness checks** — time-series cross-validation (to rule out a lucky data split)
  and an AUPRC comparison (to check the results hold under a metric more sensitive to
  class imbalance than AUC-ROC).

## Key Findings

- **Channel focus:** POS has the weakest fraud detection of any channel (AUC-ROC
  0.678); Bank Transfer carries the highest missed-fraud dollar exposure at the
  cost-optimal threshold. Both warrant dedicated review.
- **Useful features:** two existing fields — recent failed transaction count and
  transaction location — account for the large majority of predictive power in the
  dataset (94.1% of SHAP importance; every other field scores as not predictive).
- **Model uplift:** a plain logistic regression built only from existing fields reaches
  AUC-ROC 0.818, versus 0.682 for the current system on the same held-out period. A
  model restricted to just the two top fields performs within 0.0001 AUC-ROC of the
  full model — confirming, not just inferring, that they carry almost all of the
  signal.
- **Deployable rule:** flagging transactions with 4+ failed attempts in the past 7 days
  alone catches ~63% of fraud at ~48% precision and a 12% false positive rate — a
  simple, fully transparent rule deployable immediately, ahead of a full model
  rollout.

Full detail, charts, and all supporting calculations are in the notebook.

## Repo Structure

```
fraud_expert_analysis.ipynb   Full analysis notebook (all sections + appendices)
dataset.xlsx                  Transaction-level dataset used in the analysis
README.md                     This file
```

## How to Run

```bash
pip install pandas numpy matplotlib seaborn shap scikit-learn openpyxl
jupyter notebook fraud_expert_analysis.ipynb
```

## Notes & Limitations

- The investigation-cost assumption used for threshold-setting is cited from public
  industry sources, not an audited internal figure; a sensitivity check confirms the
  result is robust across a plausible cost range.
- `Risk_Score`'s internal methodology isn't visible in this dataset — findings describe
  what a transparent model built on existing fields achieves, not what the current
  system does or doesn't already account for internally.
- This is a case-study exercise, not a production system; recommendations are directional,
  not a deployment-ready implementation.
