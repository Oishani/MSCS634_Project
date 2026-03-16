# MSCS-634: Project Deliverable 2: Regression Modeling and Performance Evaluation

**Name:** Oishani Ganguly  
**Course:** MSCS-634: Advanced Big Data and Data Mining  
**Dataset:** Customer Shopping Trends Dataset  
**Source:** https://www.kaggle.com/datasets/iamsouravbanerjee/customer-shopping-trends-dataset

---

## Summary

This deliverable builds on the cleaned dataset from Deliverable 1 to develop and evaluate regression models predicting **Purchase Amount (USD)**. Four models are trained, cross-validated, and compared: Linear Regression (baseline), Ridge, Lasso, and Random Forest.

---

## Modeling Process

### Feature Engineering
Before modeling, the following feature engineering steps were applied on top of the Deliverable 1 cleaning pipeline:

- **Dropped non-predictive columns:** `Customer ID`, `Item Purchased` (25 unique values, redundant with Category), `Location` (50 states, too high cardinality), and `Color` (cosmetic, not purchase-driving)
- **Ordinal encoding of Frequency of Purchases:** Mapped 7 purchase cadences to an integer scale (1=Annually → 7=Weekly) to let models treat more-frequent shoppers as numerically distinct
- **One-hot encoding:** Applied to Gender, Category, Season, Size, Shipping Type, and Payment Method using `drop_first=True` to avoid the dummy variable trap
- **Discount_x_Promo interaction:** Product of Discount_Enc and PromoCode_Enc — captures customers who use both simultaneously
- **Age_x_PrevPurchases interaction:** Product of age and purchase history — captures the high-value older loyal customer segment
- **Engagement Score:** Sum of Subscription_Enc + Discount_Enc + PromoCode_Enc (range 0–3) — a composite loyalty signal

### Models
| Model | Regularization | Notes |
|---|---|---|
| Linear Regression | None | OLS baseline |
| Ridge Regression | L2 (alpha tuned via CV) | Shrinks all coefficients |
| Lasso Regression | L1 (alpha tuned via CV) | Zeros out irrelevant features |
| Random Forest | None | Non-linear ensemble, 200 trees |

### Evaluation
- Train/test split: 80/20 with stratification on target quartiles
- Metrics: R², MSE, RMSE
- Generalization: 5-fold cross-validation with Pipeline (scaler + model) to prevent data leakage

---

## Key Insights from Model Evaluation

**All models produced low R² scores (near zero).** This is not a modeling failure — it is the correct and expected result for this dataset. Deliverable 1 EDA showed that Purchase Amount was generated synthetically with near-zero correlation to all other attributes. The target is uniformly distributed between $20 and $100, and no feature carries enough signal to predict where in that range any given transaction falls.

**Random Forest slightly outperformed linear models**, confirming that some weak non-linear interactions exist. However, the improvement over linear models is small, confirming that model complexity alone cannot overcome the fundamental independence between features and target.

**Lasso's feature selection** identified a sparse subset of features with any linear predictive value, zeroing out the rest. This confirms that most features contribute no usable linear signal for this target.

**Cross-validation scores** were consistent with test-set results and showed low variance across folds, confirming the findings are stable and not artifacts of any particular data split.

**Feature importance** from Random Forest elevated the engineered features (`Age_x_PrevPurchases`, `Frequency_Ordinal`) among the top predictors, suggesting they capture modest non-linear patterns that raw features alone do not.

---

## Challenges and Decisions

**Target variable selection:** Purchase Amount (USD) is the most natural regression target in this dataset. The near-uniform distribution and near-zero feature correlations (identified in Deliverable 1) made low model performance predictable. This is addressed directly in the analysis rather than hidden — honest reporting of low R² with clear explanation is more valuable than over-fitting to achieve an inflated score.

**Avoiding data leakage in cross-validation:** StandardScaler was wrapped inside a `sklearn.Pipeline` for all CV evaluations. Fitting the scaler on each training fold (not on the entire dataset) prevents information from the held-out fold from influencing normalization parameters.

**High-cardinality columns:** Location (50 states) and Color (25 values) were dropped rather than one-hot encoded. OHE on 50 states would add 49 near-meaningless dummy columns without geographic aggregation (e.g., region-level). Color is a purely cosmetic attribute with no plausible mechanism for driving purchase amount.

**Alpha tuning for Ridge and Lasso:** Rather than using arbitrary defaults, alpha was selected via 5-fold CV across a grid [0.01, 0.1, 1.0, 10.0, 50.0, 100.0] on the training set only, ensuring the chosen regularization strength genuinely minimizes out-of-sample error.

**Implication for future deliverables:** The difficulty of predicting Purchase Amount linearly reinforces that Subscription Status is a more tractable target for classification. It has stronger behavioral correlates (Promo Code Use, Discount usage) and a binary structure that classification algorithms handle more naturally than a continuous uniform target.
