# BP2026
This repository contains the source code for the bachelor's thesis:
**An Analysis of the Performance of Algorithmic Trading Strategies Based on Cointegration**
(Analýza úspěšnosti algoritmických obchodních strategií založených na kointegraci) 
Prague University of Economics and Business (VŠE Praha), 2025

The repository has four folders — **BETELGEUSE**, **ALDEBARAN**, **SIRIUS**, and **RIGEL** — each containing a Jupyter notebook with the strategy logic and a `DATA/` folder with the corresponding results.

---

## Notebook Structure

All three notebooks share the same four-cell structure:

**Cell 1 — Schedule generator**  
Generates the walk-forward schedule (`CSV/schedule.csv`). The user inputs the OOS start year, end year, and IS window length in months. Each row in the output represents one IS/OOS iteration.

**Cell 2 — In-Sample (IS) analysis**  
Loads price data and settings, then screens all stock pairs within each GICS sector. For each pair it calculates correlation, runs the Engle-Granger cointegration test, computes the hedge ratio (OLS), half-life, Hurst exponent, zero crossings, and an IS Sharpe ratio simulation. Pairs that pass all filter thresholds defined in `CSV/settings.csv` are saved to `DATA/is_results.xlsx`.

**Cell 3 — Out-of-Sample (OOS) backtest**  
Loads the pairs selected in Cell 2 and simulates live trading over the OOS month. Manages a portfolio of up to `max_open_pairs` positions. Entry when |Z-score| >= `z_entry`, exit when |Z-score| <= `z_exit` or after a time-based stop. Results are saved to `DATA/oos_results.xlsx`.

**Cell 4 — Snapshot / archive**  
Copies `is_results.xlsx`, `oos_results.xlsx`, and the settings into a timestamped subfolder under `DATA/STAT_z/` so each run is preserved independently.

---

## Scripts

### BETELGEUSE — Strategy I

Notebook: `BETELGEUSE/15-BETELGEUSE.ipynb`

Baseline strategy. The spread Z-score is calculated using an **expanding window** — all available history up to the current point in time is used. No additional market-regime filter is applied.

---

### ALDEBARAN — Strategy II

Notebook: `ALDEBARAN/15-ALDEBARAN.ipynb`

The spread Z-score is calculated using a **rolling window** (`z_window = 60` trading days). This makes the Z-score more responsive to recent spread dynamics compared to Strategy I.

---

### SIRIUS — Strategy III

Notebook: `SIRIUS/15-SIRIUS.ipynb`

Same rolling Z-score as Strategy II (`z_window = 60`), with an additional **VIX filter**: new positions are blocked when the VIX Z-score (252-day rolling window) exceeds the threshold defined in settings (`vix_z_threshold = 2.0`). This prevents entering trades during high-volatility market regimes.

---

### RIGEL — DAX experiment (not part of thesis)

Notebook: `RIGEL/15-RIGEL.ipynb`

A copy of ALDEBARAN adapted for the German equity market (DAX index). Used as an exploratory test to check whether the same rolling Z-score methodology transfers to a different market. The results are not included in the thesis.

---

## Configuration

Each strategy folder contains `CSV/settings.csv` with all tunable parameters:

| Parameter | Description |
|---|---|
| `p_value_threshold` | Max p-value for Engle-Granger cointegration test |
| `min_correlation` | Minimum Pearson correlation between log-price series |
| `min_hurst` / `max_hurst` | Hurst exponent bounds |
| `min_half_life` / `max_half_life` | Mean-reversion half-life bounds (days) |
| `min_zero_crossings` | Minimum spread zero-crossings in IS window |
| `sharpe_ratio_threshold` | Minimum IS Sharpe ratio |
| `z_window` | Rolling window for Z-score (Strategies II and III) |
| `z_entry` / `z_exit` | Z-score entry and exit thresholds |
| `hl_multiplier` | Time-based exit: close after `half_life * hl_multiplier` days |
| `initial_capital` | Starting portfolio capital |
| `max_open_pairs` | Maximum simultaneous open positions |
| `commission_pct` | One-way commission as fraction of trade value |
| `vix_z_threshold` | VIX Z-score threshold for blocking entries (SIRIUS only) |

---

## Requirements

```
python >= 3.9
pandas
numpy
statsmodels
yfinance
python-dateutil
openpyxl
```

```bash
pip install pandas numpy statsmodels yfinance python-dateutil openpyxl
```

---

## Data Sources

- S&P 500 price and volume data: Yahoo Finance via `yfinance`
- GICS sector classification: S&P Global
- Historical S&P 500 constituents: fja05680 (2026). *S&P 500 Historical Components & Changes.* https://github.com/fja05680/sp500
- VIX daily data: CBOE (included in `CSV/VIX_History.csv`)
