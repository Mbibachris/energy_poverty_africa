# Drivers of Energy Poverty in Africa: An Algorithmic Approach

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![Status: Analysis Complete — Manuscript in Preparation](https://img.shields.io/badge/Status-Analysis%20Complete%20%E2%80%94%20Manuscript%20in%20Preparation-yellow)
![Language: Python](https://img.shields.io/badge/Language-Python-3776AB)
![Language: Stata](https://img.shields.io/badge/Language-Stata-1A5276)

> **Private working repository.** This project is an unpublished working paper submitted for journal review. The manuscript, figures and results tables are kept out of this repository and are not shared here. This README documents only the code, data pipeline and how to reproduce it.

---

## Overview

This repository contains the data-processing and modelling code for a study on multidimensional energy poverty across African countries, combining:

1. **Index construction** — a Multidimensional Energy Poverty Index (MEPI) built with the Alkire–Foster (2011) dual-cutoff methodology.
2. **Algorithmic driver identification** — Recursive Feature Elimination followed by nine machine learning models, interpreted with feature importance, SHAP values, Partial Dependence Plots and Individual Conditional Expectation curves.
3. **Econometric validation** — panel fixed-effects/random-effects estimation with a Hausman test.

The project began as a Sub-Saharan Africa study and has since been extended to cover a wider panel of African countries.

---

## Repository Structure

```
energy_poverty_africa/
│
├── data/
│   ├── raw/                       ← Raw World Bank WDI extract
│   └── processed/                 ← Cleaned panel data and MEPI-linked datasets
│
├── notebooks/
│   ├── 01_data_cleaning/          ← Missing-data handling and merging of raw WDI data
│   ├── 02_mepi_construction/      ← MEPI construction and validation (Alkire–Foster)
│   ├── 03_driver_analysis/        ← See note below — merged into 04_results
│   └── 04_results/                ← EDA, RFE, ML models, lag analysis, interpretability
│
├── stata/
│   └── panel_analysis.do          ← Panel FE/RE estimation and Hausman test
│
├── requirements.txt
└── README.md
```

**Note on `03_driver_analysis/`:** Recursive Feature Elimination and the machine-learning models are implemented together with the interpretability analysis in a single notebook, `04_results/Modelling.ipynb`, rather than as a separate stage. The `03_driver_analysis/` folder is kept for structural clarity but its content lives in `04_results/`.

**Not included in this repository:** the manuscript draft and generated figures/results tables are kept local-only until the paper is published, to avoid pre-publication disclosure of results.

---

## Data

All data are sourced from the **World Bank World Development Indicators (WDI)** database, covering a multi-decade panel of African countries.

| Category | Variables | Role |
|---|---|---|
| MEPI construction | 13 indicators across 5 dimensions | Dependent variable components |
| Driver variables | 34 candidate variables across 6 theoretical traditions | Independent variables |

---

## Methodology

**MEPI construction:** built across five dimensions — electricity access, affordability & efficiency, reliability, clean cooking, and sustainability — each with equal weight, using the Alkire–Foster (2011) dual-cutoff method (indicator-level cutoff zⱼ = 1/3, poverty cutoff k = 1/3, MEPI = H × A).

**Driver identification:** 34 candidate driver variables spanning six theoretical traditions (economic & income, natural resource & fiscal, institutional & governance, demographic & spatial, environmental, structural energy mix) are narrowed via Recursive Feature Elimination (Random Forest estimator) to a smaller final set used in the machine learning and panel models.

**Machine learning models:** LASSO, Ridge, Elastic Net, Decision Tree, Random Forest, Gradient Boosting, XGBoost, Support Vector Regression, and K-Nearest Neighbours — tuned with 5-fold, country-blocked `GridSearchCV` (`GroupKFold`) on a temporally-ordered train/test split, with feature selection restricted to the training partition. Interpreted using feature importance, SHAP (TreeExplainer), Partial Dependence Plots, and Individual Conditional Expectation curves. Temporal robustness is checked with a one-year-lag test and a rolling, expanding-window holdout.

**Panel econometrics:** `stata/panel_analysis.do` estimates fixed-effects and random-effects models on the selected drivers, applies log/IHS transformations for skewed variables, and uses a Hausman (1978) test to choose between FE and RE. Also reports a two-way (country + year) specification, a Pesaran cross-sectional-dependence test, Driscoll–Kraay standard errors, and a fractional response model as robustness checks.

See [REPRODUCIBILITY.md](REPRODUCIBILITY.md) for software versions, the random seed, the exact cross-validation scheme, and full file provenance.

---

## Replication

Run notebooks in order:
```
01_data_cleaning        → produces cleaned panel data
02_mepi_construction    → produces country-year MEPI scores
04_results              → RFE, ML models, lag analysis, interpretability
stata/panel_analysis.do → FE/RE estimation and Hausman test (run from within stata/)
```

### Requirements
```bash
pip install -r requirements.txt
```

---

## Progress

- [x] Data cleaning and merging
- [x] MEPI construction (Alkire–Foster)
- [x] MEPI merged with driver variables
- [x] Recursive Feature Elimination and multi-model comparison
- [x] Interpretable ML (feature importance, SHAP, PDP, ICE)
- [x] Panel FE/RE estimation and Hausman validation
- [x] Manuscript drafted (private — submitted for review)
- [ ] Peer review / journal publication

---

## Contact

Christopher Mbiba
University of Cape Coast, Ghana
GitHub: [@Mbibachris](https://github.com/Mbibachris)

---

## Citation

A citation will be added once the manuscript is published. Please contact the author before citing or reusing this work.

---

## License

This project is licensed under the MIT License. Note that the MIT License applies to the code in this repository; it does not extend to any manuscript, figures or results, which are not included here.
