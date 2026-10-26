# SPY Return Prediction: A Quantitative Research Learning Project

My first financial time-series research project, developed to learn and demonstrate an end-to-end quantitative research workflow: checking market data, constructing features, comparing models, and evaluating predictions on held-out data.

The study asks whether SPY's recent returns and volatility help predict its next-trading-day return. It focuses on research methodology, reproducibility, and interpreting mixed results. Trading profitability has not been established.

## Learning Objectives

- Check missing values, duplicate dates, date ordering, and price consistency.
- Construct historical features and align them with a future target.
- Preserve time order and handle target dates at split boundaries.
- Fit preprocessing steps using training data only.
- Compare simple baselines with linear and tree models.
- Distinguish aggregate performance from year-to-year stability.
- Reproduce an analysis from a saved data snapshot.

## Research Question

Can historical return and volatility features improve SPY next-trading-day return predictions relative to zero-return and historical-mean baselines, measured by mean squared error (MSE)?

## Data

| Item | Description |
| --- | --- |
| Instrument | SPY, an exchange-traded fund |
| Source | Yahoo Finance data downloaded through the third-party yfinance package |
| Period | January 4, 2010 to December 31, 2025 |
| Observations | 4,024 trading-day price records |
| Saved snapshot | `data/spy_raw.csv` |
| Returns | Simple returns calculated from vendor-adjusted closing prices |

The original download timestamp was not recorded. Normal runs read the saved CSV rather than downloading updated data.

## Features and Target

Each prediction is made after the prediction date's close, using six features available at that point:

| Feature | Definition |
| --- | --- |
| `return_1d` | Current trading day's return |
| `return_lag1` | Previous trading day's return |
| `return_lag2` | Return two trading days earlier |
| `return_5d` | Cumulative return over the last five trading days |
| `return_20d` | Cumulative return over the last twenty trading days |
| `volatility_20d` | Sample standard deviation of the last twenty daily returns |

The target, `target_next_1d`, is the return from the prediction date's close to the next trading day's close. It is used as the answer, never as an input feature.

## Experimental Design

| Stage | Prediction dates | Usable rows | Purpose |
| --- | --- | ---: | --- |
| Training | 2010–2018 | 2,243 | Fit initial models |
| Validation | 2019–2022 | 1,007 | Compare models and candidate settings |
| Final test | 2023–2025 | 751 | Evaluate the selected method on held-out data |

Rows with incomplete features or targets were removed. Training and validation rows whose targets crossed into the next period were excluded. Historical feature windows can use earlier observations across period boundaries without using future observations.

This implementation uses a fixed chronological validation period. Expanding-window cross-validation was discussed but not implemented.

The models compared were:

- Ordinary least squares (OLS).
- Ridge regression, with six candidate penalty strengths.
- Lasso regression, with four smaller candidate penalty strengths after an initial overly strong range produced constant predictions.
- Random Forest regression, comparing 200, 400, 600, and 800 trees with other settings fixed.
- XGBoost regression, using one fixed configuration.

Search effort differed across model families. These comparisons concern the configurations tried, rather than establishing a general ranking of algorithms. Elastic Net and a separate autoregressive model were discussed but not implemented.

Lasso with `alpha=0.00001` achieved the lowest aggregate validation MSE and was selected before final testing. Its scaler and coefficients were then refitted on 3,250 combined training and validation rows, keeping previously excluded boundary rows excluded. Model settings remained fixed after test results were inspected.

Final testing compared the selected Lasso with two baselines: predicting zero return and predicting the combined development sample's mean target return.

## Held-Out Results

MSE uses decimal returns. Lower values indicate smaller squared prediction errors.

| Method | Test MSE |
| --- | ---: |
| Zero-return baseline | 0.000094150676 |
| Historical-mean baseline | 0.000093510796 |
| Selected Lasso | 0.000092472119 |

Lasso reduced aggregate test MSE by approximately **1.11%** relative to the historical-mean baseline and **1.78%** relative to the zero-return baseline. These are reductions in prediction error, not investment returns.

Performance varied across years:

| Prediction year | Lasso MSE reduction vs historical mean |
| --- | ---: |
| 2023 | −2.40% |
| 2024 | −2.19% |
| 2025 | +4.08% |

Positive values indicate improvement. Lasso underperformed both baselines in 2023 and 2024; its aggregate advantage was driven by 2025. Annual results use the same fixed model and are descriptive checks, not separate models selected for each year.

**Research conclusion:** the selected model improved aggregate held-out MSE, but did not demonstrate a consistent advantage across years. Greater model complexity did not automatically improve validation performance.

## Limitations

- Statistical significance and uncertainty around the performance difference were not evaluated.
- Transaction costs, executable trading rules, and trading profitability were not tested.
- Features become available after the close; execution at that same closing price cannot be assumed.
- Vendor-adjusted historical prices may be revised and are not a verified point-in-time dataset.
- Basic quality checks do not establish complete data accuracy. Missing trading days were not checked against an exchange calendar.
- Repeated comparisons can overfit the validation period. The study covers one instrument and one final test period.
- The narrow margin between the best validation models does not establish reliable algorithm superiority.

## Repository Structure

```text
.
├── README.md
├── requirements.txt
├── 01_data_checks.ipynb
├── data/
│   └── spy_raw.csv
└── results/
    ├── lasso_test_predictions.csv
    ├── lasso_test_mse.csv
    └── lasso_yearly_test_mse.csv
```

The notebook contains the analysis and learning notes. The result CSVs contain final test predictions, aggregate MSE, and annual MSE comparisons.

## Reproduce the Analysis

The recorded environment used Python **3.14.7**. Main package versions are pinned in `requirements.txt`.

1. Download or clone this repository, keeping its folder structure intact.
2. From the project directory, install dependencies into your chosen Python environment:

   ```bash
   python -m pip install -r requirements.txt
   ```

3. Open the project folder in VS Code with Python and Jupyter support.
4. Open `01_data_checks.ipynb` and select the environment containing the dependencies.
5. Ensure the notebook's working directory is the project root so `data/spy_raw.csv` resolves correctly.
6. Restart the kernel and run all cells from top to bottom.

Normal execution reads the saved snapshot, trains the models, and overwrites the three result CSVs. The original download procedure is retained as a Markdown reference. Package installation is separate from normal notebook execution.

The analysis passed a local restart-and-run-all check. The corrected notebook's code cells also executed sequentially in a fresh Python process; all three regenerated result files matched the originals within numerical tolerance (`rtol=1e-10`, `atol=1e-12`). This verifies local reproducibility, not compatibility with every system.

## Possible Future Work

Potential extensions include expanding-window validation, uncertainty estimates that account for time dependence, and executable trading studies with costs. These are future directions, not completed components. Further model development would need a fresh evaluation protocol rather than continued tuning against the already inspected test period.
