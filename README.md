# PCA Portfolios in Risk Regimes

### Can regime awareness make principal portfolios investable and profitable?

## Overview

This project studies whether principal components extracted from a panel of US tech equities can be treated as investable equity factors, and whether regime awareness improves their net, implementation-aware performance.

The framework combines:

- **Linear factor extraction:** correlation-PCA on daily equity returns
- **Factor portfolio construction:** factor-mimicking principal portfolios
- **Regime conditioning:** LOW / MID / HIGH states defined from an implied-volatility proxy
- **Robust estimation:** flexible probabilities, shrinkage covariance, and walk-forward re-estimation
- **Validation:** Politis–Romano stationary block bootstrap
- **Implementability checks:** turnover, loading stability, and spread-based transaction-cost proxies
- **Risk management extension:** a slow, probability-based volatility overlay built from HAR-RV plus residual bootstrapping

## Research Question

Do higher-order principal portfolios become attractive in specific risk regimes, or does a simple buy-and-hold exposure to **PC1** remain the most robust investable choice once robustness, implementation, and dynamic risk control are taken seriously?

---

## Main Findings

### 1) PC1 is the strongest baseline exposure

Across bootstrap validation and walk-forward testing, **PC1** emerges as the most robust buy-and-hold principal portfolio.

In the current stationary block bootstrap (**N = 1000**), **PC1 gross** delivers approximately:

- **42.6% median CAGR**
- **1.59 median Sharpe ratio**
- **7.22 median Martin ratio**

### 2) Higher-order PCs are more regime-dependent

Higher PCs can look attractive in selected regimes, especially on conditional risk-adjusted metrics, but they are materially less stable through time.

### 3) Stability and implementability matter

Walk-forward PCA shows a clear trade-off:

- **PC1:** more stable loadings and lower turnover
- **Higher PCs:** noisier rebalances and larger turnover spikes

This means that apparent performance improvements in higher PCs are harder to monetize in practice.

### 4) A slow volatility overlay improves the base strategy net of costs

A slow overlay built from the **predictive distribution of forward realized volatility** improves the downside profile of the PC1 walk-forward strategy.

Using a **HAR-RV mean model + residual bootstrap** to score future volatility states, then applying a mild percentile-based exposure rule with **Corwin–Schultz spread-based costs**, the **net OOS** profile improves from:

- **Sharpe:** 1.62 -> **1.74**
- **Sortino:** 2.25 -> **2.49**
- **Max Drawdown:** -24.3% -> **-17.8%**
- **Calmar:** 1.55 -> **2.06**
- **Martin:** 7.60 -> **8.03**

while only modestly reducing CAGR:

- **CAGR:** 37.7% -> **36.6%**

This makes the overlay economically meaningful rather than purely statistical.

---

## Methodology

### Universe and frequency

- **Universe:** US tech equities
- **Frequency:** daily returns

### Factor construction

- Correlation-based PCA on the equity return panel
- Conversion of eigenvectors into **factor-mimicking principal portfolios**
- Walk-forward re-estimation to avoid static in-sample weights

### Regime definition

- Regimes classified as **LOW / MID / HIGH** using an implied-volatility state variable

### Risk and performance metrics

- CAGR / Excess CAGR
- Sharpe ratio
- Sortino ratio
- Calmar ratio
- Martin ratio
- Max drawdown
- Ulcer index
- Turnover
- Loading stability / cosine similarity

### Validation

- **Politis–Romano stationary block bootstrap**
- **N = 1000** simulations
- Dependency-aware resampling to assess robustness beyond the realized path

### Implementation-cost layer

- Asset-level **Corwin–Schultz** bid-ask spread estimation from daily high/low data
- Conversion from full spread to **one-way spread-based cost proxy**
- Aggregation of costs through the turnover of the underlying stock weights of each PC portfolio
- Additional overlay costs computed from changes in the factor exposure state

### Risk overlay layer

- **Signal engine:** HAR-RV model for forward realized volatility
- **Distribution engine:** residual bootstrap around the HAR conditional mean
- **Decision rule:** slow percentile-based state filter with hysteresis and mild de-risking
- **Current role:** improve net downside-adjusted performance without heavily impairing carry from the base PC1 exposure

---

## Selected Results

### Bootstrap validation

For **PC1 (overall, gross)**, the stationary block bootstrap indicates strong and persistent performance, with median values around:

| Metric | Median |
|---|---:|
| CAGR | 42.6% |
| Sharpe | 1.59 |
| Martin | 7.22 |

This supports the view that the strongest principal portfolio is not just an in-sample artifact.

### Walk-forward baseline

In walk-forward testing, **PC1** remains the cleanest investable baseline:

- about **42.0% gross CAGR** overall
- about **35.7% excess CAGR** overall
- about **1.55 Sharpe** overall

Spread-based costs modestly reduce performance, but do **not** overturn the ranking of PC1 as the most robust principal portfolio.

### Walk-forward with slow overlay (net)

| Strategy | CAGR | Sharpe | Sortino | MaxDD | Calmar | Martin |
|---|---:|---:|---:|---:|---:|---:|
| PC1 base OOS | 37.66% | 1.62 | 2.25 | -24.33% | 1.55 | 7.60 |
| PC1 slow overlay net | 36.63% | 1.74 | 2.49 | -17.78% | 2.06 | 8.03 |

The overlay is intentionally **mild**: it trades infrequently, pays limited incremental costs, and improves the quality of the return path rather than maximizing raw CAGR.

---

## Interpretation

This project is **not** a claim that higher PCs are useless.

Rather, it shows that:

- higher PCs can carry useful regime information,
- but their attractiveness is conditional,
- and the jump from “interesting backtest” to “investable factor” depends heavily on stability, turnover, and risk control.

The updated contribution is that **risk management overlays should be built where the predictive structure is strongest**. In this project, direct drawdown-event models were explored but did not prove robust enough to retain as the main overlay engine. By contrast, the **distribution of future realized volatility** proved much more useful for constructing a practical slow overlay.

In that sense, the project sits at the intersection of:

- quantitative risk modelling,
- factor investing,
- and implementation-aware portfolio construction.

---

## Strategy Implications

### Natural benchmark

- **Buy & Hold PC1**

### Current preferred extension

- **Slow volatility-distribution overlay on PC1**

### Additional extensions under study

- Short-horizon implied-volatility overrides
- Regime-based rotation across principal portfolios
- Cross-country or cross-universe PCA rotation
- Richer execution-cost modelling beyond spread-only proxies
- Alternative volatility-state models (for example OU/CIR challengers to HAR-based distributions)

---

## Limitations

- Corwin–Schultz costs should be interpreted as **spread-based implementation proxies**, not a full execution-cost model
- The volatility overlay currently uses **HAR + residual bootstrap** as its baseline predictive engine; alternative distributional models are still under study
- Regime-conditional sub-sample metrics are informative diagnostically, but not all of them correspond directly to continuous investable strategies
- Results remain sample-dependent and should be interpreted jointly with walk-forward and bootstrap evidence

---

## Repository Structure

- `PCA_Portfolios_in_Risk_Regimes.ipynb` — main notebook
- `requirements.txt`
- `README.md`

---

## Author

**Saverio Lauriola**
