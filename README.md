# Satellite-Based Soybean Yield Forecasting and Commodity Signal

This project builds an alternative-data pipeline for forecasting U.S. soybean yield using satellite vegetation indicators, weather variables, soil-moisture features, and USDA county-level crop outcomes. The goal is to evaluate whether growing-season satellite and environmental data can improve soybean yield forecasts and generate interpretable commodity supply signals.

The project combines county-level crop data, Google Earth Engine feature exports, WASDE benchmarks, and soybean futures price data to test three questions:

1. Can satellite and environmental features predict soybean yield out of sample?
2. Does predictive accuracy improve from July-end to August-end and September-end as more growing-season information becomes available?
3. Can satellite-implied yield deviations from WASDE forecasts provide useful directional information for soybean futures?

---

## Project Overview

The analysis uses a county-year panel across 15 major U.S. soybean-producing states from 2010 to 2025:

`AR, IA, IL, IN, KS, KY, MI, MN, MO, MS, ND, NE, OH, SD, WI`

These states represent the majority of U.S. soybean production. Features are constructed for the main growing season from May to September, covering planting, vegetative growth, flowering, pod development, and seed filling.

Main feature groups include:

* Satellite vegetation indices: NDVI, EVI
* Weather variables: rainfall and heat-stress days
* Soil-moisture features
* USDA county-level yield and harvested-acre outcomes
* WASDE soybean yield and production benchmarks
* Soybean futures proxy data

---

## Methodology

### 1. Data Preparation

The first notebook processes Google Earth Engine feature exports and USDA county outcome data into a model-ready county-year panel.

Satellite and environmental features are extracted in state-year batches to avoid oversized Earth Engine export tasks. County-level monthly features are then merged with USDA soybean yield and harvested-acre data.

### 2. Rolling Out-of-Sample Yield Forecasting

The second notebook trains yield prediction models using an expanding-window rolling out-of-sample design.

For each test year, the model is trained only on prior years:

* Train 2010–2017, predict 2018
* Train 2010–2018, predict 2019
* Train 2010–2019, predict 2020
* Continue through 2025

This design avoids look-ahead bias and better reflects a real forecasting setting than random train-test splits.

Models tested:

* Linear Regression
* Random Forest
* XGBoost
* Historical county mean benchmark

The project compares three information windows:

* July-end features
* August-end features
* September-end features

### 3. Aggregate Satellite Yield Signal

County-level predictions are aggregated into a top-15-state satellite-implied soybean yield signal using lagged county harvested acres as weights. A rolling bias adjustment is applied using only prior out-of-sample prediction errors.

### 4. WASDE Benchmark and Futures Signal

The final notebook compares satellite-implied yield signals against WASDE benchmarks.

Two analyses are performed:

1. **July/August bracket test**
   Tests whether the November WASDE soybean yield forecast falls inside the July/August satellite-implied yield bracket.

2. **August WASDE trading signal**
   Compares August-end satellite-implied yield with the August WASDE yield forecast.
   If satellite yield is meaningfully below WASDE, the signal is bullish for soybean futures.
   If satellite yield is meaningfully above WASDE, the signal is bearish for soybean futures.

A 0.5 bushel/acre threshold is used to define long, short, or no-trade signals.

---

## Key Results

### Yield Forecasting Performance

The rolling out-of-sample results show that forecast accuracy improves as more growing-season information becomes available.

Based on the current model output:

* July-end best model: Random Forest
* August-end best model: XGBoost
* August-end features reduced RMSE by approximately 13.2% versus July-end
* Rolling OOS R² improved from 54.6% at July-end to 65.8% at August-end

This suggests that satellite and environmental features become more informative as the soybean growing season progresses.

### WASDE Bracket Test

The July/August satellite-implied yield bracket contains the November WASDE yield in 4 of 7 out-of-sample years. One additional year is a near miss, with a distance of approximately 0.20 bushels/acre.

This indicates that the satellite signal contains useful information, but it should not be interpreted as a standalone final-yield estimator.

### Futures Signal Test

Using a 0.5 bushel/acre signal threshold, the August satellite-WASDE yield gap generates five trades over 2019–2025.

The futures response is horizon-dependent:

* 5-day horizon: 80% hit rate
* 20-day horizon: 80% hit rate
* 3-day, 10-day, and October-end horizons are more mixed

The signal appears more informative over selected 5- to 20-trading-day windows than over very short or very long holding periods.

---

## Repository Structure

```text
.
├── notebooks/
│   ├── 01_data_preparation.ipynb
│   ├── 02_yield_prediction_models.ipynb
│   └── 03_wasde_and_futures_signal_WIP.ipynb
│
├── outputs/
│   └── tables/
│       ├── model_performance_by_feature_window.csv
│       ├── best_model_by_feature_window.csv
│       ├── signal_pivot_july_aug_sept.csv
│       ├── july_aug_satellite_vs_november_wasde_bracket_test.csv
│       ├── july_aug_satellite_vs_november_wasde_bracket_summary.csv
│       ├── august_satellite_vs_wasde_signal_table.csv
│       └── august_satellite_wasde_strategy_horizon_summary_close.csv
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Main Output Tables

### `model_performance_by_feature_window.csv`

Compares model performance across July-end, August-end, and September-end feature windows.

### `best_model_by_feature_window.csv`

Keeps the best-performing model for each feature window based on RMSE.

### `signal_pivot_july_aug_sept.csv`

Contains July, August, and September satellite-implied yield signals by year.

### `july_aug_satellite_vs_november_wasde_bracket_test.csv`

Tests whether November WASDE yield falls inside the July/August satellite-implied yield bracket.

### `august_satellite_vs_wasde_signal_table.csv`

Constructs long, short, or no-trade futures signals using the August satellite-WASDE yield gap.

### `august_satellite_wasde_strategy_horizon_summary_close.csv`

Summarizes futures signal performance across 3-day, 5-day, 10-day, 20-day, and October-end holding horizons.

---

## Notes

This project is intended as an exploratory alternative-data commodity research project. The futures signal analysis is not a production trading strategy and does not include transaction costs, slippage, margin requirements, or contract-roll implementation details.

The main contribution is the construction of a reproducible satellite-based soybean yield forecasting pipeline and the evaluation of its relationship with WASDE benchmark revisions and soybean futures price responses.

---

## Technologies Used

* Python
* pandas
* NumPy
* scikit-learn
* XGBoost
* Random Forest
* Google Earth Engine
* yfinance
* Google Colab
* USDA crop data
* WASDE benchmark data

