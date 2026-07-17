# MuleShield: A Machine Learning System for Mule Account Detection in Banking Transactions

## Abstract

MuleShield is an end-to-end machine learning pipeline designed to detect money mule accounts within highly imbalanced banking transaction data. The system integrates unsupervised anomaly detection with supervised classification to deliver both high predictive performance and model interpretability. By systematically addressing challenges such as extreme class imbalance, target leakage, and evaluation bias, MuleShield achieves reliable and reproducible fraud detection suitable for real-world banking risk assessment.

---

## 1. Background: What Is a Mule Account?

A **money mule account** is a bank account used, knowingly or unknowingly, to receive and transfer funds derived from illegal activity — such as phishing scams, identity theft, or other forms of financial fraud — on behalf of a third party. The account holder ("mule") typically moves the illicit funds to another account, often retaining a small commission, while obscuring the money's origin and making it difficult for investigators to trace the transaction chain back to the original criminal.

Mule accounts form a critical link in the money laundering process, allowing fraudulent funds to be layered through seemingly legitimate financial activity before final withdrawal. Detecting these accounts early is essential for banks and financial institutions to prevent losses, comply with anti-money-laundering (AML) regulations, and protect genuine customers from being unwittingly implicated in fraud.

The core difficulty in detecting mule accounts computationally lies in their rarity relative to the overall customer base, and in the fact that their transactional behavior can closely mimic legitimate activity — making this a textbook case of a **severe class imbalance problem** in machine learning.

---

## 2. Problem Statement

Conventional classification models are typically trained under the assumption of a reasonably balanced class distribution. In fraud detection, this assumption breaks down: fraudulent accounts represent a very small fraction of the total population, causing standard models to become biased toward the majority (legitimate) class and to consistently underperform on the minority (fraudulent) class — the class that actually matters most.

The objective of this project was to design a machine learning system that:
- Reliably identifies mule accounts despite extreme class imbalance
- Minimizes false positives to avoid disrupting legitimate customers
- Produces evaluation metrics that genuinely reflect real-world performance, free from data leakage
- Remains interpretable, so that flagged accounts can be explained and audited by a human analyst

---

## 3. Dataset

| Attribute | Detail |
|---|---|
| Total records | 9,082 |
| Engineered features | 4,000+ |
| Class imbalance ratio | 112 : 1 (legitimate : mule) |
| Data type | Anonymized banking transaction and account-level features |

**Dataset link:** [Download here](https://drive.google.com/uc?export=download&id=12dy_n7nbQfiPolBJ5qUS_7Ng0S0wwVD5)

The high dimensionality and extreme imbalance of this dataset made it representative of real-world fraud analytics environments, where positive (fraud) cases are rare and features are numerous, often noisy, and not all equally informative.

---

## 4. Methodology

The pipeline was designed as a sequential, leakage-aware workflow:

### 4.1 Data Preprocessing and Feature Selection
Raw features were cleaned and screened to remove redundant or non-informative variables, reducing noise before model training.

### 4.2 Detection and Removal of Target Leakage
During exploratory analysis, feature **F3912** was found to be a near-duplicate of the target label itself, artificially inflating model performance to unrealistic levels. This feature was identified and removed to ensure the model learned genuine behavioral patterns rather than memorizing the label. This step was critical to producing a trustworthy, generalizable model.

### 4.3 Stratified Train-Test Split
The data was split into training and test sets using stratified sampling to preserve the original 112:1 class ratio in both partitions, ensuring the test set remained a realistic and unbiased representation of production data.

### 4.4 Class Imbalance Correction with SMOTE
The **Synthetic Minority Oversampling Technique (SMOTE)** was applied strictly to the training set only. Applying SMOTE before the split — a common mistake — would leak synthetic information into the test set and produce overly optimistic results. Restricting oversampling to the training partition preserved the integrity of the evaluation.

### 4.5 Anomaly Scoring with Isolation Forest
An **Isolation Forest** model was used to generate unsupervised anomaly scores for each account, capturing irregular behavioral patterns that deviate from the norm. These scores were incorporated as an additional engineered feature, giving the downstream classifier a signal grounded in outlier behavior rather than labeled examples alone.

### 4.6 Classification with XGBoost
The final classification stage used **XGBoost**, a gradient-boosted decision tree algorithm well suited to structured, high-dimensional, imbalanced data. Trained on the SMOTE-balanced training set enriched with anomaly scores, XGBoost served as the primary decision-making model for flagging mule accounts.

### 4.7 Model Interpretability with SHAP
**SHAP (SHapley Additive exPlanations)** was applied to the trained model to quantify the contribution of each feature to individual predictions. This step transformed the model from a "black box" into an auditable decision system, allowing analysts to understand precisely why a given account was flagged as suspicious.

---

## 5. Model Evaluation

Given the severe class imbalance, accuracy alone would be a misleading metric — a naive model predicting "legitimate" for every account would still score above 99% accuracy while catching zero fraud. The model was therefore evaluated using metrics specifically suited to imbalanced classification:

| Metric | Score | Interpretation |
|---|---:|---|
| Precision | 89% | Of all accounts flagged as mule accounts, 89% were correctly identified |
| Recall | 76% | Of all actual mule accounts in the data, 76% were successfully detected |
| ROC-AUC | 0.999 | Near-perfect ability to distinguish mule from legitimate accounts across all thresholds |
| PR-AUC | 0.936 | Strong precision-recall trade-off, the more informative metric under severe imbalance |

These results were obtained on a held-out stratified test set after leakage removal and training-only SMOTE application, ensuring they reflect genuine model performance rather than an artifact of data leakage or optimistic sampling.

---

## 6. Key Findings and Interpretation

- **Leakage removal was decisive.** The presence of feature F3912 had been inflating performance to near-trivial levels; its removal forced the model to learn from genuine transactional patterns, resulting in metrics that are lower but credible and production-relevant.
- **PR-AUC over accuracy.** In a 112:1 imbalanced setting, PR-AUC (0.936) is the most meaningful single indicator of model quality, as it directly reflects performance on the rare, high-stakes minority class.
- **Precision-recall balance reflects real banking trade-offs.** An 89% precision means the majority of flagged accounts genuinely warrant investigation, minimizing wasted analyst effort and customer friction. A 76% recall indicates the model captures a substantial majority of true mule accounts, while leaving room for complementary rule-based or manual review layers to catch the remainder.
- **Hybrid anomaly + supervised architecture adds robustness.** Combining Isolation Forest's unsupervised anomaly signal with XGBoost's supervised classification allowed the system to benefit from both labeled fraud patterns and unlabeled behavioral irregularities, improving detection of novel or evolving mule tactics that may not closely resemble historical fraud examples.
- **Interpretability enables real-world deployment.** SHAP-based explanations mean that every flagged account can be accompanied by a clear, feature-level justification — an essential requirement for compliance, auditability, and analyst trust in any AML system deployed in production.

---

## 7. Conclusion

MuleShield demonstrates that a carefully engineered, leakage-aware machine learning pipeline can achieve strong, trustworthy performance on a highly imbalanced fraud detection task. By combining anomaly detection, supervised gradient boosting, and rigorous evaluation practices, the system achieves a ROC-AUC of 0.999 and a PR-AUC of 0.936 — indicating excellent separability between mule and legitimate accounts even under a 112:1 class imbalance. Precision and recall of 89% and 76% respectively reflect a model that is both actionable in production (few false alarms) and effective at catching genuine fraud. Combined with SHAP-based interpretability, MuleShield offers a technically sound and practically deployable foundation for mule account detection in real-world banking risk systems.

---

## 8. Technologies Used

- **Python**
- **XGBoost**
- **Isolation Forest** (Scikit-learn)
- **Scikit-learn**
- **SHAP**
- **Pandas**
- **NumPy**
- **Imbalanced-learn (SMOTE)**

---

## Author

**Priya Gundale**
