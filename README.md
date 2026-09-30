# Pairs Trading: from a copied notebook to a properly tested strategy

Work in progress. This repository documents the step-by-step transformation of a public pairs-trading notebook into a backtest that can be defended statistically. Each version is a separate notebook, and each one fixes or adds something specific.

Origin and attribution
V0 (Pairs_Trading_V0.ipynb) is a copy of KidQuant/Pairs-Trading-With-Python ( https://github.com/KidQuant/Pairs-Trading-With-Python ). It is kept untouched as a baseline. All credit for V0 goes to its author; please refer to that repository for its terms of use.
The strategy itself (cointegration test, z-score of a spread, entry/exit thresholds) is a standard textbook approach. Background material is listed in References.
V1 onwards is my own work: I rewrote the selection and evaluation procedure, and each change is listed below.


## What changed from V0 to V1
 
| Topic | V0 | V1 |
|---|---|---|
| Pair selection | Cointegration test on the whole 2010-2026 period, so the test period leaked into the selection | Test on the first 70% of the data only |
| Test set | Built but never used | Used for trading and for an out-of-sample cointegration check |
| Data | Ends in early 2026 | Ends at the current date |
| Z-score | Standard deviation computed on a different window than the mean | Mean and standard deviation on the same long window |
| Universe | 12 tickers including SPY (66 pairs) | 11 stocks (55 pairs) |

## Method (V1)
 
1. Download daily prices for 11 US large-cap technology stocks with `yfinance` (2010 to today).
2. Chronological split: first 70% of days for selection, last 30% for evaluation.
3. Engle-Granger cointegration test (`statsmodels.tsa.stattools.coint`) on all 55 pairs of the training set. Pairs with p < 0.05 are kept.
4. The kept pairs are tested again on the test period, as a diagnostic only (never to choose pairs).
5. Trading on the test period: z-score of the price ratio from a short (5-day) and a long (60-day) moving average, entry when |z| > 1, exit when |z| < 0.75.


## Results (V1)
 
| Pair | p-value, train | p-value, test |
|---|---|---|
| ADBE / MSFT | 0.015 | 0.578 |
| EBAY / ORCL | 0.011 | 0.941 |
 
- 55 pairs were tested at the 5% level, so about 2.75 false positives are expected by chance alone. Two pairs passed.
- With a Bonferroni correction (threshold 0.05 / 55, about 0.0009), no pair passes.
- Both pairs stop being cointegrated on the test period. This is consistent with the two hits being selection noise rather than a stable relationship.
- Trading P&L: `-200$`


## Known limitations of V1
 
- The hedge ratio is the price ratio, while the cointegration test uses a regression on raw prices (not log prices). The tested object and the traded object differ.
- Positions accumulate every day the signal persists, and positions still open at the end are not valued.
- P&L is a single number in price units: no capital, no return, no Sharpe ratio, no drawdown.
- No transaction costs, no borrowing costs, and execution at the same close as the signal.
- No multiple-testing correction, and a small hand-picked universe of stocks that still exist today (survivorship bias).

## Roadmap
 
| Version | Goal |
|---|---|
| V1.1 | Fix V1 limitations: consistent signal, one position at a time, mark-to-market P&L, capital and returns, synthetic-data tests |
| V2 | Selection without leakage: three-way split, multiple-testing correction, larger universe, both test directions, Johansen |
| V3 | Realistic execution and costs |
| V4 | Full performance metrics and comparison with SPY |
| V5 | Ornstein-Uhlenbeck half-life and calibrated parameters |
| V6 | Dynamic hedge ratio (rolling regression, Kalman filter) and walk-forward re-selection |
| V7 | Multi-pair portfolio and risk allocation |
| V8 | Final out-of-sample validation on data never used before |
| V9 | Paper trading |

## References
- KidQuant, *Pairs-Trading-With-Python*: https://github.com/KidQuant/Pairs-Trading-With-Python
- Quantopian, *Introduction to Pairs Trading* (Lecture 42), CC BY 4.0, hosted by QuantRocket: https://www.quantrocket.com/codeload/quant-finance-lectures/quant_finance_lectures/Lecture42-Introduction-to-Pairs-Trading.ipynb
- Gatev, Goetzmann, Rouwenhorst (2006), *Pairs Trading: Performance of a Relative-Value Arbitrage Rule*, Review of Financial Studies 19(3), 797-827.
- Avellaneda, Lee (2010), *Statistical arbitrage in the US equities market*, Quantitative Finance 10(7), 761-782.
- Bailey, López de Prado (2014), *The Deflated Sharpe Ratio*, Journal of Portfolio Management 40(5), 94-107.
- statsmodels documentation for `coint`: https://www.statsmodels.org/dev/generated/statsmodels.tsa.stattools.coint.html


## Disclaimer
 
Educational project. Backtests are hypothetical and are not investment advice.
