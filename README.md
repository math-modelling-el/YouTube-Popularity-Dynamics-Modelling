# YouTube Popularity Dynamics Modelling

## Overview

This project studies how YouTube video popularity evolves over time by modelling it as a **second-order discrete dynamical system**. Rather than treating view prediction as a black-box task, the goal is to interpret the growth behaviour of videos — capturing trend, momentum, and feedback effects in a single equation.

Key questions addressed:
- Does a video exhibit rapid acceleration or steady growth?
- When does growth peak and begin to decelerate?
- Is the system dynamically stable over time?

---

## Core Model

Log-transformed view counts are modelled using a second-order difference equation with explicit momentum terms:

```
log_views(t) = α₁·log_views(t-1) + α₂·log_views(t-2)
             + β₁·log_growth(t-1) + β₂·log_growth(t-2) + ε
```

| Term | Role |
|---|---|
| α₁, α₂ | Position — level lags (where the video currently is) |
| β₁, β₂ | Momentum — growth rate lags (how fast it was growing) |

Fitted coefficients: α₁ = 0.9071, α₂ = 0.0919, β₁ = 0.8152, β₂ = −0.0733

---

## Results

### Model Comparison (70/30 temporal train/test split)

| Model | RMSE | MAE | R² | RMSE vs AR(1) |
|---|---|---|---|---|
| AR(1) baseline | 0.0395 | 0.0285 | 0.9996 | — |
| AR(2) baseline | 0.0173 | 0.0095 | 0.9999 | +56.26% |
| **Proposed (2nd-order + Momentum)** | **0.0170** | **0.0091** | **0.9999** | **+57.00%** |
| ARIMA(1,0,1) benchmark | 0.0621 | 0.0450 | 0.9982 | −57.31% |

Improvements of the proposed model over both baselines are statistically significant (paired t-test, p ≈ 0).

### Multicollinearity & Regularisation

VIF analysis revealed perfect multicollinearity among lag features (VIF = ∞ for three of four features), confirming that Ridge Regression (λ = 1.0) is required in place of OLS.

### Video Lifecycle Classification

Each observation is classified by momentum score β₁·ΔV(t-1):

| Stage | Threshold | Unique Videos |
|---|---|---|
| Acceleration / Peak | momentum > 0.1 | 6,688 |
| Moderate Growth | −0.1 to 0.1 | 17,914 |
| Deceleration / Decline | momentum < −0.1 | 4 |

### Stability Analysis

Characteristic equation: r² − 0.9071·r − 0.0919 = 0

| Root | Magnitude | Status |
|---|---|---|
| r₁ | 0.9990 | Stable (< 1) |
| r₂ | 0.0920 | Stable (< 1) |

Both roots lie inside the unit circle — the model predicts bounded, realistic growth trajectories.

---

## Dataset

- **Source:** Kaggle — YouTube Trending Dataset 2025
- **Raw records:** ~670,000 across 93 countries
- **After preprocessing:** 284,855 observations, 57,891 unique video-country pairs
- **Date range:** February 2025 – July 2025
- **Granularity:** Daily snapshots of trending videos

| Feature | Description |
|---|---|
| `log_views` | log1p-transformed cumulative view count |
| `log_growth` | First difference of log_views (momentum proxy) |
| `log_views_lag1/2` | Lagged position terms |
| `log_growth_lag1/2` | Lagged momentum terms |

> Dataset not included due to size. Place the CSV at `dataset/modified_dataset.csv`.

---

## Project Structure

```
mm_el/
├── dataset/
│   └── modified_dataset.csv           # raw data (not tracked)
├── notebooks/
│   ├── eda.ipynb                       # Phase 1 — EDA & data cleaning
│   └── model_experiments.ipynb        # Phases 2–6 — feature engineering,
│                                      #   model building, lifecycle & stability
├── outputs/
│   ├── acf_pacf_analysis.pdf          # ACF/PACF validation plots
│   ├── model_comparison.pdf           # RMSE/MAE/R² bar charts
│   ├── video_trajectories.pdf         # Actual vs predicted trajectories
│   ├── lifecycle_distribution.pdf     # Lifecycle stage distribution
│   ├── stability_analysis.pdf         # Characteristic roots — unit circle
│   ├── model_comparison.csv           # Numeric results table
│   └── vif_table.csv                  # VIF scores per feature
├── .venv/                             # Virtual environment (not tracked)
├── requirements.txt
└── README.md
```

---

## Notebook Phases

| Notebook | Phase | Content |
|---|---|---|
| `eda.ipynb` | 1 | Data loading, cleaning, log transforms, exploratory analysis |
| `model_experiments.ipynb` | 2 | Lag features, ACF/PACF, VIF analysis |
| | 3 | Four-model build & comparison, paired t-tests |
| | 4 | Category-wise analysis (requires `category_id` column) |
| | 5 | Video lifecycle classification |
| | 6 | Stability analysis — characteristic roots |

---

## Setup

```bash
# Create and activate virtual environment
python -m venv .venv
.venv\Scripts\python.exe -m pip install pandas numpy matplotlib scikit-learn scipy statsmodels jupyterlab ipykernel

# Register Jupyter kernel
.venv\Scripts\python.exe -m ipykernel install --user --name mm_el --display-name "Python (mm_el)"

# Launch JupyterLab
.venv\Scripts\python.exe -m jupyterlab
```

Open `notebooks/model_experiments.ipynb` and select the **Python (mm_el)** kernel.

---

## Limitations

- Covers only the **trending phase** — no pre-trend or long-tail decay data
- Global coefficients may not generalise across all content categories
- ARIMA benchmark evaluated on a 100-video subsample only
- Near-zero Deceleration/Decline observations suggest the dataset skews toward actively growing videos
