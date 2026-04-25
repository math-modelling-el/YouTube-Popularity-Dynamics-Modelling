# YouTube Popularity Dynamics Modelling

## Overview

This project studies how YouTube video popularity evolves over time by modeling it as a **second-order dynamical system**.

Unlike traditional approaches that focus only on prediction, this work aims to **interpret the growth behavior of videos**, answering questions such as:

- Will a video go viral?
- Will it stabilize or decline?
- Does it exhibit rapid spikes or smooth growth?

---

## Motivation

Most existing models (AR, ARIMA, ML models):

- Predict future views based on past values
- Treat the problem as a **black-box prediction task**
- Do not explain *why* a video grows or declines

This project shifts the focus from:

> ❌ “What is the next value?”  
to  
> ✅ “What is the behavior of this system?”

---

## Core Idea

We model video views using a second-order relation:

\[
V(t) = a * V(t-1) + b * V(t-2)
\]

Although mathematically similar to an AR(2) model, we reinterpret it as a **discrete dynamical system**.

This allows us to capture:

- **Trend** → growth or decline  
- **Momentum** → acceleration or deceleration  
- **Feedback effects** → recommendation-driven amplification  

---

## ⚙️ Stability Analysis

We analyze long-term behavior using the characteristic equation:

\[
r^2 - a r - b = 0
\]

Depending on the roots, the system exhibits:

| Behavior | Interpretation |
|--------|--------------|
| Stable | Views saturate over time |
| Unstable | Viral growth (explosion) |
| Oscillatory | Fluctuating attention patterns |

---

## Connection to YouTube Recommendation System

Video growth is driven by feedback loops:

### Positive Feedback
More views → more engagement → more recommendations → more views  
➡ Leads to **growth and acceleration**

### Negative Feedback
Low engagement → reduced reach → fewer views  
➡ Leads to **decline or stabilization**

---

## Dataset

- **Source:** Kaggle (YouTube Trending Dataset 2025)
- **Size:** ~670,000 records
- **Granularity:** Daily snapshots of trending videos
- **Countries:** US, IN, IL, IQ, IS, etc.

### Features Used

- Views (derived)
- Likes
- Comments
- Country
- Published Date
- Fetched Date

⚠️ **Note:** Dataset is not included due to size constraints.

---

## Limitation

- Only captures the **trending phase** of videos  
- Does not include full lifecycle (pre-trending or long-term decay)

---

## 📁 Project Structure
```
project/
├── data/
├── notebooks/
│ ├── 01_eda.ipynb
│ ├── 02_model_experiments.ipynb
│ └── 03_results_analysis.ipynb
├── src/
│ ├── data_pipeline.py
│ ├── model.py
│ └── evaluation.py
├── figures/
├── paper/
└── README.md
```


