# Multi-Horizon Electricity Demand Forecasting

End-to-end probabilistic electricity demand forecasting system — data ingestion through Streamlit dashboard. Built and tested on Apple M1 (macOS), Python 3.10.

## Current status

| Phase | Description | Status |
|-------|-------------|--------|
| 0 | Environment & repo setup | ✅ Complete |
| 1 | Data ingestion & preprocessing | ✅ Complete |
| 2 | Exploratory data analysis | ✅ Complete |
| 3 | Classical baselines (Naive, Theta, Prophet, ARIMA) | ✅ Complete |
| 4 | XGBoost quantile regression + SHAP | ✅ Complete |
| 5 | Model comparison & evaluation | 🔜 Next |
| 6 | Hierarchical reconciliation | ⬜ Pending |
| 7 | Probabilistic calibration | ⬜ Pending |
| 8 | Streamlit dashboard | ⬜ Pending |

## Model leaderboard (phases 3–4)

| Model | sMAPE | RMSE (MW) | Notes |
|-------|-------|-----------|-------|
| XGBoost q50 (val) | **1.75%** | 1,168 | 80% PI coverage: 66.5% |
| XGBoost q50 (CV) | 1.85% | 1,167 | 5-fold cross-validation |
| Prophet | 4.22% | 2,708 | CV |
| Theta | 6.89% | 4,714 | CV |
| Seasonal Naive | 9.20% | 6,925 | CV |
| ARIMA | 12.26% | 7,322 | CV, daily-aggregated |

Full CRPS, WQL, and Winkler scores computed in Phase 5.

## Stack

| Layer | Library |
|-------|---------|
| Platform | macOS Apple M1, Python 3.10 |
| Data | OPSD (Germany), Open-Meteo weather |
| Classical models | statsforecast, prophet, pmdarima |
| ML model | XGBoost (quantile regression) |
| Explainability | SHAP (native pred_contribs) |
| Reconciliation | statsforecast (MinT, BU) — Phase 6 |
| Calibration | conformal prediction — Phase 7 |
| Dashboard | Streamlit — Phase 8 |

## Dataset

- **Source:** Open Power System Data (OPSD) — Germany hourly load
- **Weather:** Open-Meteo historical archive (temperature, wind, cloud cover, solar radiation, humidity)
- **Range:** 2019-01-01 → 2020-09-30
- **Rows:** 15,336 hourly observations
- **Missing:** 0 (after imputation)
- **Features:** 30+ engineered (cyclic calendar, lag 24/48/168h, rolling stats, holiday flags)

## Setup

```bash
# Clone and create environment
git clone https://github.com/Trixie18/project-meepo.git
cd project-meepo
conda env create -f environment.yml
conda activate elec_forecast

# Add API keys (optional — OPSD works without a key)
cp .env.example .env   # add EIA_API_KEY if using US data

# Verify setup
python -m pytest tests/ -v
```

## Project structure

```
elec-forecast/
├── data/
│   ├── raw/                        # downloaded source files (gitignored)
│   └── processed/
│       └── features_DE.parquet     # 15,336 rows × 30+ features
├── notebooks/
│   ├── 01_ingestion.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_baselines.ipynb
│   └── 04_xgb.ipynb
├── scripts/
│   └── run_xgb.py                  # XGBoost training (run outside Jupyter on M1)
├── src/
│   ├── ingestion/
│   │   ├── eia_client.py           # US EIA API v2 client
│   │   ├── opsd_client.py          # Open Power System Data downloader
│   │   ├── weather_client.py       # Open-Meteo weather fetcher
│   │   └── preprocessing.py        # feature engineering pipeline
│   ├── models/
│   │   ├── baselines.py            # Seasonal Naive, Theta, Prophet, ARIMA
│   │   ├── cross_validation.py     # expanding-window CV framework
│   │   ├── xgb_model.py            # XGBoost quantile regression + SHAP
│   │   └── forecast_store.py       # parquet save/load utilities
│   ├── metrics/
│   │   └── evaluation.py           # sMAPE, MASE, RMSE, CRPS, Winkler, coverage
│   └── utils/
│       ├── config.py               # paths, constants, device detection
│       └── plot_helpers.py         # shared matplotlib style + save helpers
├── outputs/
│   ├── forecasts/                  # saved parquet forecast files
│   └── plots/                      # saved PNG plots (16 so far)
├── dashboard/                      # Streamlit app (Phase 8)
├── tests/
│   ├── test_config.py
│   ├── test_ingestion.py
│   ├── test_eda.py
│   ├── test_baselines.py
│   └── test_xgb.py
├── environment.yml
├── requirements.txt
├── pyproject.toml
└── README.md
```

## Outputs so far

### Forecast files (`outputs/forecasts/`)

| File | Contents |
|------|----------|
| `features_DE.parquet` | Processed feature table |
| `cv_all_classical.parquet` | Naive + Theta + Prophet + ARIMA CV results |
| `cv_xgb.parquet` | XGBoost 5-fold CV results |
| `cv_xgb_point.parquet` | XGBoost val set point forecasts |
| `xgb_quantile_forecasts.parquet` | XGBoost q10/q50/q90 val set |
| `xgb_shap_values.parquet` | SHAP values for 500 val samples |

### Plots (`outputs/plots/`)

| File | Description |
|------|-------------|
| `01_overview_series.png` | Full series + monthly mean + rolling mean |
| `02_stl_decomposition.png` | STL trend/seasonal/residual (64% seasonal) |
| `03_periodogram.png` | Dominant frequencies (24h, 168h) |
| `04_acf_pacf_daily.png` | ACF/PACF — daily aggregation |
| `05_hourly_acf.png` | Hourly ACF — first 72 lags |
| `06_multiseasonal_heatmaps.png` | Hour × DOW and Hour × Month |
| `07_holiday_effects.png` | Load profiles by day type |
| `08_weather_correlations.png` | Load vs weather covariates |
| `09_correlation_heatmap.png` | Feature correlation matrix |
| `10_model_leaderboard.png` | Classical model sMAPE/MASE/RMSE bars |
| `11_forecast_vs_actual.png` | Forecast vs actual — last CV fold |
| `12_smape_by_horizon.png` | sMAPE degradation by horizon hour |
| `13_residual_distributions.png` | Residual histograms — classical models |
| `14_xgb_quantile_forecast.png` | XGBoost q10/q50/q90 — first 7 val days |
| `15_xgb_feature_importance.png` | XGBoost feature importance |
| `16_shap_summary.png` | SHAP mean absolute values |

## Key findings so far

- **Dominant seasonality:** 64% of variance explained by seasonal component (STL)
- **Key periods:** 24h daily and 168h weekly cycles visible in periodogram
- **Best feature:** `load_mw_lag_168h` (same hour last week) — 20.9% importance
- **XGBoost vs classical:** 1.75% vs 4.22–12.26% sMAPE — clear ML advantage
- **Interval calibration:** 80% PI achieves 66.5% empirical coverage — needs recalibration (Phase 7)

## Known issues / workarounds

- **LightGBM segfault on M1:** replaced with XGBoost (stable ARM64 binary)
- **Jupyter kernel crash on M1:** XGBoost training runs via `python scripts/run_xgb.py` instead
- **SHAP TreeExplainer bug (XGBoost 3.x):** use native `pred_contribs=True` instead
- **OPSD data range:** only 2019–2020 available via direct download; use EIA API for longer US series

## Running tests

```bash
python -m pytest tests/ -v
# Expected: 17 passed
```

## Refreshing model scores

```bash
python -c "
import pandas as pd, numpy as np, os
from src.metrics.evaluation import smape, rmse, coverage

classical = pd.read_parquet('outputs/forecasts/cv_all_classical.parquet')
rows = []
for model, grp in classical.groupby('model'):
    rows.append({'model': model,
                 'smape': round(smape(grp.actual, grp.forecast), 2),
                 'rmse':  round(rmse(grp.actual, grp.forecast), 0)})

xgb = pd.read_parquet('outputs/forecasts/xgb_quantile_forecasts.parquet').dropna(subset=['actual','q50'])
rows.append({'model': 'XGBoost (val)', 'smape': round(smape(xgb.actual, xgb.q50), 2), 'rmse': round(rmse(xgb.actual, xgb.q50), 0)})

print(pd.DataFrame(rows).sort_values('smape').to_string(index=False))
print(f'XGBoost 80% PI coverage: {coverage(xgb.actual, xgb.q10, xgb.q90)*100:.1f}%')
"
```