# ATM Cash Demand Forecasting & Replenishment Optimization — DataFest 2024

**Stack:** Python · skforecast · LightGBM · scikit-learn · Google OR-Tools · Jupyter

> A team submission to **DataFest 2024**, a national data science competition organized by **Banco de Crédito del Perú (BCP)** and **Universidad ESAN**, tasking student teams with forecasting ATM cash demand and optimizing replenishment schedules under real operational constraints.

**Team:** Grupo 10 · 5 members · Academic Coordinator: Patricia Reyes Silva

---

## Table of Contents

- [My Contribution](#my-contribution)
- [Problem Statement](#problem-statement)
- [Dataset](#dataset)
- [Skills Demonstrated](#skills-demonstrated)
- [Tech Stack](#tech-stack)
- [Methodology & Results](#methodology--results)
- [Team](#team)

---

## My Contribution

This was a 5-person team competition. **I individually owned the forecasting workstream** — the full modeling notebook (`Forecasting_global.ipynb`): exploratory analysis, feature engineering, multi-series vs. uni-series model comparison, hyperparameter search, and backtesting. The replenishment optimization notebook (`optimized.ipynb`, Google OR-Tools constraint programming) was built as a team effort on top of my forecasting output. I'm flagging this split explicitly so the technical depth below is attributed to the right part of the project.

---

The competition's objective was to forecast daily cash demand at individual ATMs and use those forecasts to schedule cash replenishment in a way that minimizes operational cost — combining a genuine time-series forecasting problem with a downstream constrained-optimization problem, evaluated end-to-end.

## Problem Statement

Two ATM types (A and B) draw down cash daily and must be replenished before running out, but replenishment isn't free or unconstrained:

- Each ATM type has a fixed cash capacity and can only be replenished on specific days of the week.
- Replenishment can happen at most once per day per ATM.
- The forecasting and optimization problems are coupled: a forecast that's off by a wide margin either forces unnecessary replenishment trips (added cost) or risks an ATM running dry (service failure) — so accuracy in the forecasting stage directly determines feasibility in the optimization stage downstream.

## Dataset
Historical daily transaction data (`Datafest2024_Train.csv`, `Datafest2024_Test.csv`) covering per-ATM cash demand, transaction dates, ATM type (A/B), and end-of-day balances, with clear weekly and monthly seasonal patterns across ATMs.

> **Note:** the raw dataset is not included in this repository. DATAFEST 2024's official rules explicitly prohibited extracting the dataset from the competition environment by any means, so it's omitted here by design, not by oversight. The notebooks reference the expected file paths for anyone with legitimate access to the original data.

---

## Skills Demonstrated

| Area | Where it shows up |
|---|---|
| Time series forecasting at scale | `skforecast` — 20+ ATMs forecast simultaneously, comparing per-series (`ForecasterAutoreg`) vs. shared multi-series (`ForecasterAutoregMultiSeries`) models |
| Gradient boosting for regression | `LGBMRegressor`, `HistGradientBoostingRegressor` — both benchmarked head-to-head per ATM type |
| Bayesian hyperparameter optimization | `bayesian_search_forecaster` — tuned `n_estimators`, `max_depth`, `learning_rate`, `min_data_in_leaf`, lag windows, per model/ATM-type combination |
| Time-aware feature engineering | Cyclical sine/cosine encoding for weekly/monthly seasonality; calendar-derived exogenous features (day-of-week, week-of-year, peak-demand-day flags) |
| Proper time series validation | Chronological train/validation/test split (no shuffling), backtesting rather than a single holdout metric |
| Rigorous model comparison | MAE computed per ATM, aggregated with mean/min/max improvement to quantify multi-series vs. uni-series trade-offs rather than reporting a single blended number |
| Constraint programming / operations research | Google OR-Tools `CP-SAT` — replenishment scheduling under capacity limits, day-of-week eligibility, and single-replenishment-per-day constraints, minimizing total replenishment cost |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Forecasting | Python, `skforecast`, `LightGBM`, `scikit-learn` (`HistGradientBoostingRegressor`) |
| Hyperparameter search | Bayesian optimization via `skforecast.bayesian_search_forecaster` |
| Optimization | Google `OR-Tools` (CP-SAT solver) |
| Data handling & viz | Pandas, NumPy, Matplotlib, Seaborn, Plotly |
| Environment | Jupyter Notebook |

---

## Methodology & Results

### Forecasting (individual work)
For each ATM type (A and B), the notebook builds and compares two forecasting strategies:

1. **Uni-series**: one independent model fit per ATM.
2. **Multi-series**: a single shared model (`ForecasterAutoregMultiSeries`) trained jointly across all ATMs of a type, letting the model borrow signal across series.

Both `LGBMRegressor` and `HistGradientBoostingRegressor` were tuned via Bayesian search over lag windows, tree depth, learning rate, and leaf-size parameters, then backtested on a chronological validation split with cyclical seasonal encoding as exogenous features.

**Results (MAE improvement, multi-series vs. uni-series):**

| ATM Type | Mean improvement | Best case | Worst case | ATMs improved |
|---|---|---|---|---|
| Type A | +9.64% | +35.74% | −15.84% | 17 / 20 |
| Type B | +7.75% | +46.53% | −22.13% | — |

The multi-series approach won on average and by a wide margin in the best cases, but wasn't universally better — a handful of ATMs saw their forecast degrade when pooled into the shared model, most likely those with demand patterns that diverge meaningfully from the group. That trade-off is reported transparently rather than cherry-picking the aggregate number, which is the more honest way to present a model comparison.

### Optimization (team effort, built on forecasting output)
The forecasts feed into a Google OR-Tools `CP-SAT` model that schedules replenishment per ATM under real constraints: fixed capacity by ATM type, replenishment restricted to specific days of the week per type, at most one replenishment per day, and a minimum balance threshold — with the objective of minimizing total replenishment frequency and cost across the forecast horizon.

---

## Team

**Academic Coordinator**
- Patricia Reyes Silva

**Contributors**
- Renzo Andree Espíritu Cueva
- John Sovero Cubillas
- Luis Felipe Poma Astete
- Eduardo Elías Puglisevich Vergara
- Samuel Esteban Cano Chocce