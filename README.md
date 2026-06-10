# Part 3 — Churn Prediction Model & Model Card

## Overview
Binary churn classification using XGBoost (with Logistic Regression baseline). Includes threshold tuning, error analysis, feature importance, and a structured model card.

**Snapshot date:** `2025-09-30` | **Target:** `churn_next_60d` | **Split:** pre-assigned train/validation/test

---

## Repository Structure

```
├── README.md
├── churn_model.ipynb       ← Main modeling notebook (run in Google Colab)
├── model.pkl               ← Saved XGBoost model (generated after running notebook)
├── scaler.pkl              ← StandardScaler for Logistic Regression (generated)
├── feature_cols.pkl        ← Feature column list (generated)
├── metrics.json            ← Model performance metrics (generated)
├── error_analysis.md       ← FP/FN analysis with 10 customer examples
├── model_card.md           ← Structured model card for stakeholders
├── requirements.txt
```

---

## How to Run

1. Open [Google Colab](https://colab.research.google.com)
2. Upload `churn_model.ipynb`
3. Update `DATA_DIR` in cell 2:
   ```python
   DATA_DIR = '/content/drive/MyDrive/d2c_churn_data/'
   ```
4. **Runtime → Run all**
5. Download generated files (`model.pkl`, `metrics.json`, etc.) and commit to this repo

---

## Model Summary

| Component | Detail |
|---|---|
| Baseline | Logistic Regression (class_weight='balanced') |
| Final model | XGBoost (300 estimators, max_depth=5, lr=0.05) |
| Features | 25+ features from `rfm_modeling_snapshot.csv` (leakage-free) |
| Threshold | Tuned for recall ≥ 70% (cost-asymmetric: FN >> FP) |
| Evaluation | ROC-AUC, PR-AUC, F1, Precision, Recall, Confusion Matrix |

---

## Loading the Model

```python
import joblib
model = joblib.load('model.pkl')
feature_cols = joblib.load('feature_cols.pkl')
# model.predict_proba(X[feature_cols])[:, 1]
```

---

## Dataset
Uses `rfm_modeling_snapshot.csv` from: https://drive.google.com/drive/folders/1PmLapJI1VSDgvl_AxARNKwM1MCd3WFX0?usp=sharing
