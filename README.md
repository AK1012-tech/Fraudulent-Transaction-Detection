# 🛡️ Fraudulent Transaction Detection

## 📌 Overview

This project predicts fraudulent financial transactions using machine learning and derives actionable insights for fraud prevention.
It uses a **realistic financial dataset** with over **6.3 million transactions** and applies **statistical analysis, feature engineering, and explainable AI** to detect fraud patterns.

---

## 📊 Dataset Information

* **Rows**: 6,362,620 
* **Columns**: 10 (transaction details + target label)
* **Target**: `isFraud` (1 = fraudulent transaction, 0 = genuine transaction)

  <img width="1504" height="759" alt="image" src="https://github.com/user-attachments/assets/b28981ad-de3f-4ec6-a434-c8329ead8a7e" />


**Key features**:

* `step` — time step (1 = 1 hour)
* `type` — transaction type (CASH\_IN, CASH\_OUT, TRANSFER, PAYMENT, DEBIT)
* `amount` — transaction amount
* `oldbalanceOrg` / `newbalanceOrig` — origin account balances
* `oldbalanceDest` / `newbalanceDest` — destination account balances
* `isFlaggedFraud` — business rule flag (amount > 200k)

> 📌 **Note:** Some merchant accounts (`nameDest` starting with 'M') have missing balances — handled separately during preprocessing.

---

## 🛠️ Methodology

### 1️⃣ Data Loading & Sampling

* Reads CSV in **chunks** to handle memory efficiently.
* Keeps all fraud cases, caps majority (non-fraud) class for balanced training.

### 2️⃣ Data Cleaning

* Verified **no missing values** (except intentional merchant balances).
* **Integrity checks** based on data dictionary (balance equations).
* **Outlier clipping** (0.5% tails) for numeric stability.
* Dropped high-cardinality identifiers (`nameOrig`, `nameDest`) to prevent leakage.

### 3️⃣ Feature Engineering

* Transaction type one-hot encoding.
* Merchant flags (`orig_is_merchant`, `dest_is_merchant`).
* Balance mismatch flags (`orig_mismatch`, `dest_mismatch`).
* Amount-to-balance ratios (`amount_to_origbal`, `amount_to_destbal`).
* Zero balance indicators before transaction.

### 4️⃣ Multicollinearity Handling

* Correlation filtering (`|r| > 0.98`).
* VIF (Variance Inflation Factor) analysis for remaining numeric features.

### 5️⃣ Modeling

Two models trained:

1. **Logistic Regression** — interpretable, scaled features, `class_weight='balanced'`
2. **HistGradientBoostingClassifier** — handles non-linear relationships, efficient on large datasets.

### 6️⃣ Class Imbalance Handling

* Class weights proportional to fraud/non-fraud counts.
* Stratified train-test split (75% train, 25% test).

### 7️⃣ Model Evaluation

* **ROC-AUC** and **PR-AUC** for both models.
* **Threshold tuning** using **F2-score** (prioritize recall).
* **Precision\@K** (top alerts for investigation).
* **Cost-benefit analysis** for threshold selection.

### 8️⃣ Explainability

* **Permutation importance** for feature ranking.
* **SHAP analysis**:

  * Global summary plot (impact & direction of top features)
  * Business interpretation table linking features to fraud patterns.

---

## 📈 Results

| Metric              | Logistic Regression | HistGradientBoosting |
| ------------------- | ------------------- | -------------------- |
| ROC-AUC             | 0.98+               | 0.999+               |
| PR-AUC              | High                | Very High            |
| Best Threshold (F2) | \~0.3–0.4           | \~0.3–0.4            |

**Top Predictive Features** (SHAP):

1. `step` — Fraud peaks at specific hours.
2. `newbalanceOrig` — Low remaining balance after transaction → account draining.
3. `orig_mismatch` — Balance change mismatch at origin account.
4. `type_CASH_OUT` — High-risk cash-out operations.
5. `amount_to_origbal` — Large proportion of balance being moved.

---

## 🛡️ Fraud Prevention Recommendations

Based on model insights:

1. **Real-time scoring** of transactions with model output.
2. **Velocity limits**: cap number of high-value transactions per hour/day.
3. **Step-up authentication** for:

   * High `amount_to_origbal`
   * CASH\_OUT or TRANSFER above threshold
4. **Flag first-time merchant recipients** with large amounts.
5. **Time-window monitoring** during high-fraud `step` periods.

---

## 📊 Measuring Effectiveness

* **A/B Testing**:

  * Control: current system.
  * Treatment: current system + ML model alerts.
* **KPIs**:

  * Fraud rate reduction.
  * Chargeback value prevented.
  * False positive rate (investigation efficiency).
* **Drift Monitoring**:

  * Monitor feature distributions & model score stability over time.
  * Retrain when performance drops.




