# Churn Prediction and Risk Scoring

Predict which customers of an online retailer will **stop buying in the next 90 days**, using the
[Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii) transactions (Dec 2009 - Dec 2011).

## Approach

- **Data cleaning**: load both Excel sheets, remove cancellations, returns, zero/negative prices and rows without a Customer ID (805,549 order lines, 5,878 customers left).
- **Churn definition**: at a cutoff date, an active customer (bought in the previous 365 days) is a churner if they make **no purchase in the next 90 days**.
- **Features** (computed only from data before the cutoff): recency, tenure, number of orders, total spend, average order value, orders and spend in the last 90 days, average days between orders, number of products, UK flag, recency relative to the customer's usual rhythm.
- **Time-based split**: train on cutoffs 2010-09, 2010-12, 2011-03; validation on 2011-06 (threshold choice); test on 2011-09.
- **Models**: Logistic Regression baseline (log + scaling), Random Forest, XGBoost (small search on validation).
- **Evaluation**: precision, recall, PR-AUC, ROC-AUC, 95% bootstrap confidence intervals, paired bootstrap to compare models, threshold comparison (missed churners vs false alerts).
- **Error analysis** of false positives / false negatives and **SHAP** explanations (global and per customer).

## Results (test set, 4,335 customers, churn rate 50.5%)

| Model | PR-AUC [95% CI] | ROC-AUC |
|---|---|---|
| Logistic Regression | 0.740 [0.720, 0.760] | 0.762 |
| Random Forest | 0.721 [0.700, 0.743] | 0.754 |
| XGBoost | 0.737 [0.716, 0.758] | 0.766 |

- XGBoost and the Logistic Regression baseline are **not significantly different** (paired bootstrap); Random Forest is slightly worse.
- Threshold 0.4 (best F1): recall 0.94, precision 0.60. Threshold 0.6: recall 0.82, precision 0.66, ~440 fewer false alerts.
- Main drivers (SHAP): number of orders, recency, orders in the last 90 days, tenure.
- **False positives** are mostly once-a-year buyers who come back for Christmas; **false negatives** are very active customers who stop suddenly.

## How to run

```bash
pip install pandas openpyxl matplotlib scikit-learn xgboost shap jupyter
jupyter notebook churn_prediction.ipynb
```

Download `online_retail_II.xlsx` from the [UCI page](https://archive.ics.uci.edu/dataset/502/online+retail+ii) and put it next to the notebook (it is not committed, see `.gitignore`).
