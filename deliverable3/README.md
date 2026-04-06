# MSCS-634: Project Deliverable 3: Classification, Clustering, and Pattern Mining

**Name:** Oishani Ganguly  
**Course:** MSCS-634: Advanced Big Data and Data Mining  
**Dataset:** Customer Shopping Trends Dataset  
**Source:** https://www.kaggle.com/datasets/iamsouravbanerjee/customer-shopping-trends-dataset
---

## Summary

This deliverable applies three unsupervised and supervised mining techniques to the Customer Shopping Trends dataset: classification (predicting subscription status), customer segmentation (K-Means clustering), and pattern discovery (Apriori association rule mining). All findings connect back to the EDA and regression analysis performed in Deliverables 1 and 2.

---

## Classification: Predicting Subscription Status

**Target variable:** `Subscription Status` (binary: Yes=1, No=0)  
**Class balance:** ~27% subscribers, ~73% non-subscribers (3:1 imbalance handled via `class_weight='balanced'` and stratified splits)

### Models

| Model | Notes |
|---|---|
| Decision Tree (default) | Baseline; unpruned, prone to overfitting |
| Decision Tree (tuned) | GridSearchCV over max_depth, min_samples_split, criterion |
| k-NN (k tuned via CV) | Optimal k selected by 5-fold stratified CV F1 score |

### Hyperparameter Tuning
GridSearchCV with 5-fold stratified cross-validation was applied to the Decision Tree. The grid covered:
- `max_depth`: [3, 5, 7, 10, 15, None]
- `min_samples_split`: [2, 10, 20, 50]
- `criterion`: ['gini', 'entropy']

Tuning was scored on weighted F1 to account for class imbalance. Constraining max_depth and raising min_samples_split reduced overfitting on training noise and improved generalization on the test set.

### Key Results
- The tuned Decision Tree outperformed the default in both F1 and AUC-ROC
- Both models produce AUC-ROC above 0.5, confirming genuine discriminative power over the imbalanced baseline
- **Top predictors:** `PromoCode_Enc`, `Discount_Enc`, `Engagement_Score`, and `Discount_x_Promo` — consistent with Deliverable 1 EDA findings that subscribers use promotional offers at much higher rates than non-subscribers
- k-NN is competitive but slightly lower in F1, likely due to the curse of dimensionality introduced by one-hot encoding across 6 categorical columns

---

## Clustering: K-Means Customer Segmentation

**Features used:** Age, Purchase Amount, Review Rating, Previous Purchases, Frequency (ordinal), Subscription, Discount, Promo, Engagement Score, Age × Previous Purchases

**Optimal k** was determined using both the elbow method (inertia vs. k) and silhouette score. The two methods were compared to ensure the chosen k was supported by both criteria.

### Cluster Profiles
K-Means identified distinct customer segments differentiated primarily by:
- **Engagement level** (subscription + discount + promo code usage)
- **Purchase frequency** (Frequency_Ordinal range 1–7)
- **Purchase history depth** (Previous Purchases and Age × PrevPurchases)

A standardized heatmap of mean feature values per cluster enables direct business interpretation:
- High-engagement clusters (high Subscription + Discount + Promo usage) represent loyal subscribers — prime retention targets
- Low-frequency, low-engagement clusters represent occasional shoppers — acquisition targets
- Higher Age × PrevPurchases clusters represent older, high-loyalty customers with established spending patterns

**PCA projection** (2 components) visualizes cluster separation in 2D. The total variance explained by the two components is reported in the chart axis labels.

---

## Association Rule Mining: Apriori

**Transaction design:** Each customer is treated as a transaction basket containing attribute=value items from: Category, Season, Size, Discount Applied, Promo Code Used, Subscription Status, Shipping Type, and Gender.

**Parameters:**
- Minimum support: 0.15 (item combination must appear in ≥15% of customers)
- Minimum confidence: 0.60 (consequent must appear in ≥60% of transactions containing the antecedent)
- Rules ranked by lift

### Key Rules and Business Applications

**Discount + Promo Code → Subscription (high lift)**  
Customers using both discount and promo codes are significantly more likely to subscribe than the base rate predicts. *Application:* Target non-subscribing deal-users with subscription offers bundled with promo incentives.

**Category + Season → Discount Applied**  
Specific category-season combinations strongly predict discount usage. *Application:* Schedule seasonal discount campaigns proactively for the highest-lift combinations rather than applying them broadly.

**Subscription + Promo → Discount**  
Subscribers using promo codes almost always also apply discounts — a "maximum engagement" power-user segment. *Application:* Monitor this segment's sensitivity to changes in discount or promo availability; they may churn disproportionately if offers are reduced.

**Category → Shipping Type**  
Certain product categories are strongly associated with specific shipping preferences. *Application:* Pre-populate shipping type defaults at checkout based on category to reduce decision friction and improve conversion rate.

---

## Challenges and Decisions

**Class imbalance:** The 3:1 No/Yes imbalance in subscription status was handled consistently: `class_weight='balanced'` in the Decision Tree, stratified train/test splitting, and evaluation using weighted F1 and AUC-ROC rather than raw accuracy alone. A model predicting "No" for all observations would achieve 73% accuracy — these metrics ensure we measure true discriminative ability.

**Clustering feature selection:** Rather than clustering on all 31 one-hot encoded features (which would heavily weight the many dummy columns from OHE), clustering was applied to the 10 most behaviorally interpretable numeric and encoded features. This produces segment profiles that are meaningfully describable in business terms.

**Association rule item design:** Instead of mining over raw column values directly, items were formatted as `Column=Value` strings (e.g., `Category=Clothing`, `Discount=Yes`). This preserves context in each item label, making rules self-documenting and directly interpretable without needing to look up column mappings.

**Support threshold choice:** A minimum support of 0.15 was chosen to filter out rare combinations while still producing a rich set of multi-item rules. Lower thresholds (e.g., 0.05) produced thousands of low-quality rules; higher thresholds (e.g., 0.25) were too restrictive given the even distribution of categories and seasons in this dataset.

**Connecting all three deliverables:** The consistent finding across all three deliverables is that promotional engagement (Discount Applied, Promo Code Used) is the strongest behavioral signal in this dataset — appearing in Deliverable 1 EDA correlations, Deliverable 2 Random Forest feature importances, Deliverable 3 Decision Tree importances, and Deliverable 3 association rules. This convergence from four independent analytical angles strengthens confidence in the finding.
