---
name: delta-hedging
description: >
  Design a delta-hedging schedule for a long-vol or short-vol options position — discrete rebalance
  bands, expected gamma-scalp P&L, transaction-cost drag, and realized-vol breakeven. Use this skill
  when the user holds options and wants to manage directional exposure dynamically. Triggers:
  "delta hedge", "gamma scalp", "rebalance band", "realized vol breakeven", "how often should I hedge",
  "DvegaDtime", "gamma trading P&L", "is my long-vol position profitable".
  Defaults: hedge in shares, threshold |Δ| = 0.05 per 100 contracts, broker cost $0.005/share.
---

# Delta Hedging Schedule

Design and simulate a delta-hedging strategy for an option position. Decompose realized P&L into gamma scalp + vega + theta + transaction costs.

---

## Step 1: Define the Position

Required inputs:
- Underlying ticker (for live spot)
- Each leg: type, strike, expiry, quantity (signed), entry price
- Current spot, current σ implied (per leg)
- Realized-σ forecast (for expected P&L; can default to trailing 20-day realized)
- Transaction cost per share (commission + half-spread). Default: $0.005/share.

---

## Step 2: Compute Position Greeks at Current Spot

Use `greeks-calculator` for the per-leg Greeks. Net them:
```
Δ_net = Σ (sign · qty · 100 · Δ_leg)
Γ_net = Σ (sign · qty · 100 · Γ_leg)   # raw gamma per $1 of spot
ν_net = Σ (sign · qty · 100 · ν_leg)   # per vol point
Θ_net = Σ (sign · qty · 100 · Θ_leg)   # per calendar day
```

Δ_net is the share-equivalent. To get flat, **short** Δ_net shares (or buy −Δ_net if negative).

---

## Step 3: Rebalance Band Strategy

Discrete delta-hedging incurs gamma scalping P&L PLUS transaction cost drag. The classic band is:

```
Δ_band = α · σ · S · √(T_period / 252)
```

where α is a tuning parameter (commonly α = 1, conservative). Rebalance whenever |Δ_net| exceeds the band. Smaller band → more scalp P&L AND more cost.

Practical defaults:
- α = 1.0 (tight)
- T_period = 1 day → band ≈ daily vol move at α=1
- Re-check at market open and every 30 min thereafter

---

## Step 4: Expected Gamma Scalp P&L

Realized variance pays gamma; implied variance is what you paid for it. Per share of underlying, per day:

```
expected_P&L_per_day = 0.5 · Γ · S² · (σ_realized² − σ_implied²) · (1/252)
```

Scaled by position size and re-evaluated daily.

If σ_realized > σ_implied → long-gamma trades profit. If σ_realized < σ_implied → short-gamma trades profit.

---

## Step 5: Realized-Vol Breakeven

The realized vol at which gamma scalp P&L exactly offsets theta:

```
σ_RV_breakeven = √( σ_implied² − (Θ_net / (0.5 · Γ_net · S²)) · 252 )
```

Long-vol holders: realized vol must exceed σ_RV_breakeven to break even on the day.
Short-vol holders: realized vol must stay BELOW it.

If `(Θ_net / (0.5 · Γ_net · S²))` makes the radical negative, the breakeven is undefined — usually means a position with both signs of gamma across legs. Decompose by leg.

---

## Step 6: Transaction-Cost Drag

For a long-gamma position with band α·σ·S·√(dt), each rebalance trades ~Δ_band shares. Daily expected cost:

```
N_hedges_per_day ≈ 1 / α²      # for Brownian motion
daily_TC = N_hedges · Δ_band · cost_per_share
```

This is rough — empirically validated with simulation. The narrower the band (smaller α), the more N_hedges grows, and the cost-of-trading ate the scalp P&L fast.

---

## Step 7: Simulate (optional)

For more accuracy, run a Monte Carlo or use historical underlying paths to simulate the band strategy and report distribution of daily P&L.

```python
import numpy as np
def sim_delta_hedge(S0, sigma_realized, T_days, alpha, gamma0, theta_per_day, cost_per_share, n_paths=1000):
    dt = 1/252
    pnls = []
    for _ in range(n_paths):
        S = S0
        pnl = 0.0
        for d in range(T_days):
            # underlying move
            ret = np.random.normal(0, sigma_realized*np.sqrt(dt))
            S_new = S * np.exp(ret)
            # gamma scalp (continuous approximation)
            pnl += 0.5 * gamma0 * (S_new - S)**2
            # theta
            pnl += theta_per_day
            # rebalance whenever |delta| exceeded band — assume on average alpha rebalances per day
            band_size = alpha * sigma_realized * S * np.sqrt(dt)
            pnl -= band_size * cost_per_share / alpha   # rough
            S = S_new
        pnls.append(pnl)
    return np.array(pnls)
```

This simulation is a first approximation — real Greeks change with spot and time, so for a 30+ DTE position re-compute Greeks daily.

---

## Step 8: Respond to User

1. Position Greeks summary (Δ, Γ, ν, Θ at current spot).
2. Rebalance band size in shares.
3. Expected daily P&L decomposition: gamma scalp + theta + TC drag.
4. Realized-vol breakeven.
5. If σ_implied > 20-day realized AND position is short-gamma → flag risk of vol expansion.

For sizing the position, hand off to `position-sizing`. For per-leg Greeks intuition, see `greeks-calculator`.
