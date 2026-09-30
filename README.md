# quant-stat-arb-pipeline-backtesting
# This repository builds on the classic statistical arbitrage framework from Avellaneda & Lee (2008). 

It takes a standard, working stat-arb pipeline (PCA factor model, Ornstein-Uhlenbeck residual estimation, s-score trading rules) and tests a few independent extensions to see if we can improve the baseline performance. 

## The TL;DR
The biggest performance boost didn't come from fancier math—it came from fixing a couple of oversights in the original implementation (specifically, applying a beta-neutral hedge that was computed but ignored). 

Attempting to "upgrade" the model by using a Kalman filter or GICS sector factors actually **underperformed** the basic PCA + OLS setup over the full sample, though they showed interesting behavior during the 2020 COVID crash.

## What's in the Notebook

The notebook isolates and measures the impact of changes along three different axes:

1. **Pipeline Corrections**: 
   - **Alpha fix**: The original modified s-score formula silently dropped the alpha term (defaulting to 0). This is now wired up correctly.
   - **Beta-neutral hedge**: The original code computed beta-neutral weights but never actually passed them to the backtester. Applying this hedge properly bumped the Sharpe ratio from 0.68 to 0.85.

2. **Sector Factors vs. PCA**: 
   Swapped out the 15 PCA components for 10 GICS sector factors ("synthetic ETFs"). 
   *Result*: PCA explains more variance in-sample (76% vs 62%) and performs better overall, but sector factors handled the 2020 dislocation much better.

3. **Kalman Filter vs. Static OLS**: 
   Replaced the static 60-day OLS fit with a time-varying Kalman filter to estimate the O-U parameters ($\kappa$, $m$, $\sigma$).
   *Result*: The Kalman filter tracks within-window instability better, but smoothing the noise slowed down the strategy's reaction to genuine regime changes. It underperformed the OLS baseline.

4. **Combined Extensions**: 
   Tested Sector factors + Kalman O-U together. 
   *Result*: The worst of both worlds (Sharpe 0.22).

5. **Validation & Walk-Forward**:
   - Broken down by calendar regimes and volatility regimes.
   - Ran a strict out-of-sample (OOS) walk-forward selection for the entry threshold ($s_{bo}$) to ensure we aren't curve-fitting the full-sample data.

## Results Summary (Full Sample)

| Variant | Sharpe Ratio | Max Drawdown |
| :--- | :--- | :--- |
| **Baseline (PCA + OLS, beta-neutral)** | **0.85** | **-7.33%** |
| Walk-forward OOS (beta-neutral) | 0.88 | -10.50% |
| PCA + Kalman O-U | 0.63 | -9.98% |
| Sector factors + OLS | 0.45 | -11.19% |
| Combined (Sector + Kalman) | 0.22 | -14.81% |

*Takeaway: Stick to the PCA/OLS baseline, but make sure your hedges are actually applied out of sample.*

## How to Run

Just run the Jupyter notebook. 

To save time, the code caches intermediate backtest results in a local `checkpoints/` directory as `.pkl` files. A full run from scratch on the 50-stock, 10-year universe takes about 6-7 minutes. If you change the underlying logic or want to force a clean run, just delete the `checkpoints/` folder.
