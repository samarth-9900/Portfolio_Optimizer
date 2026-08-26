# Minimum Variance Portfolio Optimizer

This project builds and evaluates a long-only minimum-variance portfolio from historical market data. The workflow is implemented in [`Portfolio_Optimization.ipynb`](Portfolio_Optimization.ipynb).

The notebook downloads adjusted prices from Yahoo Finance, estimates the covariance and correlation structure of asset returns, uses PCA to inspect that structure, and optimizes portfolio weights subject to full-investment and long-only constraints.

> **Educational project:** This notebook is for research and learning purposes. It is not financial advice, and historical or simulated performance does not guarantee future results.

## Purpose

The goal is to demonstrate a complete, reproducible portfolio-optimization workflow rather than to recommend a particular investment portfolio. It connects the statistical properties of asset returns to a practical constrained optimization problem:

- **Inputs:** historical adjusted prices for a selected set of assets.
- **Estimated risk model:** the sample covariance matrix of daily returns.
- **Decision variables:** one portfolio weight for each asset.
- **Objective:** minimize estimated portfolio variance.
- **Constraints:** invest all capital and prohibit short selling.
- **Evaluation:** compare the optimized portfolio with an equal-weight benchmark using data that was not used to estimate the weights.

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

### Default Portfolio

When no tickers are entered, the notebook uses:

```text
AAPL, MSFT, GOOGL, AMZN, NVDA, JPM, META, TSLA,
AVGO, BRK-B, LLY, WMT, XOM, V, JNJ
```

These symbols represent a broad selection of widely followed large-cap companies across several sectors. Users can replace them with another set of Yahoo Finance ticker symbols. At least two symbols are required.

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

The theoretical unconstrained fully invested solution is:

$$
w^*=\frac{\Sigma^{-1}\mathbf{1}}{\mathbf{1}^T\Sigma^{-1}\mathbf{1}}
$$

However, this solution can contain negative weights. The notebook instead uses `scipy.optimize.minimize` with the SLSQP method, bounds of `0` to `1` for every asset, and an equality constraint requiring the weights to sum to `1`. The resulting optimized weights are therefore long-only and fully invested.

## Methodology Details

### Price Cleaning and Sample Split

The notebook downloads all requested prices in one call with `auto_adjust=True`. Adjusted prices account for stock splits and distributions in the downloaded price series. The data is split immediately after download:

- Training prices are observations before `2026-01-01`.
- Test prices are observations from `2026-01-01` onward.

Columns with no usable training history are removed. Remaining training prices are aligned to complete dates with `dropna()`. Asset eligibility and the complete-case training observations are therefore determined without using the test period.

To calculate the first test-period return, the last known training price is placed immediately before the test prices. This bridge price is used only to calculate the first out-of-sample return and is not used to estimate the covariance matrix.

### Returns and Risk Matrices

Daily simple returns are calculated as:

$$
r_{t,i}=\frac{P_{t,i}}{P_{t-1,i}}-1
$$

The covariance matrix retains the original volatility scale and is the matrix used by the optimizer. The correlation matrix rescales each asset to unit volatility and is used to study relationships between assets. The heatmap colors represent correlation magnitude $|r|$ from `0` to `1`, while each cell displays the signed value, for example `+0.72` or `-0.18`.

### PCA Diagnostics

The notebook performs PCA-style eigenvalue decompositions on both matrices:

- **Covariance PCA** shows variation on the original return scale, allowing high-volatility assets to have greater influence.
- **Correlation PCA** uses standardized returns, giving each asset equal variance before analyzing co-movement.

Eigenvalues indicate how much variation each component captures. Eigenvectors show the direction of each component across the assets. PCA is included as a diagnostic and exploratory tool; its components are not used directly as the final portfolio weights. The sign of an eigenvector is arbitrary, so multiplying an eigenvector by `-1` does not change the component it represents.

### Convexity Check

The covariance matrix should be positive semidefinite. The notebook checks its smallest eigenvalue before optimization. A small negative value can result from floating-point precision, while a materially negative eigenvalue indicates that the data or matrix construction should be investigated.

## Outputs

Running the notebook produces:

- The selected ticker symbols and the number of usable training and test observations.
- Descriptive statistics for each asset's optimization-period returns.
- A signed correlation matrix heatmap with `coolwarm` colors based on correlation magnitude.
- Covariance and correlation PCA eigenvalues, explained-variance ratios, and eigenvectors.
- The smallest and largest covariance eigenvalues.
- A table comparing optimized and equal portfolio weights.
- A performance summary containing total return and the ending value of an initial investment of `1.00`.
- A cumulative-return chart for the optimized and equal-weight portfolios.
- A bar chart of the optimized asset weights.

The performance chart reports cumulative return, not annualized return. No fees, slippage, dividends received outside the adjusted-price series, or rebalancing costs are modeled separately.

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

### Running in VS Code

Open the notebook in VS Code with the Jupyter extension installed. Select a Python kernel, run the first import cell, and then execute the remaining cells in order. The ticker input cell is interactive, so the notebook must be run in an environment that can accept `input()`.

### Running from a Terminal

Launch Jupyter from the project directory:

```bash
jupyter notebook
```

Then open `Portfolio_Optimization.ipynb` in the browser interface and run the cells sequentially.

## Data and Reproducibility

Data is retrieved at runtime from Yahoo Finance using `yfinance` with adjusted prices. Results can change as:

- New market data becomes available.
- Yahoo Finance revises or removes historical data.
- Different ticker selections are provided.
- The optimization and test samples contain different observations.

The notebook uses a time-based split so that test-period observations do not influence the estimated covariance matrix or optimized weights. Complete-case date alignment and asset eligibility are determined from the training sample.

The end date is generated at runtime from the current date. Consequently, the test sample grows as new market observations become available. To reproduce a previous result exactly, use the same ticker list, package versions, downloaded data, and execution date.

## Troubleshooting

- **No data returned:** confirm the ticker symbols are valid Yahoo Finance symbols and that the machine has internet access.
- **Fewer than two usable tickers:** check whether the requested assets have historical observations before the optimization cutoff.
- **No test returns:** the data source must provide usable observations from `2026-01-01` onward.
- **Package import errors:** activate the intended Python environment and rerun the installation command in that environment.
- **Notebook input does not appear:** run the interactive ticker-selection cell directly in Jupyter or VS Code rather than executing the notebook in a non-interactive batch process.

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
