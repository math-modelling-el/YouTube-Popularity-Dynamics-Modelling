# YouTube Popularity Dynamics Modelling

## Overview

This project studies how YouTube video popularity evolves over time by modelling it as a **second-order discrete dynamical system**.

Unlike traditional approaches that focus only on prediction, this work aims to **interpret the growth behaviour of videos**, answering questions such as:

- Will a video go viral?
- Will it stabilise or decline?
- Does it exhibit rapid spikes or smooth growth?

---

## Motivation

Most existing models (AR, ARIMA, ML models):

- Predict future views based on past values
- Treat the problem as a **black-box prediction task**
- Do not explain *why* a video grows or declines

This project shifts the focus from:

> "What is the next value?"  
> to  
> "What is the **behaviour** of this system?"

---

## Core Model

We model log-transformed video views using a second-order difference equation with momentum terms:

```
log_views(t) = α₁·log_views(t-1) + α₂·log_views(t-2)
             + β₁·log_growth(t-1) + β₂·log_growth(t-2) + ε
```

| Term | Role |
|---|---|
| α₁, α₂ | Position (level) — *where* the video currently is |
| β₁, β₂ | Momentum — *how fast* it was growing |

This captures trend, momentum, and feedback effects in a single interpretable equation.

---

## Stability Analysis

The long-term behaviour is determined by the characteristic equation:

```
r² - α₁·r - α₂ = 0
```

| Root magnitude | Behaviour |
|---|---|
| All \|r\| < 1 | Stable — views saturate over time |
| Any \|r\| ≥ 1 | Unstable — viral/explosive growth |
| Complex roots | Oscillatory — fluctuating attention |

---

## Video Lifecycle Classification

Using the fitted momentum coefficient β₁, each video observation is classified into one of three stages:

| Momentum β₁·ΔV(t-1) | Stage |
|---|---|
| > 0.1 | Acceleration / Peak |
| [−0.1, 0.1] | Moderate Growth |
| < −0.1 | Deceleration / Decline |

---

## Model Comparison

Four models are built and compared on a 70/30 temporal train/test split:

| Model | Features | Estimator |
|---|---|---|
| AR(1) Baseline | V(t-1) | OLS / Ridge |
| AR(2) Baseline | V(t-1), V(t-2) | OLS / Ridge |
| **Proposed (2nd-order + Momentum)** | V(t-1), V(t-2), ΔV(t-1), ΔV(t-2) | OLS / Ridge |
| ARIMA(1,0,1) Benchmark | per-series time series | ARIMA |

Statistical significance of improvements is verified using **paired t-tests**.

---

## Dataset

- **Source:** Kaggle — YouTube Trending Dataset 2025
- **Size:** ~670,000 records across 93 countries
- **Granularity:** Daily snapshots of trending videos
- **Date range:** February 2025 – July 2025

### Features used

| Feature | Description |
|---|---|
| `views` | Raw daily view count |
| `log_views` | log1p-transformed views |
| `log_growth` | First difference of log_views (momentum) |
| `log_acceleration` | Second difference of log_views |
| `log_views_lag1/2` | Lagged position terms |
| `log_growth_lag1/2` | Lagged momentum terms |

> Dataset not included due to size. Place the CSV at `dataset/modified_dataset.csv`.

---

## Limitations

- Captures only the **trending phase** — no pre-trend or long-tail decay
- Global model coefficients may not generalise to all content types
- ARIMA benchmark evaluated on a 100-video subsample only

---

## Project Structure

```
mm_el/
├── dataset/
│   └── modified_dataset.csv      # raw data (not tracked)
├── notebooks/
│   ├── eda.ipynb                  # Phase 1 — EDA & data cleaning
│   ├── model_experiments.ipynb    # Phases 2, 3, 5, 6 — models & analysis
│   └── result_analysis.ipynb      # Results summary
├── outputs/                       # Generated figures and CSVs
│   ├── acf_pacf_analysis.png      # Fig 1 — ACF/PACF validation
│   ├── model_comparison.png       # Fig 2 — model RMSE/MAE/R²
│   ├── video_trajectories.png     # Fig 4 — actual vs predicted
│   ├── lifecycle_distribution.png # Fig 6 — lifecycle stage counts
│   ├── stability_analysis.png     # Fig 7 — characteristic roots
│   ├── model_comparison.csv
│   └── vif_table.csv
├── src/
│   ├── data_pipeline.py
│   ├── model.py
│   ├── evaluation.py
│   └── results.py
├── .venv/                         # Virtual environment (not tracked)
├── requirements.txt
└── README.md
```

---

## Setup

```bash
# Create and activate virtual environment
python -m venv .venv
.venv\Scripts\python.exe -m pip install pandas numpy matplotlib seaborn scikit-learn scipy statsmodels jupyterlab ipykernel

# Register Jupyter kernel
.venv\Scripts\python.exe -m ipykernel install --user --name mm_el --display-name "Python (mm_el)"

# Launch JupyterLab
.venv\Scripts\python.exe -m jupyterlab
```

Then open `notebooks/model_experiments.ipynb` and select the **Python (mm_el)** kernel.
