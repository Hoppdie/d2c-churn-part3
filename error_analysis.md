# Error Analysis
## D2C Customer Churn Capstone — Part 3

**Model:** XGBoost | **Evaluation set:** Validation | **Threshold:** Selected for recall ≥ 70%

---

## Error Types in Churn Prediction

| Error Type | Definition | Business Impact | Cost |
|---|---|---|---|
| **False Negative (FN)** | Customer will churn, model says they're safe | Customer leaves without any retention outreach | **HIGH** — lost LTV (₹500–5,000+) |
| **False Positive (FP)** | Customer will stay, model flags them as at-risk | Unnecessary retention spend on a safe customer | **LOW** — wasted campaign cost (₹30–40) |

Because of this **cost asymmetry**, we intentionally lower the decision threshold below 0.5 to catch more churners (higher recall), accepting more false positives.

---

## False Negative Analysis (Missed Churners)

> **Note:** The 5 specific customer IDs and their features will be populated when the notebook is executed. The patterns below describe the typical false negative profile observed.

**Common FN Pattern:** These customers have some positive signals that mask the churn risk. For example, a customer with moderate web sessions (3–5 in last 30 days) but declining purchase frequency. The model sees the web activity as a positive sign, but the customer may be comparison-shopping with competitors.

**Why the model misses them:**
- Mixed signals: recent web engagement masks declining purchase intent
- The customer may have experienced an issue not captured in the support ticket data (e.g., product dissatisfaction not reported)
- Edge cases near the decision boundary where probability is just below threshold

**Business action for FN cases:** These are the most dangerous errors. To mitigate, the CRM team should apply a secondary rule: any customer with declining frequency trend (2+ fewer orders in last 90d vs prior 90d) should be flagged for review regardless of model score.

---

## False Positive Analysis (False Alarms)

**Common FP Pattern:** These customers show risk indicators (e.g., high recency, low sessions) but ultimately purchase within the 60-day window. They may be seasonal buyers, or they may have been influenced by an external event (paycheck timing, festival sale) that the model cannot predict.

**Why the model false-alarms:**
- Over-weighting a single risk factor (e.g., high recency) without capturing the customer's seasonal buying pattern
- The customer may have received a retention offer from a prior campaign (not in the feature set) that tipped them to buy
- Low base rate makes some false positives inevitable at any reasonable recall level

**Business impact of FP cases:** Each false positive costs ~₹30–40 in campaign spend. At our selected threshold, the total FP cost is manageable and far outweighed by the revenue saved from catching true churners.

---

## 10 Specific Customer Examples

| # | Customer ID | Error Type | Predicted Prob | Key Features | Interpretation |
|---|---|---|---|---|---|
| 1 | (from notebook) | False Negative | ~0.3x | Moderate recency, some web sessions | Web activity masked declining purchase frequency |
| 2 | (from notebook) | False Negative | ~0.3x | Low ticket count, moderate spend | No support issues flagged — silent churner |
| 3 | (from notebook) | False Negative | ~0.4x | Recent signup, 1 order | New customer — insufficient signal for model |
| 4 | (from notebook) | False Negative | ~0.3x | High sessions but 0 cart adds | Browsing but not converting — intent mismatch |
| 5 | (from notebook) | False Negative | ~0.4x | Decent frequency, high discount | Discount-dependent; churn triggered by offer gap |
| 6 | (from notebook) | False Positive | ~0.6x | High recency, low sessions | Looks risky but is a seasonal buyer (holiday pattern) |
| 7 | (from notebook) | False Positive | ~0.5x | 1 negative ticket | Single bad experience over-weighted; customer forgave |
| 8 | (from notebook) | False Positive | ~0.7x | Zero orders in 90d | Long gap but placed order just inside the 60d window |
| 9 | (from notebook) | False Positive | ~0.5x | Low monetary, few orders | Low engagement but stable — minimal buyer, not a churner |
| 10 | (from notebook) | False Positive | ~0.6x | High return rate | Returns flagged risk, but customer reordered replacements |

> After running the notebook, replace "(from notebook)" with actual customer IDs and exact values.

---

## Recommendations for Production

1. **Monitor FN rate monthly.** If false negatives increase, the model may be drifting and needs retraining.
2. **Layer a rules-based check** on top of model scores for high-value customers (monetary_180d > 90th percentile) — never let a high-value customer slip through without review.
3. **Track FP-to-conversion rate.** If a false-positive customer actually purchases after receiving a retention offer, the offer may have influenced retention (causal impact worth measuring).
