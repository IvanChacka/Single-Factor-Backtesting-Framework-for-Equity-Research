# Single-Factor Backtesting Framework for Equity Research

This project implements a complete pipeline for evaluating the predictive power of single factors in the Chinese A-share market. It covers raw data cleaning, factor construction, normalization, IC analysis, stratified backtesting, and mean-variance optimization with industry constraints.

---

## Overview

The framework answers: *Does a given factor reliably forecast future stock returns?* It processes over 8.7 million daily observations (2010–2021), handles survivorship bias, adjusts for trading suspensions and ST status, and produces diagnostics including layered returns, cumulative IC curves, and optimized portfolios.

---

## Key Features

### 1. Data Preprocessing

- Reads daily price, suspension, ST, and industry classification data from Parquet files.
- Constructs a master table with open, high, low, close, market cap, and industry labels.
- Filters out stocks with fewer than 60 historical records.
- Removes observations where the stock is ST (special treatment), suspended, or hits the daily price limit before the forecast.

### 2. Factor Definition

- **Factor**: 5-day simple return: `(close_t / close_{t-5}) - 1` (captures short-term momentum/reversal).
- **Forward return**: Next-day open-to-open return: `(open_{t+2} / open_{t+1}) - 1`.

### 3. Factor Normalization

Two methods are applied cross-sectionally (per date) and per industry:

| Method | Description |
|--------|-------------|
| Method 1 | Winsorize using MAD (median absolute deviation, threshold=5), then Z-score |
| Method 2 | Convert to percentiles (average ranking), then Z-score |

Four variants are produced: `norm1_ori`, `norm1_neu`, `norm2_ori`, `norm2_neu`.

### 4. Information Coefficient (IC) Analysis

- Daily IC = correlation between factor values and forward returns.
- Statistics: mean IC, std, information ratio (IR), win rate (IC>0), absolute mean IC, t-stat, p-value.
- **Key finding**: All four variants show significantly negative IC (~ -0.02), confirming a strong short-term reversal effect.
- Industry-neutralization reduces IC volatility and improves IR.

### 5. Stratified Backtesting (Layering)

- Stocks are sorted by factor value and divided into 5 layers (quintiles) using percentile ranks.
- Layer returns are market-cap weighted.
- Two versions: market-wide layering and industry-neutral layering.
- Metrics calculated: annualized return, Sharpe ratio, maximum drawdown.

**Example results for `factor_norm1_ori`:**

| Strategy | Annual Return | Sharpe | Max Drawdown |
|----------|---------------|--------|---------------|
| Market-wide long-short | 18.4% | 0.77 | -46.3% |
| Industry-neutral long-short | 13.6% | 0.82 | -36.2% |

### 6. Mean-Variance Optimization with Industry Constraints

Optimizes a portfolio from the top 100 stocks (lowest factor value, i.e., strongest reversal signal).

**Objective:**
max (wᵀr - λ * wᵀΣw)


**Constraints:**
- Sum of weights = 1
- Individual stock weights ∈ [0, 0.05] (5% cap)
- Industry weights constrained to a tolerance band around market cap targets

**Parameters:**
- λ (risk aversion) = 0.5
- Expected returns r and covariance Σ from 252-day historical window

**Results:**

| Factor | Optimized vs Equal-Weight | Max Industry Deviation |
|--------|---------------------------|------------------------|
| factor_norm1_ori (raw) | +0.28% | 10.4% |
| factor_norm1_neu (neutralized) | -0.43% | 6.97% |

The raw factor's predictive power comes largely from industry tilts, not pure stock selection.

---

## Outputs

| Output Type | Description |
|-------------|-------------|
| Plots | Cumulative layer returns, long-short curves, cumulative IC curves |
| CSV files | Optimized portfolio holdings, industry weight comparisons, factor comparison tables |
| Summary tables | IC statistics, backtest metrics for each layer and long-short |

---

## How to Use

1. Place raw data files in the specified directory（basic_path.etc.）.
2. Run the notebook cells sequentially.
3. Outputs are saved in the same directory, organized into factor-specific subfolders.

---

## Dependencies

- Python 3.8+
- pandas, numpy, scipy, matplotlib, pyarrow

---

## Conclusion

This framework provides a rigorous toolkit for single-factor evaluation. It confirms a short-term reversal effect in the Chinese market and quantifies the value of industry neutralization. The optimization module demonstrates how to combine factor signals with portfolio construction constraints.
