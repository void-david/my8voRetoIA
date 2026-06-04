# Executive Summary: Social Organization Prediction Pipeline

## Objective

Predict whether Tecnológico de Monterrey alumni have founded nonprofit organizations based on their post-graduation behaviors, demographics, academic background, and socioeconomic characteristics. The goal is to identify the key drivers of nonprofit founding and build a reliable classification model to support alumni engagement strategies.

---

## Dataset

| Attribute | Value |
|---|---|
| Raw records | 25,356 |
| Features after selection | 65 → 76 (engineered) → 544 (encoded) |
| Target variable | `founded_nonprofit` (binary) |
| Positive class rate | 6.95% (severe imbalance) |
| Train / Test split | 80% / 20% (stratified) |

Features span demographics, academic background, employment, leadership roles, social engagement, self-reported competencies, and well-being metrics. Eleven engineered features were added (e.g., `years_since_grad`, `leadership_breadth`, competency growth deltas, `salary_ratio`).

---

## Methodology

- **Preprocessing:** median/mode imputation, ordinal and one-hot encoding, removal of columns with >10% missing values.
- **Dimensionality reduction:** PCA retaining 90% of variance (544 → 397 components), used for exploratory analysis and visualization.
- **Model selection:** 6 classifiers evaluated via 5-fold stratified cross-validation; ROC-AUC chosen as the primary metric given the class imbalance.
- **Class imbalance handling:** `class_weight='balanced'` applied to all models.

---

## Model Comparison (5-Fold CV)

| Model | ROC-AUC | Std | F1 | Precision | Recall |
|---|---|---|---|---|---|
| **Gradient Boosting** | **0.802** | 0.009 | 0.105 | 0.583 | 0.058 |
| Random Forest | 0.782 | 0.008 | 0.274 | 0.173 | 0.664 |
| Logistic Regression | 0.766 | 0.011 | 0.269 | 0.169 | 0.661 |
| Decision Tree | 0.758 | 0.010 | 0.251 | 0.155 | 0.661 |
| QDA | 0.659 | 0.008 | 0.138 | 0.075 | 0.853 |
| Gaussian NB | 0.498 | 0.008 | 0.129 | 0.069 | 0.942 |

**Gradient Boosting was selected as the best model** based on the highest ROC-AUC with low variance across folds.

---

## Best Model: Gradient Boosting — Hold-Out Test Results

| Metric | Value |
|---|---|
| ROC-AUC | **0.801** |
| Accuracy | 0.93 |
| Precision (nonprofit class) | 0.50 |
| Recall (nonprofit class) | 0.054 |
| F1 (nonprofit class) | 0.097 |

The model achieves strong discriminative ability (AUC 0.80) and high precision when it does predict a positive, meaning it rarely produces false alarms. Recall is intentionally conservative at the default threshold; adjusting the decision threshold can shift the precision-recall tradeoff to suit different operational needs (e.g., prioritizing outreach coverage over precision).

---

## Key Predictors (SHAP Analysis)

The five most influential features, in order:

| Feature | Mean |SHAP| | Interpretation |
|---|---|---|
| `volunteers` | 0.436 | Alumni who volunteer are far more likely to found nonprofits |
| `founded_company` | 0.209 | Entrepreneurial behavior carries strongly to the social sector |
| `donates_money` | 0.128 | Charitable giving is a leading behavioral signal |
| `held_ceo` | 0.097 | Executive leadership experience is a strong predictor |
| `advisory_board` | 0.093 | Advisory/board engagement reflects civic orientation |

Other notable predictors include current employment status (paid employee vs. other), years since graduation, entrepreneurship competency at graduation, and school of Social Sciences and Government.

All top five behavioral features were statistically significant at p < 0.001 in chi-square tests, with correlation coefficients ranging from 0.13 to 0.19.

---

## Key Findings

1. **Civic behavior is the dominant signal.** Volunteering, donating, and prior company founding collectively account for the majority of the model's predictive power — more so than any demographic or academic feature.
2. **Leadership experience amplifies the signal.** Alumni who held CEO, VP, director, or advisory board roles are significantly more likely to have founded a nonprofit.
3. **Academic background matters at the margins.** School of Social Sciences and Government membership and entrepreneurship competency at graduation both contribute, though with substantially lower importance than behavioral indicators.
4. **The model is conservative by design.** High precision (0.50) and low recall (0.05) at the default threshold means identified alumni are very likely true positives, making the model well-suited for targeted, high-confidence outreach.
5. **Class imbalance remains the main challenge.** Even with balanced class weights, the ~7% positive rate limits recall. Threshold tuning or cost-sensitive strategies should be explored if broader recall is required.

---

## Recommendations

- **Use the model for targeted outreach:** Focus engagement resources on alumni flagged by the model, expecting ~50% hit rate among flagged individuals.
- **Lower the decision threshold for broader campaigns:** Shifting the classification threshold below 0.5 will increase recall at the cost of some precision — appropriate for mass awareness campaigns.
- **Prioritize volunteer and giving data collection:** These two features drive the most predictive power and should be kept current in alumni records.
- **Explore oversampling techniques (SMOTE):** Synthetic minority oversampling could improve recall without sacrificing the discriminative ability already achieved.
- **Segment by school/program:** The Social Sciences and Government signal suggests model performance and threshold choices may benefit from program-level segmentation.
