# Principal Portfolios with Dynamic Risk Overlays

Dynamic, implementation-aware risk overlays for investable PCA principal portfolios.

This repository studies whether **predictive distributions of future risk** can improve the return path of an already-investable walk-forward PCA factor portfolio. The focus is not the PCA extraction problem itself, but the **overlay layer**: how to scale exposure to PC1 using probabilistic volatility-state information, realistic transaction-cost proxies, and out-of-sample validation.

> **Working paper status.** SSRN preprint submitted, Abstract ID: **6781859**. Link will be added once the SSRN page is live.

---

## 1. Research Question

Can forward-looking probabilistic risk signals improve the **net downside-adjusted performance** of investable principal portfolios?

The current testbed is **PC1** extracted from a US large-cap technology equity universe using walk-forward correlation-PCA.

The retained result is an **OU-style volatility-exceedance overlay** selected on purged/embargoed development folds and evaluated out of sample net of Corwin--Schultz spread-based implementation costs.

---

## 2. Current Reference Result

Reference OOS run: **Jan 2018--Apr 2026**, aligned sample ending **2026-04-14**.

| Strategy | CAGR | AnnVol | Sharpe | MaxDD | Martin |
|---|---:|---:|---:|---:|---:|
| PC1 base | 20.74% | 25.25% | 0.873 | -35.85% | 1.868 |
| Retained OU overlay, net CS | 18.73% | 19.56% | 0.976 | -26.41% | 2.213 |
| Combined calibrated, net CS | 18.83% | 20.24% | 0.954 | -28.81% | 2.087 |
| Vol target 20%, net CS | 17.37% | 18.68% | 0.951 | -26.53% | 1.943 |
| QQQ | 19.18% | 23.89% | 0.854 | -35.12% | 1.689 |

**Interpretation.** The retained OU overlay improves the path quality of PC1: it reduces volatility and maximum drawdown materially and improves Sharpe/Martin, while giving up some CAGR. The result should be read as downside/path-quality improvement, not unconditional alpha generation.

---

## 3. Bootstrap Evidence

OOS uncertainty is assessed using the **Politis--Romano stationary bootstrap** with `N = 2000` resampled paths. The reference run uses a mean block length of **90 trading days**, selected by ACF-matching on absolute QQQ returns to preserve volatility clustering and drawdown-path dependence.

Retained OU overlay versus PC1 base:

| Metric | Mean delta | 5% | Median | 95% | P(target wins) |
|---|---:|---:|---:|---:|---:|
| Delta Sharpe | 0.0950 | -0.0348 | 0.0955 | 0.2190 | 89.35% |
| Delta MaxDD | 0.0745 | 0.0053 | 0.0768 | 0.1383 | 98.30% |

The drawdown improvement is the most robust part of the result, which is consistent with the overlay's design objective.

---

## 4. Framework

### 4.1 Base Portfolio

- US large-cap technology equity universe.
- Walk-forward PCA on the return correlation matrix.
- Factor-mimicking principal portfolios normalized to unit gross exposure.
- PC1 is used as the investable baseline.
- Base PC1 returns are net of PC rebalance costs estimated from Corwin--Schultz one-way spread proxies.

### 4.2 Retained Slow Layer: OU Volatility-Exceedance Overlay

The retained layer fits an OU-style model to positive realized-volatility state variables. For a candidate state variable and horizon, the model derives a predictive distribution for future volatility and computes:

\[
p_t = \mathbb{P}(X_{t+H} > \tau \mid \mathcal{F}_t).
\]

The score is mapped into exposure states using percentile thresholds and hysteresis. The retained configuration is selected using development-sample predictive discrimination, not final OOS performance.

### 4.3 Medium Layer: HAR-RV Residual Bootstrap

A HAR-RV model forecasts forward realized volatility. Residual bootstrapping converts the HAR point forecast into a predictive distribution and exceedance probability.

This layer is treated as a **medium-horizon challenger/component**. It is useful in some subperiods, but the standalone retained result remains the slow OU overlay.

### 4.4 Slow-Layer Challengers: CIR and Rough-Volatility Diagnostic

The notebook includes a positive-variance **CIR challenger** for the slow layer. It is not retained in the current reference result.

The notebook also estimates the Hurst exponent of the volatility state as a rough-volatility diagnostic. Rough volatility is not promoted to a retained model unless evidence for \(H < 0.5\) is stable across walk-forward windows.

### 4.5 Fast Tactical Layer: Kalman/Mahalanobis Challengers

The notebook includes robust, hybrid, and gated Kalman/Mahalanobis fast-layer challengers.

The best admissible fast variant in the current reference run is the robust backward-looking Kalman layer, but it does not dominate the retained OU layer. The gated Kalman design remains an active research extension rather than a retained result.

### 4.6 Combined Decision Rules

The final validation compares:

- standalone slow OU overlay;
- medium HAR-RV;
- fast Kalman challenger;
- fixed weighted-deficit combination;
- calibrated weighted-deficit combination;
- minimum-exposure rule;
- deterministic volatility-targeting and drawdown-control baselines.

In the current reference run, the calibrated combination assigns:

```text
slow = 0.70
medium = 0.30
fast = 0.00
```

This is evidence that the current fast layer is not yet valuable enough in the global decision rule. It is not evidence that the fast-layer research path should be removed.

---

## 5. Transaction-Cost Model

Transaction costs are estimated using Corwin--Schultz high-low spread proxies:

1. estimate full proportional spread per asset;
2. convert full spread to one-way cost by taking half the spread;
3. aggregate one-way costs through the absolute PC1 weights;
4. charge PC rebalance costs in the base PC1 series;
5. charge incremental overlay costs when scalar exposure changes.

For an overlay exposure \(e_t\), incremental overlay cost is approximated as:

\[
TC_t = |\Delta e_t| \cdot c^{PC1}_t,
\]

where

\[
c^{PC1}_t =
\frac{\sum_i |w_{i,t}| c^{CS}_{i,t}}{\sum_i |w_{i,t}|}.
\]

The underlying PC1 one-way basket cost is common across overlays. Layer-level costs differ because layers trade with different frequencies and turnover.

Reference cost diagnostics:

| Layer | Turnover-weighted cost (bps) | Avg. overlay turnover | Total overlay cost | Trade days |
|---|---:|---:|---:|---:|
| Slow OU selected | ~25.7 | 0.0036 | 1.84% | 29 |
| Medium HAR | ~25.7 | 0.0036 | 1.75% | 26 |
| Fast Kalman robust | ~25.7 | 0.0076 | 5.28% | 129 |
| Combined calibrated | ~25.7 | 0.0036 | 1.82% | 51 |
| Combined minimum | ~25.7 | 0.0080 | 5.20% | 112 |

No hard-coded flat-bps cost is used for the main final results.

---

## 6. Validation Design

The final notebook reports:

- OOS period: **2018--2026**;
- walk-forward PCA re-estimation;
- PC1 net of rebalance costs;
- gross and Corwin--Schultz-net overlay performance;
- deterministic volatility-targeting and drawdown-control benchmarks;
- QQQ and equal-weight universe benchmarks;
- subperiod analysis;
- stationary block-bootstrap inference;
- ACF-matched block-length selection on absolute QQQ returns.

The OU design sweep is selected using **development-sample AUC of the volatility-exceedance score**, not OOS Sharpe or OOS P&L. Purged and embargoed development folds are used to reduce leakage from overlapping forward labels.

---

## 7. Subperiod Interpretation

The layers have different behaviour across regimes and subperiods:

- **2018--2019:** the retained OU overlay reduces drawdown and volatility while preserving most of the return.
- **2020:** fast/medium layers can become competitive because tactical stress and recovery dynamics are more important.
- **2021:** overlays naturally create some upside drag in a strong bull market.
- **2022:** the slow OU layer provides the clearest protection in the bear-market stress period.
- **2023--2026:** HAR-RV and fast-layer variants can become more competitive, suggesting that a future version should test regime-conditioned or walk-forward adaptive layer weights.

This is why the current combined architecture is treated as a research extension. A journal-quality multi-layer version should test adaptive or regime-conditioned layer weights with ablation studies.

---

## 8. Repository Structure

```text
.
├── principal_portfolios_with_dynamic_risk_overlays.ipynb
├── README.md
└── requirements.txt
```

---

## 9. How to Run

1. Install dependencies:

```bash
pip install -r requirements.txt
```

2. Open the notebook:

```bash
jupyter notebook principal_portfolios_with_dynamic_risk_overlays_clean.ipynb
```

3. Run cells top-to-bottom.

The notebook downloads data from Yahoo Finance through `yfinance`. Results can change slightly if the data vendor revises adjusted prices, high/low fields, or index histories.

---

## 10. Methodological Caveats

- The universe is a transparent US large-cap technology sample, not a fully point-in-time CRSP universe.
- PC1 is the current base portfolio; the overlay framework should be tested on additional universes before claiming broad generality.
- The current retained result is the standalone slow OU overlay; HAR-RV, CIR, Kalman, and combined rules are challengers/extensions.
- The layer-combination policy is currently selected on development folds and then held fixed OOS. A future version should test walk-forward or regime-conditioned policy recalibration.
- The fast Kalman/Mahalanobis layer is methodologically interesting but not yet a retained headline result.

---

## 11. Planned Extensions

- Walk-forward or regime-conditioned layer weights.
- Cross-market validation across US broad equity, Europe, Japan, sectors, and factor/ETF universes.
- Full ablation study: OU-only, HAR-only, fast-only, gated vs non-gated, weighted deficit vs min-rule.
- Rough-volatility challenger if the Hurst diagnostic supports stable \(H < 0.5\).
- Heston / risk-neutral-to-physical bridge using option-implied information.
- Cost-sensitivity analysis around Corwin--Schultz estimates.

---

## 12. Author

Saverio Lauriola
