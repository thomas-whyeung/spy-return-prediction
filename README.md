# ETF Return Prediction

A reproducible study of whether SPY's past returns and
volatility help predict its next-trading-day return.

## Research Question

Can a model using historical return and volatility features
outperform zero-return and historical-mean baselines on
held-out data?

## Data

- Instrument: SPY
- Source: Yahoo Finance via the third-party yfinance package
- Period: January 4, 2010 to December 31, 2025
- Observations: 4,024 daily price records
- Local snapshot: data/spy_raw.csv

Returns are calculated from vendor-adjusted closing prices.
The original download timestamp was not recorded.

## Features and Target

Features use information available after the prediction
date's close:

- Current daily return
- Daily returns lagged by one and two trading days
- 5-day and 20-day cumulative returns
- 20-day rolling standard deviation of daily returns

The target is the next-trading-day adjusted-price return.

## Method

- Training period: 2010–2018
- Validation period: 2019–2022
- Held-out test period: 2023–2025

Rows with incomplete features or targets were removed.
Training and validation rows whose targets crossed the
respective period boundaries were excluded.

OLS, Ridge, Lasso, Random Forest and XGBoost were compared.
Search effort differed across model families, so these
results do not establish a general ranking of algorithms.

Lasso with alpha=0.00001 was selected using validation MSE.
It was then refitted, with feature standardization, on
3,250 combined development rows before final testing.

The two baselines predict zero return and the development
sample's mean target return.

## Results

| Method | Test MSE |
| --- | ---: |
| Zero return | 0.000094150676 |
| Historical mean | 0.000093510796 |
| Selected Lasso | 0.000092472119 |

Lasso reduced aggregate test MSE by approximately 1.11%
relative to the historical-mean baseline.

However, performance varied across years:

| Prediction year | MSE reduction vs historical mean |
| --- | ---: |
| 2023 | -2.40% |
| 2024 | -2.19% |
| 2025 | +4.08% |

Positive values indicate improvement. The aggregate
advantage was driven by 2025 and was not consistent
across years.

## Limitations

- Statistical significance has not been established.
- Transaction costs and trading profitability were not tested.
- Features become available after the close; execution at
  that same closing price cannot be assumed.
- Historical adjusted prices are not a verified
  point-in-time dataset.
- Basic data checks do not establish complete data accuracy.
- Results concern one instrument and one test period.

## Reproduce the Analysis

1. Keep the saved snapshot at data/spy_raw.csv.
2. Open 01_data_checks.ipynb.
3. Select a Python environment with the required packages.
4. Restart the kernel and run all cells from top to bottom.

The notebook reads the local snapshot rather than
downloading new data. Running it overwrites result CSVs.

A local restart-and-run-all check completed without errors,
and yearly test results matched at the displayed precision.

## Recorded Environment

- Python: 3.14.7
- pandas: 3.1.0
- NumPy: 2.5.4
- scikit-learn: 1.9.1
- XGBoost: 3.4.1

Matplotlib is also required for plotting. yfinance is used
by the retained original download code.

## Output Files

- results/lasso_test_predictions.csv
- results/lasso_test_mse.csv
- results/lasso_yearly_test_mse.csv