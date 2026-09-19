# Armenia Inflation Forecasting — SARIMA vs ARIMAX

## Objective

Assess whether the supplied macroeconomic, external and structural-shock information improves monthly Armenian YoY inflation forecasts beyond a model driven only by inflation's past dynamics.

## Data pipeline

The notebook is one end-to-end project: it reads the supplied raw CPI, USD/AMD, policy-rate, Brent and M2 files; cleans and merges them into `df`; adds the four structural-shock dummies; performs EDA; and uses that same in-memory `df` as the forecasting input. The pipeline produces 187 monthly observations from 2011-01 through 2026-07. `yoy_index` is the target; `USD`, `Policy_Rate`, `M2_YoY`, `Brent`, and four supplied shock dummies are covariates. No missing values or duplicate timestamps were found. Creating FX and Brent percentage changes removes the first observation, leaving 186 usable months (2011-02—2026-07).

## Method

- ADF and KPSS tests are reported for all five level series, then for FX and Brent growth.
- Fixed temporal holdout: train 2011-02—2024-07; test 2024-08—2026-07 (24 months).
- Naive and Seasonal Naive forecasts are evaluated on exactly the same test period.
- A 72-model SARIMA grid searches `(p,d,q)` with `p,q∈{0,1,2}`, `d∈{0,1}`, and `(P,0,Q,12)` with `P,Q∈{0,1}`. Seasonal differencing is excluded because the target is already a 12-month inflation rate. Models within two AIC points of the minimum are compared by BIC.
- ARIMAX uses the selected SARIMA order plus `FX_growth`, `Policy_Rate`, `M2_YoY`, `Brent_growth`, and all supplied dummies.
- Rolling-origin validation uses 12 expanding-window origins (2024-07—2025-06), each producing 1–12 month forecasts. Metrics are reported at horizons 1, 3, 6, and 12.
- Two-sided Diebold–Mariano tests use squared-error loss and a small-sample Harvey adjustment.

## Results

Selected univariate model: **SARIMA(0,1,2)×(0,0,1,12)** (AIC 303.772; BIC 315.706).

| Model | Test MAE | Test RMSE |
|---|---:|---:|
| SARIMA | 0.805 | 0.974 |
| ARIMAX | 1.219 | 1.566 |
| Naive | 1.942 | 2.238 |
| Seasonal Naive | 2.183 | 2.475 |

SARIMA is the best model on the fixed test set. ARIMAX statistically improves on Naive (DM p<0.001), but is significantly worse than SARIMA (DM p=0.008). Rolling-origin RMSE likewise favors SARIMA at 6 months (0.884 vs 1.053) and 12 months (1.219 vs 2.047); ARIMAX is best at the one-month horizon (0.372).

The selected ARIMAX model estimates a statistically significant association for `M2_YoY` (coefficient -0.107, p=0.001), but this is a conditional predictive association, not a causal effect. The estimated `D_oil_2026` coefficient is not identified in the fixed test design because the dummy has no variation in the training sample ending in July 2024.

## Scenario analysis

The 12-month ARIMAX fan chart provides 80% and 95% prediction intervals. Scenarios set a one-off August 2026 monthly FX growth shock (+10%), Brent growth shock (+30%), or both; policy rate and M2 growth are held at their July 2026 values. This is **scenario analysis, not causal inference**. The shocks only affect the first forecast month in this specification because they are specified as one-off monthly growth shocks.

## Limitations

- Macroeconomic data may be revised; no real-time vintage dataset is available.
- Policy rate is endogenous.
- ARIMAX coefficients and scenarios do not establish causality.
- Only a small set of macro variables is included.
- Structural breaks may destabilize relationships.
- Backtests condition on realized future exogenous variables, which overstates a real-time system unless those regressors are forecast separately.

## Reproducibility

Open and run `armenia_inflation_sarima_arimax.ipynb` from the project root. It maps raw inputs to `/Users/invoke/Desktop/Новая папка`, saves its derived dataset to `outputs/final_dataset_from_eda.csv`, and passes the same `df` directly into the forecasting steps. It uses `pandas`, `statsmodels`, `scikit-learn`, `scipy`, and `matplotlib`, and saves the output figures in `figures/`.
