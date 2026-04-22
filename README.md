# MSCS-634: Advanced Big Data and Data Mining

**Author:** Oishani Ganguly  

## Overview

This project applies a full data mining pipeline to the **Customer Shopping Trends Dataset** — a 3,900-record synthetic retail dataset with 19 attributes. The analysis progresses through data cleaning, exploratory analysis, regression modeling, classification, clustering, and association rule mining, culminating in actionable business insights and ethical recommendations.

## Key Finding

**Promotional engagement** (Discount Applied + Promo Code Used) is the dominant behavioral signal in this dataset — consistently identified across four independent analytical methods (EDA, regression feature importances, classification feature importances, and association rule mining).

## Project Structure

- **Deliverable 1:** Data cleaning, validation, and exploratory data analysis (EDA)
- **Deliverable 2:** Regression modeling (Linear, Ridge, Lasso, Random Forest) to predict Purchase Amount
- **Deliverable 3:** Classification (Decision Tree, k-NN), K-Means clustering, and Apriori association rule mining to predict Subscription Status
- **Deliverable 4:** Consolidated pipeline, cross-deliverable synthesis, and business recommendations

## Notable Files in This Repository

```
.
├── deliverable_1.ipynb       # Data cleaning & EDA
├── deliverable_2.ipynb       # Regression models (4 models compared)
├── deliverable_3.ipynb       # Classification, clustering, association rules
├── deliverable_4.ipynb       # Complete synthesis & recommendations
├── shopping_trends.csv       # Original dataset (3,900 rows × 19 columns)
└── README files              # One per deliverable + root overview
```

## Quick Stats

| Metric | Value |
|---|---|
| Dataset size | 3,900 customer records |
| Features (raw) | 19 attributes |
| Features (engineered) | 31 total after OHE |
| Regression models | 4 (Linear, Ridge, Lasso, Random Forest) |
| Classification models | 3 (DT default, DT tuned, k-NN) |
| Clustering segmentation | K-Means (k=optimal) |
| Association rules mined | 100+ rules (support ≥ 0.15, confidence ≥ 0.60) |
| Visualizations | 11 (publication-ready) |

## How to Use

1. **Start with `deliverable_1.ipynb`** for dataset overview and cleaning narrative
2. **Review `deliverable_2.ipynb`** to understand why regression is challenging (near-zero feature correlations)
3. **Explore `deliverable_3.ipynb`** for classification, segmentation, and pattern discovery
4. **Consult `deliverable_4.ipynb`** for the complete consolidated pipeline and final recommendations

All notebooks are fully executable with `shopping_trends.csv` in the same directory.