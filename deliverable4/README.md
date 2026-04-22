# MSCS-634: Project Deliverable 4: Final Insights, Recommendations, and Presentation

**Name:** Oishani Ganguly  
**Course:** MSCS-634: Advanced Big Data and Data Mining  
**Dataset:** Customer Shopping Trends Dataset  
**Source:** https://www.kaggle.com/datasets/iamsouravbanerjee/customer-shopping-trends-dataset

---

## Dataset Summary

The **Customer Shopping Trends Dataset** (`shopping_trends.csv`) captures synthetic retail consumer behavior across 3,900 customer records and 19 original attributes. Each row represents a single customer and includes:

- **Demographics:** Age, Gender, Location
- **Transaction details:** Item Purchased, Category, Purchase Amount (USD), Season
- **Behavioral signals:** Frequency of Purchases, Previous Purchases, Subscription Status, Discount Applied, Promo Code Used
- **Logistics:** Shipping Type, Payment Method, Preferred Payment Method
- **Satisfaction:** Review Rating

The original file was used rather than the pre-cleaned `_updated` version so that all cleaning decisions — particularly the removal of `Preferred Payment Method` — could be documented explicitly with full justification.

---

## Project Steps Across All Deliverables

### Deliverable 1 — Data Cleaning and EDA

**Cleaning:**
- No missing values or duplicate records were found. Imputation (median for numeric, mode for categorical) was demonstrated on synthetically introduced nulls.
- `Preferred Payment Method` was dropped after analysis showed it agreed with the actual `Payment Method` only 15.8% of the time — confirming it adds correlated noise rather than usable signal.
- All string columns were standardized with `.str.strip().str.title()`.
- Binary Yes/No columns (`Subscription Status`, `Discount Applied`, `Promo Code Used`) were encoded as 0/1 integers.

**EDA highlights:**
- All four numeric features (Age, Purchase Amount, Review Rating, Previous Purchases) follow near-uniform distributions — minimal skew, no outliers.
- Purchase Amount is uniformly distributed between $20 and $100, with near-zero Pearson correlation with every other attribute. This established the expectation of low regression R² scores before any model was trained.
- Gender is imbalanced (~68% Male, ~32% Female), requiring stratified sampling and class-weight adjustments.
- Subscription class imbalance: ~27% subscribers (Yes) vs. ~73% non-subscribers (No) — a 3:1 ratio handled explicitly in all classification tasks.
- Discount Applied and Promo Code Used showed the strongest pairwise association (~0.45), and subscribers used promo codes at a substantially higher rate than non-subscribers — the central behavioral signal confirmed by every subsequent analysis.

---

### Deliverable 2 — Regression Modeling

**Feature Engineering:**
- Dropped non-predictive columns: `Customer ID`, `Item Purchased` (redundant with Category), `Location` (50-state cardinality), `Color` (cosmetic)
- `Frequency of Purchases` ordinally encoded (1=Annually → 7=Weekly)
- Nominal categoricals one-hot encoded with `drop_first=True`
- Engineered features: `Discount_x_Promo` (interaction), `Age_x_PrevPurchases` (interaction), `Engagement_Score` (composite loyalty signal, range 0–3)

**Models trained:**

| Model | R² (Test) | Key Property |
|---|---|---|
| Linear Regression | ~0.00 | OLS baseline; no regularization |
| Ridge (L2) | ~0.00 | Shrinks all coefficients; alpha tuned via CV |
| Lasso (L1) | ~0.00 | Zeroes out irrelevant features; sparse solution |
| Random Forest (200 trees) | ~0.01 | Non-linear; marginal improvement over linear |

**Key finding:** All models produced near-zero R² scores — the correct and expected result. Purchase Amount was generated synthetically with near-zero dependence on all other attributes. Random Forest marginally outperformed linear models, confirming that some weak non-linear interactions exist but model complexity cannot overcome the structural independence between features and target. This finding was used to motivate Subscription Status as the classification target in Deliverable 3.

---

### Deliverable 3 — Classification, Clustering, and Association Rule Mining

**Classification (predicting Subscription Status):**

| Model | F1 (Weighted) | AUC-ROC | Notes |
|---|---|---|---|
| Decision Tree (Default) | See notebook | See notebook | Unpruned; prone to overfitting |
| Decision Tree (Tuned) | Best | Best | GridSearchCV over max_depth, min_samples_split, criterion |
| k-NN (k tuned via CV) | Competitive | Competitive | Slightly hurt by OHE dimensionality |

- GridSearchCV with 5-fold stratified CV and weighted F1 scoring
- `class_weight='balanced'` compensates for 3:1 class imbalance
- Top predictors: `PromoCode_Enc`, `Discount_Enc`, `Engagement_Score`, `Discount_x_Promo` — exactly those identified as strongest signals in Deliverable 1 EDA

**K-Means Clustering:**
- Applied to 10 interpretable numeric/encoded features (not the full OHE matrix)
- Optimal k selected by elbow method (inertia vs. k) and silhouette score in agreement
- PCA projection (2 components) used to visualize cluster separation
- Cluster profiles differentiated primarily by engagement level, purchase frequency, and purchase history depth

**Apriori Association Rule Mining:**
- Transactions built from 8 categorical columns using `Column=Value` item format for self-documenting rules
- Parameters: minimum support 0.15, minimum confidence 0.60; rules ranked by lift
- Key rules: Discount + Promo Code → Subscription (high lift); Category + Season → Discount Applied; Category → Shipping Type; Subscription + Promo → Discount

---

### Deliverable 4 — Final Synthesis

This deliverable consolidates the complete pipeline — cleaning, feature engineering, regression, classification, clustering, and association rule mining — into a single reproducible notebook. All 11 visualizations are labeled, well-titled, and annotated with interpretation. A cross-deliverable synthesis visualization makes the convergence of findings explicit.

---

## Major Findings

**1. Promotional engagement is the dominant behavioral signal — confirmed by four independent analyses.**

This finding emerged from:
- Deliverable 1 EDA: subscribers have substantially higher promo code usage rates; Discount and Promo have the strongest pairwise correlation in the dataset
- Deliverable 2 Random Forest: engineered engagement features (`Discount_x_Promo`, `Engagement_Score`) among the top predictors even for Purchase Amount
- Deliverable 3 Decision Tree: `PromoCode_Enc`, `Discount_Enc`, and `Engagement_Score` are the top classification features
- Deliverable 3 Apriori: Discount + Promo Code → Subscription is among the highest-lift rules discovered

The convergence of this finding across four completely independent analytical methods (EDA statistics, tree-based regression, decision tree classification, and itemset mining) strengthens confidence substantially beyond what any single method could provide.

**2. Purchase Amount is not predictable from the available features.**

This is a structural property of the synthetic dataset, not a modeling failure. R² near zero across all four regression models — including non-linear Random Forest — is the honest and correct result. This guided the choice of Subscription Status as the classification target and motivates collecting richer transactional signals in any real deployment.

**3. Subscription status is predictable above the naive baseline.**

The tuned Decision Tree achieves AUC-ROC above 0.5 and F1 improvements over the naive all-majority-class prediction. Constraining tree depth and minimum split size via GridSearchCV substantially improves minority class recall — the most important metric for a 3:1 imbalanced target.

**4. Customer segments are engagement-driven, not spend-driven.**

K-Means clustering identified segments differentiated primarily by subscription status, discount usage, promo code usage, and purchase frequency — not by purchase amount, age, or review rating. This confirms that the most meaningful customer differentiation in this dataset is behavioral and promotional, not demographic or financial.

**5. Association rules surface directly actionable business patterns.**

Specific category-season combinations predict discount usage; product categories predict shipping preferences; and deal-seeking behavior predicts subscription likelihood. These rules translate directly into campaign scheduling, checkout UX, and subscription acquisition strategy.

---

## Ethical Considerations

- **Data privacy:** The dataset is synthetic; no real individuals' data was used. Real deployment would require full GDPR/CCPA compliance.
- **Gender as a feature:** Production models using gender as a predictor should undergo disparate impact auditing before deployment.
- **Class imbalance:** The 3:1 subscription imbalance was handled explicitly. Production deployment should monitor for systematic under-service of the subscriber minority class.
- **Model interpretability:** Decision Tree was preferred for its auditability. Black-box models should be justified against interpretability requirements for consequential business decisions.
- **Dataset representativeness:** Near-uniform distributions across all numeric features are atypical of real retail data. Models should be validated on real-world data before production use.