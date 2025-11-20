## Project Summary
- Forecast next-day (48 half-hour) electricity demand, benchmarked against a naive “yesterday’s load” baseline.
- Optimise a 220 kW / 440 kWh battery schedule using forecasted demand plus published tariffs to minimise cost.
- Approach combines engineered time-series features, a Random Forest forecaster, and a deterministic linear program built in PuLP.

## Dataset Understanding
- `assessment_demand_data.csv`: half-hourly mean site demand (`mean_demand_kw`) from 2023‑04‑01 to 2023‑09‑30 23:30.
- `assessment_tariff_data.csv`: price per kWh (pence) for each half-hour from 2023‑10‑01 00:00 to 23:30.


## Forecasting Approach
1. **Baseline:** The naive baseline uses the demand from the same half-hour slot on the previous day. It’s a sensible first check, since commercial buildings tend to follow a similar daily pattern.
2. **Features:** half-hour slot, weekday index, weekend flag, month, sine/cosine encodings for daily & weekly cycles, `lag_48`, `lag_96`, and a 48-step rolling mean.
3. **Split:** Chronological holdout—last 7 days reserved for testing to avoid leakage and mimic real deployment.
4. **Model:** Random Forest Regressor (400 trees for validation, 500 for final fit) for non-linear interactions without heavy tuning.
5. **Metrics:** MAE (interpretable in kW) and RMSE (penalises peaks). RF beat the naive baseline by ~25 % MAE and ~21 % RMSE, demonstrating added value.

## Forecasting Results
- RF MAE/RMSE on the holdout: 9.8 / 13.7 kW vs naive 13.1 / 17.4 kW.
- It handles morning increases and evening drops more accurately than the naive baseline.
- Limitations: no external drivers, so sudden operational changes would still surprise the model, but for a stable commercial site the next-day forecast is reliable.

## Battery Scheduling Approach
- **Constraints:** 0–440 kWh SoE, ±220 kW power, perfect efficiency, no export, SoE(0) = SoE(48) = 220 kWh, 30‑minute steps.
- **Method:** I solved the dispatch as a linear optimisation problem using PuLP. Variables: charge, discharge, SoE per slot. Constraints enforce power bounds, SoE recursion, and demand ≥ discharge. Objective minimises tariff-weighted grid import.
- **Result:** Optimal plan charges overnight, discharges through the afternoon peak, and saves ~£123/day (~9 %) versus running without the battery.

## Repository Contents
- `assessment_solution.ipynb`: end-to-end notebook covering EDA, feature engineering, modelling, forecasting, optimisation, and plots.
- `assessment_demand_data.csv` / `assessment_tariff_data.csv`: input datasets.



