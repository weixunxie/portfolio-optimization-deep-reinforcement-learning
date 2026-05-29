# Portfolio Optimization with Deep Reinforcement Learning

> **Original implementation — PPO-based portfolio allocation for Chinese pharmaceutical and biotechnology equities**

---

## Relationship to Extended Version

This repository preserves the original PPO portfolio optimization implementation. A newer optimized and extended version with regime-aware analysis and improved experiment organization is available here:

**[weixunxie/regime-aware-portfolio-allocation](https://github.com/weixunxie/regime-aware-portfolio-allocation)**

The newer repository builds on this work with additional features and a cleaner experimental framework. This repository documents the original implementation as a standalone, self-contained study.

---

## Project Overview

This project investigates portfolio allocation strategies in the Chinese pharmaceutical and biotechnology sector by comparing classical optimization methods with a modern deep reinforcement learning approach. The study evaluates four strategies — Equal Weight, Mean–Variance Optimization, Risk Parity, and PPO — on a universe of 13 sector equities over the period 2018–2025.

A custom OpenAI Gymnasium-compatible environment simulates dynamic portfolio rebalancing with transaction costs, and a PPO agent trained via Stable-Baselines3 learns adaptive asset weight allocations under non-stationary market conditions.

---

## Research Question

Can an adaptive PPO-based portfolio allocation strategy achieve better risk-adjusted performance or greater robustness compared to static classical baselines — Equal Weight, Sharpe-maximizing Mean–Variance, and Risk Parity — in a volatile sector-specific Chinese equity universe?

---

## Motivation

Classical portfolio optimization methods (mean–variance, risk parity) rely on static historical estimates of returns and covariances. These estimates can be unstable in sector-specific or emerging markets, where structural breaks are common. Reinforcement learning offers an alternative: rather than fitting a static model to historical moments, an RL agent learns allocation behavior through repeated interaction with a simulated trading environment, potentially adapting to changing market dynamics without explicit re-estimation.

---

## Data

| Field | Detail |
|---|---|
| Universe | 13 Chinese pharmaceutical and biotechnology equities |
| Tickers (display names) | Aier, Berry, BGI, Zhifei, Sanjiu, E-Jiao, Hengrui, Fosun, Mindray, Vcanbio, AppTec, Yiling, Pientzehuang |
| Frequency | Daily returns |
| Sample period | October 2018 – 2025 |
| Source file | `data/Return.xlsx` |

Returns are pre-computed daily percentage changes stored in `Return.xlsx`. The data covers a period that includes the 2020–2021 biotech boom and subsequent multi-year drawdown in Chinese healthcare equities.

---

## Methods

### 1. Equal Weight (EW)

A naive baseline allocating 1/N weight to each of the N assets at all times. No estimation required. Provides a passive diversification benchmark.

### 2. Mean–Variance Optimization — Sharpe Ratio Maximization

Maximizes the portfolio Sharpe ratio under a classical mean–variance framework. Historical mean returns and covariance are estimated from the full sample. Implemented via [Riskfolio-Lib](https://riskfolio-lib.readthedocs.io/).

### 3. Risk Parity — Equal Risk Contribution (ERC)

Allocates weights so that each asset contributes equally to total portfolio variance. Compared to mean–variance optimization, risk parity is less sensitive to return estimates and tends to produce more diversified allocations. Implemented via Riskfolio-Lib.

### 4. Deep Reinforcement Learning — PPO

A Proximal Policy Optimization agent trained in a custom portfolio environment. The agent learns a mapping from a recent window of return observations to portfolio weights, without requiring explicit return or covariance estimation.

---

## Custom Reinforcement Learning Environment

The `PortfolioEnv` class (`notebooks/ppo_portfolio_optimization.ipynb`) implements a Gymnasium-compatible environment:

| Component | Description |
|---|---|
| **State** | Rolling window of daily returns — shape `(window_size=5, n_assets=13)` |
| **Action space** | Continuous weight vector over 13 assets, clipped to [0, 1] and normalised to sum to 1 (long-only constraint) |
| **Reward** | `log(1 + portfolio_return) − transaction_cost − weight_change_penalty` |
| **Transaction cost** | 0.2% × absolute weight change per asset per step |
| **Rebalancing penalty** | 0.5% × total weight change (discourages excessive turnover) |
| **Episode length** | Full data length (single-pass backtest) |

The log-return reward stabilises training under compounding and the combined transaction cost / penalty term encourages the agent to adopt a lower-turnover strategy.

---

## Evaluation Metrics

The following metrics are computed in the notebooks:

- Cumulative return
- Sharpe ratio (annualised, risk-free rate = 0)
- Sortino ratio
- Individual and portfolio-level 2-year and 5-year returns
- Portfolio weight composition and risk contribution per asset

---

## Repository Structure

```
portfolio-optimization-deep-reinforcement-learning/
├── data/
│   └── Return.xlsx                          # Daily returns, 13 stocks, 2018–2025
├── docs/
│   └── thesis_weixun_xie.pdf                # Supporting thesis document
├── notebooks/
│   ├── ppo_portfolio_optimization.ipynb     # PPO agent: environment, training, evaluation
│   └── classical_portfolio_methods.ipynb    # EW, MV (Sharpe), Risk Parity analysis
├── results/
│   └── figures/
│       ├── ppo_cumulative_portfolio_value.png
│       ├── ew_cumulative_return_2yr.png
│       ├── ew_cumulative_return_5yr.png
│       ├── sr_cumulative_return_2yr.png
│       ├── sr_cumulative_return_5yr.png
│       ├── rp_cumulative_return_2yr.png
│       ├── rp_cumulative_return_5yr.png
│       ├── sharpe_mv_composition.png
│       ├── rp_portfolio_composition.png
│       ├── rp_variance_composition.png
│       ├── sr_risk_composition.png
│       └── comparison_asset_weights.png
├── README.md
├── requirements.txt
└── .gitignore
```

---

## Key Outputs

All pre-computed figures are in `results/figures/`:

| Figure | Description |
|---|---|
| `ppo_cumulative_portfolio_value.png` | PPO agent cumulative portfolio value over the backtest period |
| `ew_cumulative_return_2yr/5yr.png` | Equal Weight portfolio cumulative return — 2yr and 5yr windows |
| `sr_cumulative_return_2yr/5yr.png` | Sharpe MV portfolio cumulative return — 2yr and 5yr windows |
| `rp_cumulative_return_2yr/5yr.png` | Risk Parity portfolio cumulative return — 2yr and 5yr windows |
| `sharpe_mv_composition.png` | MV (Sharpe) portfolio weight allocation by asset |
| `rp_portfolio_composition.png` | Risk Parity weight allocation by asset |
| `rp_variance_composition.png` | Risk contribution breakdown for the Risk Parity portfolio |
| `sr_risk_composition.png` | Risk contribution breakdown for the MV (Sharpe) portfolio |
| `comparison_asset_weights.png` | Side-by-side weight comparison across multiple risk-measure-optimal portfolios |

---

## How to Reproduce

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

> Riskfolio-Lib and Stable-Baselines3 may require additional system dependencies (C++ compiler, BLAS). See their respective installation guides if `pip install` fails.

### 2. Run the classical portfolio analysis

```bash
jupyter notebook notebooks/classical_portfolio_methods.ipynb
```

Covers Mean–Variance (Sharpe), Risk Parity, multi-measure optimization, and individual asset return summaries.

### 3. Run the PPO training and evaluation

```bash
jupyter notebook notebooks/ppo_portfolio_optimization.ipynb
```

Builds the custom Gymnasium environment, trains the PPO agent for 20,000 timesteps, runs a backtest, and reports Sharpe and Sortino ratios. The trained model is saved to `results/models/ppo_portfolio_model.zip`.

> **Note on reproducibility:** PPO training involves stochastic optimization. Results may vary slightly across runs due to random initialization. To reproduce a specific result exactly, set a fixed random seed before calling `model.learn()`.

---

## Requirements

```
numpy
pandas
scipy
matplotlib
seaborn
riskfolio-lib
gymnasium
stable-baselines3[extra]
torch
jupyter
openpyxl
```

See `requirements.txt` for the full list.

---

## Limitations

- **Backtest-only:** All results are in-sample or rolling backtests on a single asset universe and sample period. Out-of-sample performance is not evaluated in this repository.
- **Sample-period sensitivity:** The 2018–2025 window for Chinese healthcare equities includes an unusually strong sector drawdown. Results may not generalize to other periods or sectors.
- **Transaction cost assumptions:** A flat 0.2% per-trade cost is used. Real execution costs vary by market conditions, position size, and broker.
- **PPO sensitivity:** Agent performance depends on reward design, random seed, and hyperparameter choices. The default 20,000 timestep training run is lightweight and may underfit.
- **No live trading:** This project is research-oriented and does not constitute a deployable trading system or investment recommendation.
- **Experiment organization:** This is the original implementation. The [extended version](https://github.com/weixunxie/regime-aware-portfolio-allocation) has improved experiment structure and additional analysis.

---

## Portfolio Notes

This repository represents an original research implementation comparing classical and reinforcement learning portfolio strategies on a sector-specific equity universe. Key contributions:

- A custom Gymnasium-compatible portfolio environment with transaction costs and rebalancing penalties
- A direct comparison of PPO against Equal Weight, Mean–Variance (Sharpe), and Risk Parity on the same dataset
- A foundation that was later extended into the [regime-aware portfolio allocation project](https://github.com/weixunxie/regime-aware-portfolio-allocation)

The project is presented as a research artifact and portfolio demonstration, not as a production investment system.

---

## Author

Weixun Xie (Stephanie Xie)  
M.A. Statistics — Columbia University
