# EWA/EWC pairs trading — revisiting Chan (2013)

I started this project after reading Ernest P. Chan's *Algorithmic Trading* (2013). Chan uses the Australia (EWA) and Canada (EWC) ETFs to illustrate cointegration. I wanted to test a simple question: **does a mean-reversion strategy calibrated on data available before 2013 still work on later data?**

I use daily ETF prices from Yahoo Finance, estimate the relationship on **2006–2012**, and evaluate a fixed-parameter trading signal over **2013–2026**. This is an **adaptation of Chan's methods**, not an exact reproduction of one of his published backtests: his linear portfolio example uses EWA/EWC/IGE, while his two-ETF EWA/EWC trading example uses a Kalman filter.

The calculations, plots and code are in [`analysis.ipynb`](analysis.ipynb).

## Method

1. **Estimate a hedge ratio.** Regress EWC prices on EWA prices using OLS on 2006–2012 data:

   $$P_t^{\mathrm{EWC}} = \alpha + \beta P_t^{\mathrm{EWA}} + \varepsilon_t.$$

   Construct the spread $S_t=P_t^{\mathrm{EWC}}-\hat\beta P_t^{\mathrm{EWA}}$, with $\hat\beta=1.3937$.

2. **Test for cointegration.** Use ADF tests on the individual price levels and first differences, followed by the Engle–Granger test (`statsmodels.coint`) on the pair. The training-period Engle–Granger p-value is **0.0668**: significant at 10%, but **not at 5%**. This is weaker evidence than Chan reports for his historical sample.

3. **Estimate mean reversion.** Fit $\Delta S_t=\mu+\lambda S_{t-1}+u_t$ on the training spread. With $\hat\lambda=-0.017273$, Chan's approximation $h=-\ln(2)/\hat\lambda$ gives a half-life of about **40 trading days**.

4. **Generate a trading signal.** Calculate the spread's rolling 40-day z-score and take $q_t=-z_t$ spread units. Each unit is long one share of EWC and short $\hat\beta$ shares of EWA. The hedge ratio and lookback stay fixed after 2012; the rolling mean and standard deviation use only observations up to the signal date.

5. **Backtest.** Compute daily P&L using the *previous day's* share positions:

   $$\mathrm{PnL}_t=\sum_{i\in\{\mathrm{EWA},\mathrm{EWC}\}}n_{i,t-1}(P_{i,t}-P_{i,t-1}).$$

   Following Chan's simple convention, daily returns are P&L divided by the previous day's **gross dollar exposure**.

## Results (pre-cost)

| Metric | 2006–2012 (training) | 2013–2026 (evaluation) |
|---|---:|---:|
| Annualized mean daily return | 5.77% | 3.71% |
| Annualized growth of compounded return index | 5.44% | 3.58% |
| Annualized daily-return volatility | 9.78% | 6.26% |
| Sharpe (zero risk-free rate assumed) | 0.59 | 0.59 |
| Max drawdown of return index | −20.81% | −17.25% |

The return index rises to approximately **1.62** over 2013–2026. My main takeaway is that the simplified signal still produces **positive pre-cost returns**, even though the spread based on the 2006–2012 hedge ratio drifts considerably in later years. That is not enough to conclude that the strategy is investable.

### What the results do *not* mean

The index is **not an actual brokerage-account equity curve**. Returns are divided by the gross value of the positions, while the position size itself is determined by the z-score rather than by available capital. In this return definition, the absolute size of a nonzero z-score position cancels out: the series mainly measures the performance of the *direction* of the signal, not the economic effect of linearly scaling position size. Therefore, an index of 1.62 does **not** mean that an actual $100 account would have grown to $162.

The backtest also omits trading costs, short borrow and financing costs, margin constraints and slippage. It uses adjusted closing prices and a simplified close-to-close execution convention. I have inspected the later data during exploratory analysis, so the evaluation period is not a completely untouched holdout.

## Run the notebook

```bash
python -m pip install -r requirements.txt
```

Open [`analysis.ipynb`](analysis.ipynb), select the project environment as the Jupyter kernel, and run the cells from top to bottom. The notebook downloads data with `yfinance` (`auto_adjust=True`); data revisions and download dates may affect the results.

## Next steps

The first priority is a capital-based portfolio backtest, including position limits and transaction costs. I would also like to compare the fixed hedge ratio with the Kalman-filter approach from Chan's actual EWA/EWC example.

## Reference and code provenance

Chan, Ernest P. (2013). *Algorithmic Trading: Winning Strategies and Their Rationale*. Wiley, Chapters 2–3.

This is an independent learning project based on Chan's research ideas. I used AI assistance to draft parts of the Python implementation and documentation; the statistical interpretation, assumptions, and limitations are documented here so the work can be examined and reproduced.
