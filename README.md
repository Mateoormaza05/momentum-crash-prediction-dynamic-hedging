# Momentum Crash Prediction & Dynamic Hedging

## Motivation
Daniel & Moskowitz (2016) documented that classical momentum strategies have negatively skewed returns driven by time-varying risk exposures. Under market downturns, the short "loser" portfolio tends to populate with high-beta equities that have fallen with the market. In this case, the long "winner" portfolio tends to consist lower-beta equities that have resisted market conditions. In this scenario, sharp market upturns cause the high-beta short portfolio to grow at higher rates than the low-beta long portfolio, leading to unprofitable crashes. This raises the question of whether hedging against market conditions using signals (volatility, recent market returns) would improve returns.

## Research Question
Can lagged market signals help predict momentum crashes and improve risk-adjusted returns?

## Data
- Kenneth French Data Library: UMD monthly returns, Fama-French 3 factors (2000-2024)

- FRED: CBOE VIX monthly closing values (2000-2024)

- 300 monthly observations after feature construction

- All signals lagged one month to ensure ex-ante implementability

## Methodology
- Crashes were identified through the bottom 10th percentile of monthly UMD returns in training period (2000-2009).

- The originally selected signals were VIX, realized volatility, 12-month market returns, 6-month momentum returns (all lagged) based on Daniel & Moskowitz (2016) and economic rationale. After conducting univariate t-tests, 6-month momentum returns were found to not be a significant predictor of market crashes. Furthermore, joint logistic regression found realized volatility to have multicollinearity with VIX. 6-month momentum returns were exluded for lack of statistical significance, and realized volatility was excluded because it had no independent predictive power conditional on VIX (logistic regression p-value = 0.952).

- Z-scores were computed using training window parameters only and applied to the full period. The composite risk score weights both signals equally (0.5 × VIX_lag_z − 0.5 × Mkt12m_lag_z). 

- Exposure is scaled continuously using training window p10/p90 bounds: exposure = 1 − (risk_score − p10) / (p90 − p10), clipped to [0, 1]. High risk score → low exposure.

- Walk-forward validation was done using an expanding window, with 2000-2009 as the training window and 2010-2024 as OOS.

## Results

### Sub-Period Sharpe

| Period | Dynamic Sharpe | Baseline Sharpe |
|---|---|---|
| 2010–2024 (Full OOS) | 0.401 | 0.147 |
| 2010–2017 | 0.408 | 0.294 |
| 2018–2024 | 0.392 | 0.034 |

### Full OOS Metrics

| Metric | Dynamic | Baseline |
|---|---|---|
| Sharpe (2010–2024) | 0.401 | 0.147 |
| Sortino (2010–2024) | 0.636 | 0.199 |
| Max Drawdown | -15.1% | -28.5% |
| Crash-Conditional Return | -2.1% | -12.6% |
| Mean OOS Exposure | 74% | 100% |
| FF3 Alpha (monthly) | 0.27% (p=0.098) | — |
| Sharpe at 50bps TC | 0.240 | — |


## Limitations
- Walk-forward Jobson-Korkie test (p-value = 0.2430) showed no significant change in Sharpe ratio despite metrics mentioned above. This test was underpowered at 180 monthly observations.

- UMD factor returns are not directly tradeable. Real implementation requires trading the underlying long-short equity book, which carries higher transaction costs and capacity constraints.

- Transaction costs only calculated on overlay.

- Structural break in momentum post-2009.

## References

Daniel, K. & Moskowitz, T. (2016). Momentum crashes. 
Journal of Financial Economics, 122(2), 221-247.

Barroso, P. & Santa-Clara, P. (2015). Momentum has its moments. 
Journal of Financial Economics, 116(1), 111-120.
