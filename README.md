# Statistical Arbitrage: Rolling Cointegration and Pairs Trading

## Research question

**Can economically related, liquid U.S. equities exhibit sufficiently stable cointegration for a mean-reversion pairs-trading strategy to remain profitable after transaction costs?**

This project develops and tests a rules-based pairs-trading strategy, compares a cointegration-filtered baseline against an unfiltered alternative, and evaluates performance on a separate out-of-sample period. The goal is to assess the evidence, **not** to optimize a strategy until it appears profitable.

## Universe and data

Five pairs were selected for their economic relationships **before** statistical testing:

| Pair | Economic rationale |
|---|---|
| XOM / CVX | Integrated oil and gas |
| KO / PEP | Competing beverage businesses |
| V / MA | Global payment networks |
| HD / LOW | Home-improvement retailers |
| UPS / FDX | Parcel delivery and logistics |

- **Source:** Yahoo Finance historical equity prices, retained in `prices_snapshot.csv` for reproducibility.
- **Snapshot:** through September 25, 2026. The source table contains missing values where some stocks lack historical observations; the common sample is constructed in the notebook.
- **Formation / in-sample:** historical data through December 2021; in-sample portfolio performance is evaluated over April 2009–December 2021.
- **Out-of-sample:** January 2022–August 2026. September 2026 is excluded from trading evaluation.

## Methodology

### 1. Rolling cointegration estimation

For each pair, the notebook estimates an OLS hedge relationship in **log prices** over a **252-trading-day formation window**:

$$
\log P_{A,t} = \alpha + \beta\log P_{B,t} + u_t.
$$

The residual spread is standardized to a z-score using the formation-period parameters. Models are re-estimated monthly, and the resulting parameters are held fixed during the subsequent trading month. The baseline permits trading only when the rolling **Engle–Granger cointegration test has p < 0.05**.

### 2. Trading rules

- **Entry:** trade when `|z| >= 2`; short A / long B for a positive spread deviation, and reverse the positions for a negative deviation.
- **Execution:** ordinary signals observed at the close are executed at the **next trading close**.
- **Exit:** close on a zero crossing or an adverse deviation reaching `|z| >= 3`.
- **Monthly reset:** close any remaining position at month-end when the current model expires. This forced month-end close is a separate execution assumption from the ordinary next-close signal rule.
- **Re-entry:** following an exit, the z-score must first return inside `|z| < 2` before a new entry can be triggered.
- **Sizing:** the estimated hedge ratio determines relative positions; shares are fixed for the duration of each trade.

Trading P&L is computed from changes in the **underlying share prices and fixed signed quantities**, rather than treating changes in the log spread as directly tradable returns.

### 3. Ablation and transaction costs

Two otherwise identical strategies are compared:

- **Baseline:** rolling Engle–Granger filter (`p < 0.05`).
- **Ablation:** no cointegration eligibility filter.

Each trade starts with **$1,000 gross exposure**. Portfolio analysis assumes **$10,000 initial equity**, allows concurrent positions across pairs, and does not compound trade sizes. Transaction costs are charged on **entry and exit traded notional**, under **5, 10, and 20 basis-point** scenarios.

Performance is assessed using trade statistics, equity curves, CAGR, volatility, Sharpe ratio, and maximum drawdown.

## Results

### In-sample vs. out-of-sample

| Metric | Baseline IS | Ablation IS | Baseline OOS | Ablation OOS |
|---|---:|---:|---:|---:|
| Number of trades | 31 | 321 | 4 | 107 |
| Gross Sharpe ratio | 0.141 | 0.425 | -0.064 | -0.194 |
| Net Sharpe ratio (10 bps) | -0.045 | -0.069 | -0.279 | -0.610 |

The cointegration filter sharply reduced trading activity but did not improve in-sample risk-adjusted returns. Both variants had negative Sharpe ratios at **10 bps** in-sample. Neither strategy delivered positive out-of-sample risk-adjusted performance, even **before** costs. The baseline's four OOS trades make its performance estimate especially uncertain.

### Mean-reversion diagnostics

The notebook also fits an **AR(1)** model to formation-window spreads and maps its persistence coefficient to an Ornstein–Uhlenbeck (OU) mean-reversion speed. For `0 < phi < 1`, estimated half-life is:

$$
HL = \frac{\ln(0.5)}{\ln(\phi)}.
$$

The median estimated half-life across formation windows is approximately **18.7 trading days**. Median half-life increased in the OOS period for **four of five pairs**, suggesting that slower convergence may be poorly matched to forced month-end exits. This is **diagnostic evidence**, not proof of causality or a guarantee that spreads will converge. OU quantities are derived from the AR(1) estimates, not independently fitted.

## Interpretation and limitations

**Main finding:** Under the specified rules, rolling cointegration was not sufficient to produce a robust, profitable out-of-sample pairs-trading strategy. Transaction costs materially worsened results. Removing the cointegration filter increased the number of trades but did not rescue OOS performance.

Important limitations include:

- Only five economically selected pairs are tested; findings need not generalize to other universes.
- The baseline has very few OOS trades, limiting statistical precision.
- Trading assumes execution at daily closing prices and models transaction costs through fixed bps scenarios rather than a full execution simulator.
- Financing costs, short-borrow fees, and margin constraints are not included.
- Monthly forced exits can truncate trades before the estimated spread half-life has elapsed.
- Historical statistical relationships may be unstable, and the backtest is not evidence of future profitability.

## Reproducing the analysis

Place the notebook and `prices_snapshot.csv` in the same directory. The notebook reads the saved price snapshot rather than fetching a changing dataset from Yahoo Finance.

Install dependencies:

```bash
pip install -r requirements.txt
```

Then open `stat_arb_pairstrading.ipynb` in Jupyter and run all cells in order. `yfinance` is used for the original data acquisition and may not be needed when running solely from the saved CSV.

## Research background and references

The project uses the following methodological foundations. It is **not** a direct replication of either paper; trading rules, pair selection, implementation details, and empirical results are specific to this notebook.

1. **Engle, R. F., & Granger, C. W. J. (1987).** *Co-integration and Error Correction: Representation, Estimation, and Testing.* **Econometrica, 55**(2), 251–276. [https://doi.org/10.2307/1913236](https://doi.org/10.2307/1913236)  
   **Relevance:** foundational theory and testing framework for cointegration; this project uses the Engle–Granger test to evaluate pair eligibility.

2. **Brunetti, M., & De Luca, R. (2023).** *Pre-selection in cointegration-based pairs trading.* **Statistical Methods & Applications, 32**, 1611–1640. [https://doi.org/10.1007/s10260-023-00702-4](https://doi.org/10.1007/s10260-023-00702-4)  
   **Relevance:** motivation for a one-year formation period and monthly rolling re-estimation. Their broader research examines pair pre-selection; the present notebook does **not** reproduce their pre-selection methods or entire trading design.

### Data source

- **Yahoo Finance:** historical equity prices originally retrieved using the `yfinance` Python package. A fixed copy is included as [`prices_snapshot.csv`](prices_snapshot.csv) so that later data updates do not alter the study's sample.


---

*Educational quantitative research project. Not investment advice.*

## Interactive Visualizations

The interactive Plotly charts are available in the [`plotly_figures/`](plotly_figures/) directory as standalone HTML files.

Download the HTML files and open them in a browser to explore the charts interactively. GitHub's notebook preview does not render interactive Plotly outputs.
