# QS-Agric-T3: Early Maize-Yield Intelligence

## Purpose

This prototype supports government agricultural planning in Kenya by estimating end-of-season maize yield early enough to inform decisions during the season.

Kenya already has substantial agricultural statistics, monitoring systems, and official production processes. This project does **not** claim that Kenya lacks maize-production information or prediction models. The gap being explored is narrower and operational: providing a sufficiently early, locally relevant county-level estimate of end-of-season yield using only information available by mid-June.

The 2025 national reporting context highlights continuing challenges with agricultural-data quality, availability, and fragmentation. Official statistics are produced through annual county data collection and validation processes; this prototype is intended to complement those processes with an earlier analytical signal, not replace official statistics.

## Decision use

Early yield estimates can help public agencies and partners plan:

- food-security monitoring and early-warning actions;
- maize market, storage, and logistics requirements;
- targeted extension and resilience support;
- county-level preparedness before final harvest statistics are available.

The outputs should be treated as planning evidence with uncertainty, not as official estimates or guaranteed outcomes.

## Data and leakage rule

The model uses the supplied maize-yield workbook and county rainfall workbooks in `data/Agr data/`. Rainfall is aggregated only through **30 June of the prediction year**. July and later observations are excluded to preserve the mid-season information boundary.

Features are:

- cumulative rainfall through 30 June;
- number of rainy days, using rainfall of at least 1 mm;
- longest consecutive dry spell, using rainfall below 1 mm;
- rainfall anomaly relative to the same county's training-years-only mean.

Yield data is available for 2012–2018 and 2020; 2019 is missing from both source sheets and is not imputed. After merging with the supplied rainfall data, the executed panel contains 36 rows covering 2016–2018 and 2020 for nine counties. The chronological evaluation is training through 2017, validation in 2018, and testing in 2020.

## Quantum experiment

`quantum_model.ipynb` uses a four-qubit Qiskit `ZZFeatureMap` and `FidelityQuantumKernel`. Kernel ridge regression is solved explicitly with linear algebra. The ridge parameter is selected by validation MAE from `[0.01, 0.1, 1.0, 10.0]`.

The robustness check repeats the kernel computation with real shot noise using `ComputeUncompute` and `StatevectorSampler`, 1,024 shots, and seeds 7, 17, 27, 37, and 47.

## Results and files

- [Quantum notebook](quantum_model.ipynb): complete data pipeline, model, evaluation, plots, and exports.
- [2020 predictions CSV](quantum_predictions.csv): actual and predicted yield for the nine test counties.
- [Bungoma and Nandi 2020 comparison table](bungoma_nandi_2020_comparison.html): viewable and printable HTML table.
- [Comparison table CSV](bungoma_nandi_2020_comparison.csv): spreadsheet-friendly version.
- [Bungoma 2020 chart](actual_vs_predicted_Bungoma.png)
- [Nandi 2020 chart](actual_vs_predicted_Nandi.png)
- [2021 forecast CSV](quantum_forecasts_2021.csv)
- [2021 forecast chart](bungoma_nandi_forecast_2021.png)
- [Final assumptions](FINAL_ASSUMPTIONS.md): detailed assumptions and limitations.

The 2020 test results are based on one observation per county. Therefore Bungoma and Nandi's 2020 MAE values are single-observation errors, and county-level R2 is undefined.

## Forecast coverage limitation

Bungoma's supplied rainfall file extends through 30 June 2026. The supplied Nandi rainfall file ends in 2020. Consequently, the pipeline can produce a rainfall-supported 2021 forecast for Bungoma, but it must report Nandi's 2021 forecast as unavailable unless rainfall through 30 June 2021 is supplied. Substituting 2020 rainfall would be a scenario based on 2020 conditions, not a genuine 2021 observed-input forecast.

## Reproducibility

1. Open the repository in VS Code.
2. Select `\.venv\Scripts\python.exe` as the notebook kernel.
3. Open `quantum_model.ipynb`.
4. Run all cells from top to bottom.

The notebook checks for required packages and installs missing packages into the active kernel. It writes prediction CSV files and PNG charts to the repository root.

## Claim discipline

This is an experimental quantum model report. It describes the observed quantum-model metrics, runtime, and seed robustness only. It does not claim that quantum methods outperform classical methods. Comparison with the separate classical baseline remains pending until that notebook provides comparable results.
