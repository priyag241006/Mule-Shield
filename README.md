# MuleShield – Mule Account Detection System

## Overview

MuleShield is a machine learning project developed to identify suspicious mule bank accounts from highly imbalanced banking transaction data. The project combines anomaly detection with supervised learning to improve fraud detection while maintaining model interpretability.

The primary focus was to build a reliable fraud detection pipeline by addressing challenges such as class imbalance, target leakage, and model evaluation.

---

## Problem Statement

Money mule accounts are frequently used to transfer illegally obtained funds through legitimate banking systems. Since fraudulent accounts represent only a small fraction of the total data, conventional classification models often fail to identify them effectively.

This project aims to develop a robust machine learning model capable of detecting mule accounts while minimizing false positives and ensuring reliable evaluation.

---

## Dataset

- Banking transaction dataset
- **9,082 records**
- **4,000+ engineered features**
- Severe **112:1 class imbalance**
- Dataset used- https://drive.google.com/uc?export=download&id=12dy_n7nbQfiPolBJ5qUS_7Ng0S0wwVD5

---

## Methodology

The model was developed using the following workflow:

1. Data preprocessing and feature selection.
2. Detection and removal of target leakage (`F3912`) to prevent artificially inflated performance.
3. Stratified train-test split to preserve the original class distribution.
4. Application of **SMOTE** only on the training data to address class imbalance.
5. Generation of anomaly-based information using **Isolation Forest**.
6. Final mule account classification using **XGBoost**.
7. Model interpretation using **SHAP** to understand feature contributions.

---

## Model Evaluation

The final model was evaluated using a stratified train-test split after removing target leakage and applying SMOTE only to the training set.

| Metric | Score |
|---------|------:|
| Precision | **89%** |
| Recall | **76%** |
| ROC-AUC | **0.999** |
| PR-AUC | **0.936** |

These results indicate that the model effectively distinguishes fraudulent accounts from legitimate ones while maintaining strong performance on the minority class.

---

## Key Observations

- Identified and removed a target leakage feature (`F3912`) that initially produced unrealistically high performance.
- Applied SMOTE only to the training data to prevent data leakage during evaluation.
- Used Precision, Recall, ROC-AUC, and PR-AUC as evaluation metrics due to the severe class imbalance.
- SHAP analysis improved model interpretability by explaining the contribution of individual features.
- The combination of Isolation Forest and XGBoost improved the model's ability to identify suspicious mule accounts.

---

## Technologies Used

- Python
- XGBoost
- Isolation Forest
- Scikit-learn
- SHAP
- Pandas
- NumPy
- Imbalanced-learn (SMOTE)

---


---

## Author

**Priya Gundale**
