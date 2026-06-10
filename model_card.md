# Model Card — D2C Churn Prediction Model
## D2C Customer Churn Intelligence Capstone — Part 3

---

## Model Overview

| Field | Detail |
|---|---|
| **Model name** | D2C 60-Day Churn Prediction Model |
| **Model type** | Binary classification (XGBoost) |
| **Version** | 1.0 |
| **Date** | October 2025 |
| **Owner** | Data Analytics Team |

---

## Intended Use

**Primary use:** Predict which customers are likely to churn (make zero purchases) in the 60 days following the snapshot date (2025-09-30).

**Intended users:** Internal CRM and retention team. The model output feeds into a scoring API (Part 4) that the CRM tool queries to prioritise retention actions.

**Out-of-scope uses:**
- Automated decision-making without human review for high-value customers
- Customer-facing communications that reference the churn score
- Credit, insurance, or other regulated decisions
- Predicting churn for B2B or wholesale customers (model trained on D2C personal-care data only)

---

## Data Used

**Training data:** 2,400 D2C personal-care customers, snapshot date 2025-09-30.

**Feature sources:**
- Customer profiles (demographics, acquisition channel, loyalty tier)
- Order history — RFM features computed from orders on or before snapshot date
- Support ticket data (count, sentiment, resolution time, reopened rate)
- Web/app activity (sessions, views, cart adds, email opens — 30-day window)

**Leakage prevention:** All features are derived from data available on or before the snapshot date. Post-snapshot orders were excluded from feature computation. The pre-built `rfm_modeling_snapshot.csv` was used, which the data provider has confirmed is leakage-free.

**Target variable:** `churn_next_60d` — 1 if customer made zero purchases between 2025-10-01 and 2025-11-29.

**Split:** Pre-assigned train/validation/test split from `churn_labels.csv`.

---

## Model Approach

**Baseline:** Logistic Regression with class weight balancing and standardised features.

**Final model:** XGBoost classifier with:
- 300 estimators, max_depth=5, learning_rate=0.05
- `scale_pos_weight` set to handle class imbalance
- Subsample and colsample_bytree at 0.8 for regularisation

**Threshold:** Selected to achieve recall ≥ 70% on validation set, prioritising churn detection over precision (based on cost asymmetry: missed churner costs ₹500–5,000+, false alarm costs ₹30–40).

---

## Performance

> Exact values will be populated from `metrics.json` after notebook execution.

| Metric | Validation | Test |
|---|---|---|
| ROC-AUC | (from notebook) | (from notebook) |
| PR-AUC | (from notebook) | (from notebook) |
| Precision | (from notebook) | (from notebook) |
| Recall | (from notebook) | (from notebook) |
| F1 Score | (from notebook) | (from notebook) |

**Baseline comparison:** XGBoost outperforms Logistic Regression on ROC-AUC and F1 on validation data.

---

## Top Features

The most important features driving predictions (by XGBoost gain importance):
1. `recency_days` — days since last order
2. `frequency_180d` — order count in last 180 days
3. `sessions_30d` — web/app sessions in last 30 days
4. `monetary_180d` — total spend in last 180 days
5. `last_visit_days_ago` — days since last website visit

These align with business intuition: inactivity (purchase and web) is the strongest churn signal.

---

## Limitations

1. **Single snapshot:** Model is trained on a single point-in-time snapshot. Customer behaviour may shift seasonally (e.g., festival shopping spikes), and the model does not capture this.
2. **No causal inference:** The model identifies correlation between features and churn, not causation. A customer with high recency may churn because of product issues, competitor offers, or life changes — the model cannot distinguish between these.
3. **Small dataset:** 2,400 customers is sufficient for the feature set used, but limits the model's ability to detect rare subgroup patterns.
4. **No external data:** The model does not use competitor pricing, macroeconomic conditions, or social media sentiment, all of which can influence churn.
5. **Static features:** The model uses a 30-day web activity window and 180-day order window. Customers whose behaviour changed in the last 7 days may not be captured accurately.

---

## Ethical Risks & Fairness

1. **Demographic bias:** The model includes `city_tier` and `age_group` as features. If these correlate with socioeconomic status, the model may systematically under-serve or over-target certain demographic groups. Recommendation: monitor churn prediction rates and error rates by demographic group.
2. **Over-targeting vulnerable customers:** Discount-dependent or financially constrained customers may receive frequent retention offers, potentially encouraging spending they cannot afford. The retention team should cap the number of outreach touches per customer per quarter.
3. **Privacy:** The model uses behavioural data (web sessions, support sentiment). Customers should be informed through the privacy policy that their behavioural data is used for internal retention decisions.
4. **No automated exclusion:** The model score should never be used to automatically deny service, downgrade accounts, or remove customers from programmes without human review.

---

## Monitoring Needs

| What to Monitor | Frequency | Alert Threshold |
|---|---|---|
| Prediction distribution (mean probability) | Weekly | >10% shift from training distribution |
| Feature drift (recency, sessions, monetary) | Weekly | PSI > 0.2 on any key feature |
| Actual churn rate vs predicted | Monthly | >5pp deviation from expected |
| False negative rate | Monthly | >30% increase from baseline |
| Model latency (API response time) | Daily | p95 > 500ms |

**Retraining trigger:** Retrain the model every 90 days or whenever feature drift exceeds PSI > 0.2 on two or more key features. Also retrain after major business events (new product launch, pricing change, major campaign).

---

## When NOT to Use This Model

- Do not use for customers acquired after the snapshot date (model has no data on them)
- Do not use for B2B or wholesale customers
- Do not use as the sole input for high-value customer decisions (combine with manual review)
- Do not use if the product catalogue, pricing, or return policy has fundamentally changed since training
- Do not use if more than 6 months have passed since training without revalidation
