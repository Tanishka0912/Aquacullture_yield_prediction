#  Aquaculture Yield Prediction

Predicting **harvest yield per acre (kg)** of aquaculture ponds using Linear Regression and regularized regression (Ridge, Lasso, ElasticNet), with full data cleaning, EDA and regression diagnostics.

---

##  Project Overview

Aquaculture farmers need to know which pond and management factors actually move harvest. This project:

1. Cleans a pond-level dataset (duplicates, missing values)
2. Explores it with univariate and bivariate analysis
3. Builds and evaluates regression models to predict `harvest_kg_per_acre`
4. Checks the regression assumptions and interprets the drivers of yield

**Target variable:** `harvest_kg_per_acre`

---

##  Dataset

- **Rows / columns:** 2,302 × 22 (2,300 rows after removing duplicates)
- **Granularity:** one row per pond

| Group | Columns |
|---|---|
| Identifier | `pond_id` (not used as a feature) |
| Location / type | `state`, `species`, `water_type` |
| Pond & stocking | `pond_area_acres`, `avg_depth_m`, `pond_age_yrs`, `stocking_density_per_acre`, `seed_size_g`, `culture_days` |
| Feeding & equipment | `total_feed_kg_per_acre`, `feed_type`, `aerators_installed` |
| Water quality | `avg_water_temp_c`, `dissolved_oxygen_mg_l`, `ph`, `ammonia_mg_l`, `water_exchange_pct_per_week` |
| Management | `disease_outbreak`, `probiotic_use`, `farmer_experience_yrs` |
| **Target** | `harvest_kg_per_acre` |

**Data quality issues found and fixed**
- 2 exact duplicate rows → removed
- Missing values in 3 columns: `dissolved_oxygen_mg_l` (63), `ammonia_mg_l` (69), `water_exchange_pct_per_week` (47) → filled with the **mean** (continuous columns) and **random sampling** from the existing distribution (discrete column)

---

##  Methodology

| Step | What was done |
|---|---|
| 1. Data cleaning | Null and duplicate checks, duplicate removal, missing-value imputation |
| 2. EDA | Countplots / pie charts, KDE plots, bar plots and scatter plots against the target, correlation heatmap |
| 3. Encoding | One-hot encoding (`drop_first=True`). `water_type` was dropped because it is fully determined by `species` (all brackish ponds are Vannamei Shrimp), which would cause perfect multicollinearity |
| 4. Split & scaling | 70/30 train-test split (`random_state=57`); `StandardScaler` fit on the training set only |
| 5. Modeling | OLS (statsmodels) and Linear Regression (scikit-learn) as baseline |
| 6. Diagnostics | Linearity, VIF (multicollinearity), Breusch–Pagan (heteroscedasticity), Durbin–Watson (autocorrelation), residual normality |
| 7. Regularization | Ridge, Lasso and ElasticNet tuned with 5-fold cross-validation |

---

##  Results

| Metric | Value |
|---|---|
| Test R² | **0.822** |
| Train R² | 0.841 |
| 5-fold CV R² | 0.833 ± 0.014 |
| Test RMSE | ≈ 604 kg/acre (about 18% of the mean harvest of 3,302) |
| Test MAE | ≈ 432 kg/acre (about 13% of the mean) |

Ridge, Lasso and ElasticNet scored the same as plain Linear Regression (test R² ≈ 0.822), so regularization gave no real gain. The model is not overfitting and multicollinearity is low.

**Diagnostics**
- All VIF values < 2.5 (after removing the redundant `water_type`)
- Durbin–Watson ≈ 2.0 → no autocorrelation
- Breusch–Pagan p ≈ 5e-41 → **heteroscedasticity is present** (error spread grows for high-yield ponds); residuals are also somewhat heavy-tailed

---

##  Key Findings

- **Feed per acre** and **stocking density** are the strongest drivers of harvest (correlations 0.71 and 0.65). About +435 kg harvest per extra 1,000 kg of feed per acre, and about +138 kg per extra 1,000 seeds per acre, other factors held equal.
- **Species** matters most among categories: Pangasius yields about 2,200 kg/acre more than Catla (the baseline).
- A **disease outbreak** lowers harvest by about 350 kg/acre.
- **Dissolved oxygen**, **aerators** and **probiotic use** have small positive effects.
- **pH, water temperature, seed size, pond age, pond area and farmer experience** have no significant effect.
- **Ammonia** is positively correlated with harvest (0.25), but is not significant once feed is controlled for (p ≈ 0.055). The raw link most likely reflects heavier feeding.

> These results show associations, not proven cause and effect.

---

## ⚠️ Limitations & Next Steps

- Linear Regression explains about 82% of the variance. In a quick extra test, tree-based models scored higher (Random Forest R² ≈ 0.88, Gradient Boosting ≈ 0.87), suggesting some non-linear effects.
- Heteroscedasticity means p-values and confidence intervals should be read with caution; robust standard errors (e.g. HC3) are recommended.
- Possible extensions: tree-based models, interaction terms, outlier treatment, and feature selection.

---

## 🛠️ Tech Stack

Python · pandas · NumPy · matplotlib · seaborn · plotly · statsmodels · scikit-learn

---

## 🚀 How to Run

1. Clone or download this repository.
2. Install the requirements:
   ```bash
   pip install pandas numpy matplotlib seaborn plotly statsmodels scikit-learn jupyter
   ```
3. Keep the dataset CSV in the same folder as the notebook (or update the file path in the first code cell that reads the CSV).
4. Open the notebook and use **Kernel → Restart & Run All**:
   ```bash
   jupyter notebook Aquaculture_Yield_Prediction.ipynb
   ```

A fixed seed (`57`) is used for the random fills and the train-test split, so results are reproducible.

---

## 📁 Repository Structure

```
├── Aquaculture_Yield_Prediction.ipynb   # Full analysis: cleaning, EDA, modeling, diagnostics
├── aquaculture_yield.csv                # Dataset (2,302 rows × 22 columns)
└── README.md
```


