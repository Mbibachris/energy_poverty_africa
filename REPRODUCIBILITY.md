# Reproducibility Guide

This file documents everything needed to reproduce the pipeline in this repository: software versions, the random seed, the cross-validation scheme, the final driver set, and which notebook produces which file. It intentionally contains no results, coefficients, or findings — see the note in the main [README](README.md) on why those are kept out of this repository.

## Software versions

| Package | Version |
|---|---|
| Python | 3.10 |
| pandas | 2.3.3 |
| numpy | 2.2.6 |
| scipy | 1.15.3 |
| scikit-learn | 1.7.2 |
| xgboost | **2.1.4** (pinned — xgboost 3.x's model serialization breaks `shap`'s `TreeExplainer`) |
| shap | 0.49.1 |
| Stata | 18 (do-file also compatible with Stata 17+) |

Install with:
```bash
pip install -r requirements.txt
```

## Random seed

`random_state = 42` is used consistently for: the train/test split, `RandomForestRegressor` (both the RFE estimator and the model itself), all 9 tuned models, and the deterministic group ordering used by `GroupKFold` (which itself takes no seed).

## Cross-validation scheme

- **Feature selection (RFE)** is fit on the training partition only — never on the held-out test set. The train/test split happens *before* scaling and RFE, not after, so feature selection cannot see the test set.
- **Hyperparameter tuning** uses `GroupKFold(n_splits=5)`, grouped by country, so no country's observations appear in both the training and validation fold of the same split.
- **Rolling time-based holdout**: an expanding-window evaluation over the last four years of the panel, using each model's already-tuned hyperparameters (no re-tuning per window) — reported alongside the one-year-lag robustness check.

## Final driver set (12 variables: RFE-selected + theoretical additions)

- RFE-selected (10, fit on the training partition only): `NPI`, `GEFF`, `POP`, `POPD`, `RPOP`, `UPOP`, `CO2PC`, `CO2CH`, `FWPC`, `ALTNUC`
- Added on theoretical grounds (2, not RFE-selected): `GDPG`, `FFUEL`

**Note on `UPOP`:** the panel econometric regression (`stata/panel_analysis.do`) drops `UPOP` from the estimated equation because `RPOP + UPOP = 100` in every country-year — the two are exact complements, so once `RPOP` is in the model `UPOP` is perfectly collinear with it and the constant. The regression therefore uses 11 of the 12 drivers. This is not an issue for the machine learning models, which can register separate importance for both via non-linear splits.

## Notebook execution order and file provenance

Run in this order:

```
notebooks/01_data_cleaning/Data_cleaning_missing_data_handling.ipynb
    → data/processed/ENERGY_POVERTY_CLEANED.xlsx
    → data/processed/bounds_correction_log.csv
    → data/processed/missingness_treatment_log_revised.csv

notebooks/02_mepi_construction/MEPI_Construction.ipynb
    → data/processed/mepi_with_drivers.xls
    → data/processed/mepi_country_summary.xls
    → data/processed/mepi_k_robustness.csv
    → data/processed/mepi_driver_overlap_check.csv

notebooks/04_results/Modelling.ipynb
    → data/processed/stata_panel_data.csv
    → outputs/figures/*.png, outputs/tables/*.csv (local-only, gitignored)

stata/panel_analysis.do   (run from within stata/, or let it cd there itself)
    → reads data/processed/panel_data.dta
    → panel_analysis.log (gitignored)
```

`data/processed/panel_data.dta` is rebuilt from `stata_panel_data.csv` with short Stata-style variable names and labels; it is not regenerated automatically by any notebook and should be kept in sync manually if the driver set changes.

## Data quality note

Two "share of total" driver variables (bounded to [0, 100] by definition) were found to contain out-of-range values in the raw WDI extract itself: `Fossil fuel energy consumption (% of total)` and `Electricity production from renewable sources, excluding hydroelectric (% of total)`. These are treated as data errors — set to missing and imputed by the same staged mean/KNN procedure used for all other missing values, with a post-imputation clip to [0, 100] as a safety net. See `data/processed/bounds_correction_log.csv` for the exact affected observations. No MEPI indicator is affected by this correction.

## Country list

See the `Country Name` column of `data/processed/stata_panel_data.csv` or `data/processed/mepi_with_drivers.xls` for the exact 54-country list and spelling used throughout.
