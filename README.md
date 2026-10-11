# EWA–EWC Pairs Trading: Revisiting Chan (2013)

A small quantitative-finance research project examining whether a mean-reversion strategy inspired by Ernest P. Chan's *Algorithmic Trading: Winning Strategies and Their Rationale* (2013) remains effective on data after the book's publication.

**Research question:** Can an EWA–EWC mean-reversion signal, calibrated on 2006–2012 data, generate positive returns over 2013–2026 without re-estimating its core parameters?

> **Scope:** This is an **adaptation**, not an exact replication of a published two-ETF backtest. Chan's Example 2.6 studies cointegration between EWA and EWC; his Example 2.8 backtests a linear z-score strategy on **EWA, EWC, and IGE**; and Example 3.3 applies a **Kalman filter** to EWA/EWC. This project combines the two-ETF pair with the simpler linear trading rule as a transparent baseline.

## Methodology

**Data.** Daily Yahoo Finance prices for EWA (Australia ETF) and EWC (Canada ETF), obtained through `yfinance` with `auto_adjust=True`. All prices are denominated in USD. The initial parameter-estimation period is **2006–2012**, with **2013 onward** used for historical out-of-sample evaluation. Adjusted price data may be revised by the provider.

### 1. Hedge ratio and spread

Estimate an OLS regression using training data only:

$$
P_t^{\mathrm{EWC}}=\alpha+\beta P_t^{\mathrm{EWA}}+\varepsilon_t.
$$

Construct the tradable two-ETF spread:

$$
S_t=P_t^{\mathrm{EWC}}-\hat\beta P_t^{\mathrm{EWA}}.
$$

The estimated training-period hedge ratio is **$\hat\beta=1.3937$**.

### 2. Integration and cointegration tests

Apply augmented Dickey–Fuller tests to each price series and its first differences, followed by an Engle–Granger cointegration test (`statsmodels.tsa.stattools.coint`) with an intercept and one fixed lag.

- Both ETF price series are consistent with being integrated of order one, $I(1)$: unit-root tests on levels fail to reject, whereas tests on first differences reject.
- The 2006–2012 Engle–Granger test yields **test statistic −3.2192** and **p-value 0.0668**: evidence at the 10% level, **not significant at 5%**.

Failing to reject a unit root does not establish that it exists, and a cointegration test does not by itself guarantee a profitable trading strategy.

### 3. Mean-reversion half-life

Estimate on the training spread:

$$
\Delta S_t=\mu+\lambda S_{t-1}+u_t.
$$

Using Chan's approximation,

$$
h\approx-\frac{\ln 2}{\hat\lambda}.
$$

The fitted coefficient is **$\hat\lambda=-0.017273$**, implying a **40.13-trading-day half-life** and a **40-day lookback**. The model-implied equilibrium is approximately **3.76**.

### 4. Trading signal and positions

Calculate rolling mean $\mathrm{MA}_{40,t}$ and rolling standard deviation $\mathrm{SD}_{40,t}$, and set:

$$
z_t=\frac{S_t-\mathrm{MA}_{40,t}}{\mathrm{SD}_{40,t}},
\qquad q_t=-z_t.
$$

The desired share quantities are:

$$
n_t^{\mathrm{EWC}}=q_t,
\qquad n_t^{\mathrm{EWA}}=-\hat\beta q_t.
$$

The historical hedge ratio and lookback remain fixed after 2012; the rolling mean and volatility update through time.

### 5. Backtest and performance

Calculate daily profit and loss using the **previous day's holdings**:

$$
\mathrm{PnL}_t=\sum_{i\in\{\mathrm{EWA},\mathrm{EWC}\}}n_{i,t-1}(P_{i,t}-P_{i,t-1}).
$$

Following Chan's simplified return definition, divide by prior gross dollar exposure:

$$
r_t=\frac{\mathrm{PnL}_t}{\sum_i\left|n_{i,t-1}P_{i,t-1}\right|}.
$$

These are **returns on gross exposure, not returns on an actual brokerage account's equity**. The performance index is the cumulative product $\prod_t(1+r_t)$.

## Preliminary results

The following figures are from the initial notebook run, **before trading costs, financing costs, or realistic execution modelling**:

| Metric | 2006–2012 (in-sample) | 2013–2026 (post-2012 evaluation) |
|---|---:|---:|
| Annualized arithmetic return (APR) | 5.77% | 3.71% |
| Compounded annual growth of return index | 5.44% | 3.58% |
| Annualized volatility | 9.78% | 6.26% |
| Sharpe ratio (zero risk-free rate assumed) | 0.590 | 0.593 |
| Maximum drawdown of return index | −20.81% | −17.25% |

The post-2012 compounded **gross-exposure return index** ends near **1.62×** its starting value. This **does not** imply that a real $100 brokerage account would have grown to $162: the backtest has not modelled account equity, margin, or dynamic capital allocation.

**Interpretation:** The initial two-ETF baseline produces a positive pre-cost return series, but its moderate risk-adjusted performance and the instability of the fixed-beta spread raise questions about practical profitability.

## Limitations and next steps

- **Execution:** Daily close-based signals with lagged positions approximate the timing of trading. Executing at known close prices is not guaranteed.
- **Costs:** Transaction costs, bid–ask spreads, ETF short-borrow fees, financing, and dividend-related short obligations are not modelled.
- **Capital:** Share quantities are signal-based rather than scaled to account equity; the reported compounded index is not an investable wealth process.
- **Data:** Adjusted-price returns simplify the treatment of distributions and may not match cash flows from actual long/short stock positions.
- **Inference:** The training-period cointegration result is not significant at 5%. Data from the evaluation period have also been inspected during exploratory analysis, so it is not a completely untouched holdout.
- **Extensions:** A self-financing capital-based backtest, cost and turnover analysis, a consistent benchmark, and comparison with Chan's actual EWA/EWC Kalman-filter specification.

## Reproduce the analysis

1. Create a Python virtual environment and select it as the interpreter / Jupyter kernel.
2. Install dependencies:

   ```bash
   python -m pip install -r requirements.txt
   ```

3. Open [`analysis.ipynb`](analysis.ipynb) and execute cells in order.

Note that Yahoo Finance may revise or add historical data; numbers can vary across download dates. Review the notebook's downloaded sample and the selected end date when reproducing results.

## Reference

Chan, E. P. (2013). *Algorithmic Trading: Winning Strategies and Their Rationale*. Wiley. See Chapter 2 (Examples 2.4, 2.6–2.8) and Chapter 3 (Example 3.3).

## Code provenance

This is a learning-oriented research project. Some implementation code and explanations were drafted with AI assistance; statistical assumptions, output validation, and interpretations require independent review.

*For research and educational purposes only; not investment advice.*
