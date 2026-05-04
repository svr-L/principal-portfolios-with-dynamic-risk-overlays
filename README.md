# Principal Portfolios with Dynamic Risk Overlays

Dynamic, implementation-aware risk overlays for investable PCA factor portfolios.

This repository studies whether probabilistic risk signals can improve the return path of an investable walk-forward PCA principal portfolio. The focus is not factor extraction itself, but the **overlay layer**: how to scale exposure to an already-investable PC1 portfolio using predictive volatility-state information, realistic transaction-cost proxies, and out-of-sample validation.

## Research Question

Can predictive distributions of future risk be used to design dynamic overlays that improve the **net downside-adjusted performance** of investable principal portfolios?

The current testbed is **PC1** extracted from a US large-cap technology equity universe using walk-forward correlation-PCA.

## Current Result

The retained result is an **OU-style log-volatility exceedance overlay**, selected by development-sample predictive discrimination and evaluated net of Corwin--Schultz spread-based implementation costs.

| Strategy | CAGR | AnnVol | Sharpe | MaxDD | Martin |
|---|---:|---:|---:|---:|---:|
| PC1 base | 20.5% | 25.3% | 0.86 | -35.9% | 1.85 |
| Selected OU overlay, net CS | 19.8% | 20.6% | 0.98 | -26.3% | 2.34 |

**Interpretation.** The retained overlay sacrifices modest CAGR while materially improving path quality: lower volatility, lower maximum drawdown, and higher Sharpe / Martin ratios over the 2018--2026 OOS period.

Stationary block-bootstrap inference supports the drawdown result: the selected OU overlay improves MaxDD versus PC1 with high bootstrap frequency, while the Sharpe improvement is positive but less mechanically guaranteed.

## Framework

### 1. Base Portfolio

- US large-cap technology equity universe
- Walk-forward PCA on the return correlation matrix
- Factor-mimicking principal portfolios normalized to unit gross exposure
- PC1 used as the robust investable baseline
- Base PC1 returns are net of rebalance costs estimated from Corwin--Schultz one-way spread proxies

### 2. Retained Slow Layer: OU Log-Volatility Exceedance

The main overlay fits an OU-style model to positive realized-volatility state variables. For a candidate state variable and horizon, the model derives a predictive distribution for future log-volatility and computes an exceedance score:

\[
p_t = \mathbb{P}(X_{t+H} > \tau \mid \mathcal{F}_t).
\]

The score is mapped into exposure states using percentile thresholds and hysteresis.

### 3. Medium Layer: HAR-RV Residual Bootstrap

A HAR-RV model forecasts forward realized volatility. Residual bootstrapping turns the point forecast into a predictive distribution and an exceedance probability. This layer is retained as a methodological benchmark and medium-speed component, but it is not the strongest standalone result in the current OOS sample.

### 4. Fast Tactical Layer

The implemented fast layer is a conservative IV-based tactical override. It is included as an architecture component / challenger. A natural next iteration is a Kalman innovation / Mahalanobis surprise detector for short-horizon stress.

### 5. Combined Architecture

The combined overlay takes the minimum exposure across the selected OU slow layer, HAR medium layer, and fast tactical layer. In the current results, the combined architecture is robust but does not dominate the selected OU layer net of costs.

## Cost Model

Transaction costs are estimated using Corwin--Schultz high-low spread proxies:

1. estimate full proportional spread per asset;
2. convert full spread to one-way cost by taking half the spread;
3. aggregate one-way costs through the absolute PC1 weights;
4. charge incremental overlay costs when scalar exposure changes.

For an overlay exposure \(e_t\), incremental overlay cost is approximated as:

\[
TC_t = |\Delta e_t| \cdot c^{PC1}_t,
\]

where

\[
c^{PC1}_t =
\frac{\sum_i |w_{i,t}| c^{CS}_{i,t}}{\sum_i |w_{i,t}|}.
\]

No hard-coded flat bps cost is used for the main final results.

## Validation

The final notebook reports:

- OOS period: **2018--2026**
- QQQ and equal-weight universe benchmarks
- gross and Corwin--Schultz-net performance for every overlay layer
- binding-layer diagnostics
- stationary block-bootstrap inference
- ACF-matched block-length selection on absolute QQQ returns

## Important Methodological Caveat

The OU design sweep is selected by **development-sample AUC of the volatility-exceedance score**, not by OOS Sharpe or OOS P&L.

This correction avoids OOS selection bias. However, the development window is short and calm; the overlay state machine may not trade in development (`dev_trades = 0` across candidates). For that reason, development-period overlay P&L is degenerate and cannot be used as a selection criterion. The AUC-based selection should therefore be interpreted as predictive-signal selection, with this limitation disclosed.

## Repository Structure

```text
.
├── principal_portfolios_with_dynamic_risk_overlays.ipynb
├── README.md
└── requirements.txt
```

## Relationship to Upstream Work

This repository is downstream of a separate PCA / regime-analysis project. The upstream project studies whether PCA principal portfolios are investable and regime-sensitive. This repository assumes PC1 as the investable base and focuses only on **dynamic risk overlays**.

## Planned Extensions

- replace / benchmark the fast IV layer with a Kalman innovation / Mahalanobis surprise overlay;
- add subperiod tables: 2018--2019, Covid, 2022 bear market, 2023--2026;
- add cost sensitivity around Corwin--Schultz estimates;
- add deterministic volatility-targeting and drawdown-control baselines;
- replicate the framework across additional universes, sectors, and regions.

## Author

Saverio Lauriola
