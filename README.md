## Project Summary
- Forecasted the next day’s 48 half-hour electricity loads, compared against a naive “yesterday’s value” benchmark, then optimised a 220 kW / 440 kWh battery to minimise purchase cost using those forecasts.

## Dataset Understanding
- `assessment_demand_data.csv`: half-hourly `mean_demand_kw`, i.e. the average site power during each 30‑minute slot from 2023‑04‑01 through 2023‑09‑30 23:30.
- `assessment_tariff_data.csv`: price per kWh (pence) for the next 48 slots (2023‑10‑01). Each demand forecast must pair with its tariff to compute grid cost.


## Forecasting Approach
1. **Baseline:** Naive “same slot yesterday” forecast—captures the dominant daily seasonality and serves as the minimum bar for improvement.
2. **Features:** half-hour slot, weekday index, weekend flag, month, sine/cosine encodings for daily & weekly cycles, `lag_48`, `lag_96`, and a 48-step rolling mean.
3. **Split:** Chronological holdout—last 7 days reserved for testing to avoid leakage and mimic real deployment.
4. **Model:** Random Forest Regressor (400 trees for validation, 500 for final fit) for non-linear interactions without heavy tuning.
5. **Metrics:** MAE (interpretable in kW) and RMSE (penalises peaks). RF beat the naive baseline by ~25 % MAE and ~21 % RMSE, demonstrating added value.

## Forecasting Results
- RF MAE/RMSE on the holdout: 9.8 / 13.7 kW vs naive 13.1 / 17.4 kW.
- It handles morning increases and evening drops more accurately than the naive baseline.


## Battery Scheduling Approach
- **Constraints understood up front:** SoE range 0–440 kWh, power limit ±220 kW, perfect efficiency, no export (discharge cannot exceed site demand), initial=final SoE of 220 kWh, 30‑minute time steps.
- **Method:** I solved the dispatch as a linear optimisation problem using PuLP. Variables: 
charge, discharge, SoE per slot. Constraints enforce power bounds, SoE recursion, and 
demand ≥ discharge. Objective minimises tariff-weighted grid import.
- **Outcome:** LP charges during the cheapest overnight periods, discharges through the afternoon peak, and finishes at the required SoE. Daily grid cost drops from ~£1,349 to ~£1,226—about £123 (9 %) saved.

## Repository Contents
- `assessment_solution.ipynb`: python notebook covering EDA, feature engineering, modelling, forecasting, optimisation, and plots.
- `assessment_demand_data.csv` / `assessment_tariff_data.csv`: input datasets.


