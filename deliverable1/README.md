# MSCS-634: Project Deliverable 1: Data Collection, Cleaning, and Exploration

**Name:** Oishani Ganguly  
**Course:** MSCS-634: Advanced Big Data and Data Mining  
**Dataset:** Customer Shopping Trends Dataset  
**Source:** https://www.kaggle.com/datasets/iamsouravbanerjee/customer-shopping-trends-dataset

---

## Dataset Summary

The original `shopping_trends.csv` file contains 3,900 customer records across 19 attributes capturing retail consumer behavior. Each row represents a single customer and includes:

- **Demographics:** Age, Gender, Location
- **Transaction details:** Item Purchased, Category, Purchase Amount (USD), Season
- **Behavioral signals:** Frequency of Purchases, Previous Purchases, Subscription Status, Discount Applied, Promo Code Used
- **Logistics:** Shipping Type, Payment Method, Preferred Payment Method
- **Satisfaction:** Review Rating

After cleaning, the dataset is reduced to 18 attributes (see Data Cleaning below). It exceeds the minimum project requirements of 500+ records and 8–10 attributes, and contains a rich mix of numeric, categorical, and binary variables suitable for all four project deliverables.

---

## Key Insights from EDA

**Numeric Distributions**
- All four numeric features (Age, Purchase Amount, Review Rating, Previous Purchases) follow near-uniform distributions with near-equal mean and median. This indicates minimal skew and no extreme outliers.
- Purchase Amount ranges between $20 and $100 with a mean of ~$60 — a stable, well-bounded regression target.

**Categorical Distributions**
- Gender is imbalanced (~68% Male, ~32% Female), requiring stratified sampling and class-weight adjustments in classification to avoid majority-group bias.
- Clothing accounts for ~45% of all purchases and dominates across every season — a key feature for clustering and association rules.
- Purchase frequency spans weekly to annually, providing a natural basis for customer segmentation.

**Payment Method Analysis**
- The dataset originally contained both `Payment Method` (actual transaction method) and `Preferred Payment Method` (stated preference). Customers used their stated preferred method only **15.8%** of the time (84.2% disagreement rate). Both columns contain the same six values with near-identical distributions, confirming that `Preferred Payment Method` adds correlated noise rather than useful signal.

**Subscription and Promotional Behavior**
- Only ~27% of customers hold a subscription (3:1 class imbalance) — a manageable but notable imbalance for binary classification.
- Subscribers use promo codes at a substantially higher rate than non-subscribers, making promo code usage a strong predictive feature for the classification model.
- Average purchase amount is virtually identical between subscribers and non-subscribers (~$60), suggesting subscription relates to visit frequency rather than per-transaction spend.

**Correlations and Relationships**
- No strong linear correlations exist between any pair of features. Discount Applied and Promo Code Used show the strongest association (~0.45), consistent with their natural behavioral overlap.
- Age shows no meaningful linear relationship with Purchase Amount for either gender, motivating the use of tree-based or non-linear models in the future.

**Outliers**
- IQR analysis confirmed zero outliers across all four numeric features. All values fall within realistic real-world bounds.

---

## Data Cleaning Steps

1. **Missing Values:** The dataset contains no missing values. Imputation techniques (median fill for numeric, mode fill for categorical) were demonstrated on a synthetically modified copy to satisfy the preprocessing requirement.

2. **Duplicate Records:** No exact duplicate rows or repeated Customer IDs were found.

3. **Redundant Column Removal:** `Preferred Payment Method` was dropped after analysis revealed it agrees with the actual `Payment Method` only 15.8% of the time. Since the column does not reliably reflect transaction behavior and shares the same value domain, retaining it would introduce correlated noise. This mirrors the change made in the dataset's `_updated` version, and is documented here explicitly with justification.

4. **Inconsistent Formatting:** All string columns were standardized with `.str.strip().str.title()` to remove whitespace and normalize casing.

5. **Numeric Range Validation:** All numeric columns were checked against expected real-world bounds. No corrections were needed.

6. **Binary Encoding:** Subscription Status, Discount Applied, and Promo Code Used were encoded as integer 0/1 columns in preparation for future modeling deliverables.

---

## Challenges and Decisions

**Two-file dataset:** The Kaggle source provides both `shopping_trends.csv` (19 columns) and `shopping_trends_updated.csv` (18 columns). Rather than using the pre-cleaned version, we used the original file and explicitly documented the `Preferred Payment Method` removal as a real, justified cleaning decision — strengthening the preprocessing narrative.

**No natural missing values:** Because the dataset is synthetic and complete, missing value handling had to be demonstrated on an artificially modified copy. Null values were injected at controlled rates (4% numeric, 3% categorical) to keep the demonstration realistic.

**Imputation strategy:** Median fill was chosen over mean for numeric imputation because median is more robust to skew. Mode fill was used for categorical columns as the most principled default for nominal data.

**Encoding scope:** Only binary Yes/No columns were encoded in this deliverable. Full categorical encoding (one-hot or label encoding for multi-value columns) is deferred to future deliverables, where the choice of modeling algorithm will dictate the appropriate strategy.

**Correlation findings:** The near-zero correlations between all features and Purchase Amount indicate that linear regression alone will likely produce low R² scores. This motivated early planning for ensemble methods (Random Forest, Gradient Boosting) in future deliverables as primary models with linear regression included for baseline comparison.
