# Principal Portfolios with Dynamic Risk Overlays

### Dynamic, implementation-aware risk overlays for investable PCA factor portfolios

## Overview

This repository studies **dynamic risk overlays** applied to **investable principal portfolios** extracted from equity returns.

The starting point is a PCA factor-portfolio framework in which **PC1** emerges as the most robust investable baseline. This project then asks whether its return path can be improved through **dynamic, implementation-aware overlays** built from forward-looking risk signals.

This work sits at the intersection of quantitative risk modelling and systematic portfolio construction — using risk forecasting tools typically associated with risk management to improve investment decisions.

The current framework focuses on a **slow volatility-distribution overlay** based on:

- **HAR-RV forecasting** for forward realized volatility
- **Residual bootstrapping** to obtain a predictive distribution, not just a point forecast
- **Percentile-based state filters** with hysteresis and mild de-risking
- **Spread-based implementation costs** via Corwin–Schultz one-way cost proxies

The broader research agenda also includes:

- **Short-horizon implied-volatility signals** for tactical risk control
- **Alternative predictive engines** for volatility distributions, including OU/CIR challengers
- **Downside-aware and regime-aware overlay rules** beyond pure volatility targeting

This repository is therefore about **overlay design**, not factor extraction per se.

---

## Relationship to the PCA Base Project

This project builds on a separate upstream repository centered on **PCA principal portfolios in risk regimes (https://github.com/svr-L/pca-portfolios-in-risk-regimes)**.

There, the main question is whether principal portfolios extracted from an equity universe are investable, regime-sensitive, and robust once implementation is taken seriously.

Here, the focus is narrower and more practical:

> Given an investable principal portfolio baseline, can dynamic overlays improve its **net downside-adjusted performance** without materially damaging carry?

In the current implementation, the main testbed is **PC1**.

---

## Research Question

Can predictive distributions of future risk be used to design **dynamic overlays** that improve the **net risk-adjusted performance** of investable principal portfolios?

More specifically:

1. Does the **distribution of forward realized volatility** contain useful information for de-risking principal portfolios?
2. Can a **slow, low-turnover overlay** improve downside control while preserving most of the base strategy's return profile?
3. Do **short-horizon IV signals** add value as tactical overrides on top of the slow overlay?

---

## Current Framework

### 1) Base Strategy

- Investable PCA factor portfolios extracted from equity returns
- Current focus: **PC1** as the most robust baseline exposure
- Walk-forward construction to avoid static in-sample weights

### 2) Slow Overlay Layer

- **Signal engine:** HAR-RV model for forward realized volatility
- **Distribution engine:** residual bootstrap around the HAR conditional mean
- **Overlay rule:** expanding-percentile state filter with hysteresis
- **Trading frequency:** slow / low-turnover by design

### 3) Fast Overlay Layer *(under development)*

- Short-horizon **implied-volatility signals**
- Intended role: tactical override during short-lived stress episodes
- Not yet retained as the main signal engine

### 4) Cost Model

- Asset-level **Corwin–Schultz** spread-based proxies
- Aggregation through the underlying stock weights of the principal portfolio
- Incremental overlay costs computed from changes in factor exposure

---

## Key Findings

### 1) The slow overlay is economically meaningful

Using a **HAR-RV + residual bootstrap** predictive distribution and a mild percentile-based state rule, the **net OOS** profile of the PC1 strategy improves meaningfully.

### 2) The benefit comes from better path quality, not higher raw CAGR

Relative to the base PC1 OOS strategy, the current slow overlay:

- reduces annualized volatility
- materially reduces max drawdown
- improves Sharpe, Sortino, Calmar, and Martin ratios
- only modestly reduces CAGR

### 3) The score is more useful as a ranking/regime detector than as a perfectly calibrated probability

The predictive distribution is especially useful for identifying **high-risk volatility states**, which makes percentile-based overlay rules more natural than continuous probability-to-exposure mappings.

### 4) Drawdown-event logit models were explored, but not retained as the main engine

Direct event models for forward drawdown were tested as challengers, but so far they have been less robust than volatility-distribution-based overlays.

---

## Current Reference Result

For the current **PC1 OOS** testbed, the slow overlay improves the base profile approximately as follows:

| Strategy | CAGR | Sharpe | Sortino | MaxDD | Calmar | Martin |
|---|---:|---:|---:|---:|---:|---:|
| PC1 base OOS | 37.66% | 1.62 | 2.25 | -24.33% | 1.55 | 7.60 |
| PC1 slow overlay (net, CS) | 36.63% | 1.74 | 2.49 | -17.78% | 2.06 | 8.03 |

**Interpretation:**

- CAGR drag is modest (-1.0pp)
- Downside control improves materially (MaxDD from -24.3% to -17.8%)
- Risk-adjusted performance improves across all metrics, even after costs
- Total overlay cost: ~0.59% over the OOS period, with only 9 rebalance events in 761 days

---

## Methodology

### Predictive Risk Layer

- HAR-RV mean model for forward realized volatility
- Residual bootstrap to obtain predictive distributions
- Threshold events of the form `P(RV_fwd(t, t+H) > tau | F_t)`

### Decision Layer

- Expanding historical percentiles of the risk score
- Mild de-risking states (rather than aggressive all-in/all-out switching)
- Hysteresis to reduce unnecessary turnover

### Validation Layer

- Development / OOS split
- Net-of-cost evaluation via Corwin–Schultz spread-based proxies
- Path-quality metrics (Sharpe, Sortino, MaxDD, Calmar, Martin), not only raw return metrics

---

## Why This Repository Exists

Many systematic strategies look good in backtests but become much less attractive once one accounts for:

- unstable risk regimes
- weak signal calibration
- turnover
- implementation costs
- path-dependent investor pain

This project asks a practical question:

> If the base factor portfolio is already investable, can we improve the quality of the return path with overlays that are dynamic, risk-based, and still realistic to trade?

---

## Planned Extensions

- **Fast implied-volatility override** on top of the slow overlay
- **OU/CIR challengers** for volatility-distribution modelling
- **Alternative downside-aware overlay rules**
- **Abdi–Ranaldo execution-cost robustness checks**
- **Extension beyond PC1** to other principal portfolios when justified by stability and costs

---

## Limitations

- Current results are based on a ~3-year OOS window (2023–2025); the sample includes limited stress episodes and results should be interpreted accordingly
- Corwin–Schultz costs are **spread-based implementation proxies**, not full execution-cost models
- Probability forecasts are currently most useful as **risk-state scores**, not as perfectly calibrated literal probabilities
- The framework is still evolving; this repository documents the overlay layer as a standalone research track

---

## Repository Structure

```
.
├── principal_portfolios_with_dynamic_risk_overlays.ipynb
├── README.md
└── requirements.txt
```

---

## Related Work

This repository is part of a broader research programme on systematic factor investing:

- **[PCA Portfolios in Risk Regimes](https://github.com/svr-L/pca-portfolios-in-risk-regimes)** — upstream project on PCA factor extraction, regime-conditional performance, and investability of principal portfolios

---

## Author

**Saverio Lauriola**
