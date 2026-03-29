# Methodology

## Forecasting Objective

The target in this project is:

- `target_24h = ERCOT load at time t + 24 hours`

I chose a 24-hour-ahead setup since my earlier one-step-ahead set up was too easy to predict. The earlier iteration relied too heavily on the most recent lag and produced unrealistically strong results (R<sup>2</sup> > .99) without learning much beyond persistence.

## Data Preparation

I merged:

- hourly ERCOT native load data
- hourly Open-Meteo weather data from representative ERCOT zone locations

The ERCOT load data includes zone-level load columns plus total ERCOT load.  
The weather data was mapped from representative cities to ERCOT weather zones.

## Weather Zone Mapping


![Ercot Zone Map](ercot_zone_map.jpg)
<small>Source: https://www.ercot.com/news/mediakit/maps</small>

| location_id | City Proxy        | Zone  |
|------------:|-------------------|-------|
| 0           | Houston           | COAST |
| 1           | Tyler             | EAST  |
| 2           | Midland           | FWEST |
| 3           | Dallas-Fort Worth | NCENT |
| 4           | Corpus Christi    | SOUTH |
| 5           | Austin            | SCENT |
| 6           | Abilene           | WEST  |
| 7           | Wichita Falls     | NORTH |

These are proxy locations rather than full spatial averages, which keeps the pipeline simpler while still capturing broad geographic weather variation.

## Feature Engineering

### Time features
I used calendar and cyclic features such as:

- hour
- day of week
- day of year
- month
- `hour_sin`, `hour_cos`
- `dow_sin`, `dow_cos`
- `month_sin`, `month_cos`
- `is_weekend`

### Load features
I created load-based lag and rolling features including:

- `ERCOT_current`
- `ERCOT_lag_1`
- `ERCOT_lag_24`
- `ERCOT_lag_48`
- `ERCOT_lag_168`
- `ERCOT_roll_mean_24`
- `ERCOT_roll_std_24`
- `ERCOT_roll_mean_168`
- `ERCOT_roll_std_168`

These features help capture short-term persistence, daily repetition, and weekly repetition.

### Weather features
I used raw zone-level weather variables and also aggregated them into system-level summary features such as:

- `temp_mean`
- `apparent_temp_mean`
- `humidity_mean`
- `wind_mean`
- `radiation_mean`
- `precip_mean`
- `cdd_mean`
- `hdd_mean`

The aggregate weather features were especially helpful for keeping the LSTM input more compact.

## Train / Validation / Test Design

I used time-based splits rather than random splits.

- **Train:** 2021–2023
- **Validation:** 2024
- **Test:** 2025
- **Secondary forward check:** 2026 YTD

I used 2024 for validation and hyperparameter selection, then treated 2025 as the main holdout year. I kept 2026 YTD separate because it is incomplete and does not contain a full seasonal cycle.

## Models Compared

### Baseline
The baseline predicts tomorrow’s load using current ERCOT load:

- `predict target_24h using ERCOT_current`

This served as a strong persistence benchmark.

### Random Forest
I tuned a small grid over:

- `n_estimators`
- `max_depth`
- `min_samples_leaf`

### LightGBM
I tuned a small grid over:

- `n_estimators`
- `learning_rate`
- `num_leaves`

### LSTM
I trained a PyTorch LSTM on sequential hourly features, using:

- `ERCOT_current`
- `ERCOT_lag_24`
- `ERCOT_lag_168`
- cyclic time features
- aggregate weather features

I searched over a few compact configurations rather than doing a large brute-force sweep due to computational constraints.

## Experiment Tracking

To keep the notebook organized, I saved:

- validation search results
- final results for each split
- event-window metrics

to CSV files in the `output/` folder.

## Design Choices I Found Important

### 1. Avoiding trivial forecasting setups
One-step-ahead forecasting gave unrealistically easy results because lag-based persistence dominates.

### 2. Treating 2025 as the main benchmark
A full holdout year is more interpretable than an incomplete recent window.

### 3. Keeping 2026 YTD separate
This made it easier to distinguish main holdout performance from short recent forward robustness.

### 4. Comparing classical ML and deep learning honestly
I was under the impression that deep learning would win here, but after this experiment and some more research. I found that in tabular data tree based learning algorithms and persistence are hard to beat. Here is a good paper diving into why tree-based  models outperform deep learning on tabular data: [arXiv:2207.08815](https://arxiv.org/abs/2207.08815).
