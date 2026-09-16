# Conditional Evaluation of Machine Learning Portfolio Strategies

A functional inequality approach to asking whether machine learning portfolio strategies beat traditional Fama-French benchmarks *in every market state*, not just on average. Applying the test of Li, Liao and Zhou (2025) to 7,043 trading days of U.S. equity data (January 1998 to December 2025), conditioning on the CBOE VIX, no ML strategy conditionally dominates any traditional benchmark in any volatility regime. Constructing a confidence set for the most superior strategy across all eight candidates, only the 1/N equal-weight portfolio survives.

## Key Results

Annualized performance over the full sample:

| Strategy | Ann. Mean | Ann. Vol | Sharpe |
|----------|-----------|----------|--------|
| Market | 0.088 | 0.196 | 0.452 |
| FF3 | 0.036 | 0.089 | 0.408 |
| FF5 | 0.033 | 0.055 | 0.596 |
| EqWeight (1/N) | 0.117 | 0.215 | 0.547 |
| ML LASSO | −0.001 | 0.120 | −0.008 |
| ML Ridge | 0.007 | 0.115 | 0.061 |
| ML Random Forest | 0.001 | 0.147 | 0.005 |
| ML LightGBM | −0.017 | 0.122 | −0.135 |

Bilateral functional inequality tests at the 5% level (ρ = 0.001):

| Benchmark | Alternatives | Statistic | Critical Value | Reject |
|-----------|--------------|-----------|----------------|--------|
| Market | ML (all 4) | 2.337 | 3.010 | No |
| FF5 | ML (all 4) | 2.521 | 3.048 | No |
| EqWeight | ML (all 4) | 2.675 | 3.036 | No |
| ML Ridge | Traditional (all 4) | 3.668 | 2.998 | Yes |

The pattern is asymmetric. No ML strategy's conditional Sharpe ratio exceeds any traditional benchmark's at any VIX level, while the best ML strategy is itself conditionally dominated in specific regimes. The direction runs one way.

Confidence set for the most superior strategy, rotating the benchmark role across all eight:

| Strategy | Statistic | Critical Value | In CSMS |
|----------|-----------|----------------|---------|
| Market | 3.398 | 3.181 | No |
| FF3 | 4.041 | 3.242 | No |
| FF5 | 3.515 | 3.244 | No |
| EqWeight (1/N) | 3.198 | 3.212 | **Yes** |
| ML LASSO | 3.376 | 3.232 | No |
| ML Ridge | 3.668 | 3.232 | No |
| ML Random Forest | 4.971 | 3.193 | No |
| ML LightGBM | 4.798 | 3.207 | No |

This is a conditional counterpart to the unconditional finding of DeMiguel, Garlappi and Uppal (2009) that the 1/N portfolio is hard to beat.

## Why Conditional Evaluation Changes the Answer

The most instructive result in this project is the FF5 case. FF5 has the highest unconditional Sharpe ratio in the sample at 0.596, higher than the 1/N portfolio's 0.547. It is nonetheless excluded from the confidence set, because its conditional Sharpe ratio function falls below 1/N's in specific VIX regimes and the test statistic (3.515) exceeds the critical value (3.244).

That is exactly the distinction unconditional evaluation cannot make. A strategy can hold the best average performance in the sample while still being conditionally dominated in the market states an investor most cares about. Averaging over regimes hides it.

The same logic explains the Market's exclusion. Its conditional Sharpe ratio sits near 0.5 when VIX is close to 10 and falls below zero above roughly VIX 25, while the equal-weight portfolio stays flatter across the conditioning space.

**The EqWeight result is borderline and is reported as such.** The statistic is 3.198 against a critical value of 3.212, a margin of 0.014. The qualitative conclusion holds, but the membership is suggestive rather than decisive.

## Methodology

The test evaluates the null hypothesis that a benchmark's conditional Sharpe ratio is at least as large as every alternative's, pointwise across the full support of the conditioning variable. Rejection implies there exists some market state in which some alternative does better.

**Conditioning variable.** The CBOE VIX, following the recommendation in Li et al. (2025). The raw series is right-skewed, so it is transformed by taking logs, standardizing, and mapping through the standard normal CDF onto [−1, 1] with an approximately uniform marginal.

**Nonparametric estimation.** Conditional means and variances are estimated by sieve regression on orthonormal Legendre polynomials, with sieve dimension K = max{4, ⌊1.2n^(1/5)⌋}, giving K = 7 at n = 7,043.

**Shape-constrained variance.** The unconstrained sieve estimate of the conditional variance can produce negative fitted values at some points, a finite-sample artifact of polynomial approximation. A quadratic programming step enforces a data-driven positive lower bound that becomes asymptotically unbinding.

**Generated-variable correction.** Normalizing returns by an estimated conditional standard deviation makes the dependent variable in the Sharpe ratio regression itself an estimate. Ignoring this understates sampling variability and biases the test toward over-rejection, so the covariance matrix is built from an influence function carrying a correction term for the sensitivity of the Sharpe ratio coefficients to the variance estimate.

**Critical values.** A two-step Bonferroni procedure following Romano, Shaikh and Wolf (2014), with ρ ∈ {0.001, 0.005}, α = 0.05, and 10,000 Gaussian simulation draws. The supremum is evaluated over 950 grid points trimmed at the 2.5% and 97.5% quantiles.

## Portfolio Construction

**Traditional benchmarks.** Market excess return, equal-weight combinations of the three and five Fama-French factor portfolios, and the 1/N equal-weight average of the 25 sorted portfolios.

**ML strategies.** Rebalanced monthly on a rolling 60-month window. At each rebalance a panel of 25 portfolios × 60 months = 1,500 observations is assembled from 11 features: three lagged own returns, seven lagged factor values (the five Fama-French factors, momentum, and the risk-free rate), and the cross-sectional percentile rank of the most recent month's return. Features are standardized within the training window. The five portfolios with the highest predicted returns take equal long weights of 1/5, the five lowest take short weights of −1/5, held through the following month.

Four models: LASSO (α = 0.001), Ridge (α = 1.0), Random Forest (100 trees, max depth 5, min leaf 10), and LightGBM (100 rounds, max depth 4, learning rate 0.05).

## Robustness Checks

| Check | Result |
|-------|--------|
| Pairwise tests (Market vs. each ML model) | Statistics 1.71 to 2.34, no rejection |
| Sieve dimension K = 5 to 8 | No rejection throughout; K = 4 marginally rejects |
| Cross-validated hyperparameter tuning | No rejection (2.97 vs. 3.05); ML performance does not improve |
| Subsample split at January 2010 | No rejection in either half |
| Transaction costs, 10 bps per unit turnover | No rejection (2.23 vs. 3.00) |
| Newey-West HAC covariance | **Marginal rejection** (3.16 vs. 3.01) |

The HAC result is the one that moves. The baseline treats the influence function as a martingale difference sequence, motivated by a no-arbitrage argument for daily excess returns. But the influence function involves nonlinear transformations of returns, and those need not inherit the MDS property. Under HAC the statistic rises from 2.34 to 3.16 and the test marginally rejects. Whether that is signal or a finite-sample artifact of the HAC estimator's own variance cannot be settled from the data alone, so the MDS specification is kept as the primary result and the HAC outcome is reported as a stress test of that assumption rather than buried.

## Repository Structure

```
├── cspa_analysis.ipynb                  # Full analysis
├── capstone_cspa_ml-Rudransh Khera.pdf  # Paper
├── csr_plot.pdf                         # Conditional Sharpe ratio functions
├── cumulative_returns.pdf               # Cumulative wealth paths
├── requirements.txt
└── data/raw/                            # Not tracked, see below
```

## Data

The data files are not committed. All twelve are freely available and can be reassembled in a few minutes.

**Kenneth French Data Library** (https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html), downloaded as CSV:

- `25_Portfolios_5x5_Daily.csv`
- `25_Portfolios_5x5.csv`
- `100_Portfolios_ME_OP_10x10.csv`
- `F-F_Research_Data_5_Factors_2x3_daily.csv`
- `F-F_Research_Data_5_Factors_2x3.csv`
- `F-F_Momentum_Factor.csv`

**FRED** (https://fred.stlouisfed.org), one CSV per series ID:

- `VIXCLS.csv`, CBOE Volatility Index
- `AAA.csv`, Moody's Aaa corporate bond yield
- `BAA.csv`, Moody's Baa corporate bond yield
- `GS10.csv`, 10-year Treasury constant maturity rate
- `TB3MS.csv`, 3-month Treasury bill secondary market rate
- `USREC.csv`, NBER recession indicator

Place all twelve in `data/raw/`.

## How to Run

1. Download the data files as described above into `data/raw/`.
2. Install dependencies: `pip install -r requirements.txt`.
3. Open `cspa_analysis.ipynb` and run all cells in order.

All tables and figures are generated inline.

## Environment

Python 3.10, with `numpy`, `pandas`, `scipy`, `scikit-learn`, `statsmodels`, `lightgbm`, `matplotlib`, `seaborn`, `tqdm`.

## References

- Li, J., Liao, Z., & Zhou, W. (2025). A general test for functional inequalities. *Journal of Econometrics*, 251, 106063.
- Li, J., Liao, Z., & Quaedvlieg, R. (2022). Conditional superior predictive ability. *Review of Economic Studies*, 89(2), 843–875.
- DeMiguel, V., Garlappi, L., & Uppal, R. (2009). Optimal versus naive diversification: How inefficient is the 1/N portfolio strategy? *Review of Financial Studies*, 22(5), 1915–1953.
- Gu, S., Kelly, B., & Xiu, D. (2020). Empirical asset pricing via machine learning. *Review of Financial Studies*, 33(5), 2223–2273.
- Romano, J. P., Shaikh, A. M., & Wolf, M. (2014). A practical two-step method for testing moment inequalities. *Econometrica*, 82(5), 1979–2002.

## Context

Capstone project for the Master of Quantitative Economics at UCLA, June 2026. Faculty advisor: Professor Zhipeng Liao.

## Author

**Rudransh Khera**, Master of Quantitative Economics, UCLA

- [GitHub](https://github.com/kherarudransh-oss)
- [LinkedIn](https://www.linkedin.com/in/rudransh-khera/)
