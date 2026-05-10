# Multi-Horizon Electricity Demand Forecasting

End-to-end probabilistic electricity demand forecasting system covering
data ingestion, EDA, classical baselines, Temporal Fusion Transformer,
hierarchical reconciliation, calibration, and a Streamlit dashboard.

## Stack
- **Platform**: macOS (Apple M1), Python 3.11
- **Models**: Seasonal Naive, Theta, Prophet, ARIMA, TFT
- **Deep learning**: PyTorch (MPS), pytorch-forecasting, Lightning
- **Tuning**: Optuna
- **Reconciliation**: statsforecast (MinT, BU)
- **Dashboard**: Streamlit

## Setup
\```bash
conda env create -f environment.yml
conda activate elec-forecast
cp .env.example .env   # add your EIA_API_KEY
pytest tests/ -v
\```

## Project structure
\```
data/           # raw + processed (gitignored)
notebooks/      # EDA + experiments
src/            # reusable modules
outputs/        # forecasts, plots, calibration reports
dashboard/      # Streamlit app
tests/
\```

## Phases
| Phase | Description |
|-------|-------------|
| 0 | Environment & repo setup ✅ |
| 1 | Data ingestion & preprocessing |
| 2 | Exploratory data analysis |
| 3 | Baseline & classical models |
| 4 | Temporal Fusion Transformer |
| 5 | Model comparison & evaluation |
| 6 | Hierarchical reconciliation |
| 7 | Probabilistic calibration |
| 8 | Streamlit dashboard |
