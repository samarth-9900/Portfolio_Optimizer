# Minimum Variance Portfolio Optimizer

This project builds and evaluates a long-only minimum-variance portfolio from historical market data. The workflow is implemented in [`Portfolio_Optimization.ipynb`](Portfolio_Optimization.ipynb).

The notebook downloads adjusted prices from Yahoo Finance, estimates the covariance and correlation structure of asset returns, uses PCA to inspect that structure, and optimizes portfolio weights subject to full-investment and long-only constraints.

> **Educational project:** This notebook is for research and learning purposes. It is not financial advice, and historical or simulated performance does not guarantee future results.

## What the Notebook Does

1. Accepts comma-separated ticker symbols or uses a fixed default portfolio of 15 large-cap companies.
2. Downloads adjusted daily prices from Yahoo Finance for the period beginning 2018-01-01.
3. Splits the data into:
	- **Optimization sample:** dates before 2026-01-01.
	- **Held-out test sample:** dates from 2026-01-01 onward.
4. Calculates daily percentage returns and descriptive statistics.
5. Computes the return covariance and correlation matrices.
6. Displays a correlation heatmap. Cell colors show correlation magnitude, while annotations show the signed correlation value.
7. Performs both covariance-based and correlation-based PCA.
8. Checks that the covariance matrix is positive semidefinite.
9. Finds minimum-variance weights using SLSQP with the constraints:

	$$
	\sum_{i=1}^{n} w_i=1, \qquad 0\leq w_i\leq1
	$$

10. Compares the optimized portfolio with an equal-weight portfolio on the held-out test period.

## Portfolio Model

For asset returns $r_t$ and portfolio weights $w$, the daily portfolio return is:

$$
r_{p,t}=w^T r_t
$$

The optimization minimizes portfolio variance using the covariance matrix $\Sigma$:

$$
\min_w\; w^T\Sigma w
$$

The weights are estimated only from observations before 2026-01-01. They remain fixed throughout the test period; the notebook does not perform look-ahead reoptimization or periodic rebalancing.

## Requirements

- Python 3.9 or newer
- Jupyter Notebook or JupyterLab
- Internet access for Yahoo Finance data

Install the required packages with:

```bash
python -m pip install numpy pandas matplotlib yfinance scikit-learn scipy jupyter
```

## Usage

1. Clone or download this repository.
2. Install the dependencies listed above.
3. Open [`Portfolio_Optimization.ipynb`](Portfolio_Optimization.ipynb) in Jupyter, JupyterLab, or VS Code.
4. Run the cells from top to bottom.
5. When prompted, enter ticker symbols separated by commas, such as:

	```text
	AAPL,MSFT,GOOGL,AMZN
	```

	Press Enter without typing anything to use the notebook's default portfolio.

The final cells report portfolio performance from an initial investment of `1.00`, plot cumulative returns, and show the optimized portfolio weights.

## Data and Reproducibility

Data is retrieved at runtime from Yahoo Finance using `yfinance` with adjusted prices. Results can change as:

- New market data becomes available.
- Yahoo Finance revises or removes historical data.
- Different ticker selections are provided.
- The optimization and test samples contain different observations.

The notebook uses a time-based split so that test-period observations do not influence the estimated covariance matrix or optimized weights. Complete-case date alignment and asset eligibility are determined from the training sample.

## Project Structure

```text
.
├── Portfolio_Optimization.ipynb  # Analysis, optimization, and backtest
├── README.md                     # Project documentation
└── LICENSE                       # Project license
```

## Limitations

- The model is long-only and does not include transaction costs, taxes, turnover limits, liquidity constraints, or portfolio rebalancing.
- Covariance estimates are sensitive to the selected historical window and asset universe.
- The holdout period begins on 2026-01-01 and requires current Yahoo Finance data to contain usable observations after that date.
- A minimum-variance portfolio optimizes historical volatility, not expected return or investment suitability.
