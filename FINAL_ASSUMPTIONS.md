# Final Model Assumptions

## Scope

This document records the assumptions used by `quantum_model.ipynb` for the quantum maize-yield experiment. It is an assumptions and limitations record, not a claim that the quantum model outperforms a classical model.

## Target and geography

- Target: annual maize yield in tonnes per hectare (`yield_t_ha`).
- Counties: Bomet, Bungoma, Elgeyo/Marakwet, Kakamega, Nakuru, Nandi, Narok, Trans Nzoia, and Uasin Gishu.
- Primary reporting counties: Bungoma and Nandi.
- Supplementary reporting: the other seven counties.

## Data assumptions

- Yield data comes from both sheets of `Annual Maize Yield Production 2012-2020.xlsx`.
- Sheet1 is parsed as the specified wide three-column year-block layout.
- Sheet4 is parsed as the specified long layout and pivoted by county and year.
- Duplicate county-year yield records are resolved by keeping the Sheet4 row first, matching the notebook pipeline.
- 2019 is absent from both yield sheets and is not imputed.
- Rainfall files are discovered by the filename prefix and normalized with the supplied county-name map.
- Rainfall headers begin one row below the Excel file header and are loaded with `skiprows=1`.
- Missing or non-numeric rainfall values are excluded from the rainfall aggregates.
- A county-year is retained only when both yield and rainfall features are available.

## No-leakage assumptions

- Only rainfall dated on or before 30 June of the same year is used to predict that year's yield.
- Rainfall after 30 June is excluded, including for years with longer source files.
- The rainfall anomaly baseline is each county's mean cumulative rainfall over training years only.
- `StandardScaler` is fitted on training rows only and then applied to validation, test, and future rows.
- Yield values are used as labels only; harvested area and production are not model features.

## Features

The four quantum features are:

1. `cum_rainfall_30jun`: cumulative rainfall through 30 June.
2. `rainy_days_30jun`: number of days with rainfall at least 1 mm through 30 June.
3. `longest_dry_spell`: longest consecutive run of days below 1 mm through 30 June.
4. `rainfall_anomaly`: cumulative rainfall minus the county's training-years-only mean.

## Evaluation assumptions

- The intended chronological split is training years <= 2017, validation year 2018, and test year 2020.
- Because the rainfall sources begin in 2016, the actual merged panel contains 36 rows: four years per county for 2016, 2017, 2018, and 2020.
- Therefore the executed split is 18 training rows, 9 validation rows, and 9 test rows.
- The 2020 test set has only one row per county, so Bungoma and Nandi MAE/RMSE are single-observation errors and R2 is undefined.
- Lambda is selected from `[0.01, 0.1, 1.0, 10.0]` using validation MAE; it is not hardcoded.

## Quantum model assumptions

- Encoding: four-qubit Qiskit `ZZFeatureMap`, two repetitions, full entanglement.
- Kernel: `FidelityQuantumKernel`.
- Regression: explicit kernel ridge linear algebra using `np.linalg.solve` for training coefficients and a kernel matrix multiplication for predictions.
- The main evaluation uses the statevector simulator path.
- Robustness uses `ComputeUncompute` with `StatevectorSampler`, 1024 shots, and seeds 7, 17, 27, 37, and 47.
- No optimizer is used because ridge coefficients are solved in closed form.

## 2021-2026 forecast assumptions

- Future forecasts require rainfall observations through 30 June for the requested year.
- The forecast cell emits a county-year only when that rainfall slice exists.
- Bungoma rainfall coverage extends through 30 June 2026, so Bungoma can be forecast for 2021–2026.
- Nandi's supplied rainfall file ends in 2020, so Nandi has no data-supported 2021–2026 forecast under this pipeline. Missing Nandi years must remain gaps, not fabricated values.
- Future rainfall anomaly values use the historical training baseline; future yields are unavailable for evaluation.
- The 2021–2026 graph is therefore a conditional rainfall-driven model projection, not an observed-yield comparison.

## Reporting and claims

- Report MAE, RMSE, R2, seed-to-seed MAE mean and standard deviation, and runtime for this quantum experiment.
- Do not claim quantum superiority over classical methods until the separate classical notebook supplies a comparable result.
- The notebook retains the placeholder: **Comparison with classical baseline (MAE = ___, pending Samuel Muoria's notebook)**.
