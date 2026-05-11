---
name: portfolio-greeks
description: >
  Aggregate Greeks across a multi-position options book and surface concentration risks. Use this skill
  when the user has multiple open options trades and asks "what's my net delta", "am I net long or short
  vol", "what's my book theta", "where am I concentrated", or wants a single-pane view of book-level risk.
  Triggers: "portfolio greeks", "book greeks", "net delta", "net vega", "concentration risk",
  "what if X moves 5%", "stress test my options book".
  Use when the user provides multiple positions; do NOT use for single-trade Greeks (use `greeks-calculator`).
---

# Portfolio Greeks Aggregator

Sum Greeks across an options book and flag concentration risks.

---

## Step 1: Gather All Positions

Required per position:
- Underlying ticker
- Each leg: type, strike, expiry, signed quantity, fill price
- Current spot per underlying (live)
- Current implied vol per leg (live or last known)

If positions come from a broker screenshot, parse via `options-chain-reader` first.

---

## Step 2: Compute Per-Position Greeks

For each position (potentially multi-leg), use `greeks-calculator` to get position-level Δ, Γ, ν, Θ.

---

## Step 3: Aggregate by Underlying

Group all positions by underlying ticker. Within each group, sum the dollar Greeks:

```
Δ_$    = Σ Δ_position                          # share equivalent
Γ_$    = Σ Γ_position · spot²/100              # P&L per 1% spot move squared
ν_$    = Σ vega_per_volpt_position             # P&L per 1.00 vol point
Θ_$    = Σ theta_per_day_position              # daily decay
```

**Why dollar Greeks**: comparing 1 contract on TSLA at $300 to 1 contract on SPY at $580 is misleading — dollar Greeks normalize.

---

## Step 4: Cross-Underlying View

Sum across all underlyings to get book totals. Caveat: book-level vega across different tickers is NOT directly fungible — TSLA vega and SPX vega don't move together. Display per-underlying first, then book total with a label.

---

## Step 5: Concentration Risk Checks

| Check | Threshold |
|---|---|
| Any single underlying > 25% of book BP | Flag |
| Net long vega + net short gamma > book equity | Flag — toxic combo |
| All positions on the same sector | Flag |
| One expiry > 50% of total theta | Flag — concentrated time risk |
| Net delta > 50% of account equity in $ | Flag — directional exposure dominates the book |

---

## Step 6: Stress Tests

Run these scenarios and report book P&L for each (using current Greeks, linear approximation):

| Scenario | Spot move | Vol move |
|---|---|---|
| Mild rally | +2% | −5% IV |
| Crash | −5% | +30% IV |
| Slow drift down | −1% | flat |
| Volatility spike, no spot move | 0% | +50% IV |
| Pin (close to ATM, near expiry) | 0% | −20% IV |

Linear P&L approximation:
```
ΔPnL ≈ Δ_$ · spot_move + 0.5 · Γ_$ · (spot_move%)² · 100 + ν_$ · iv_move_points + Θ_$ · days
```

For large moves (> 5%) the linear approx breaks down — note that and recommend repricing each leg at the stressed scenario.

---

## Step 7: Respond to User

1. Per-underlying table: dollar Greeks + position count.
2. Book totals (with caveat on cross-ticker vega).
3. Concentration flags (if any).
4. Stress-test table.
5. One-line summary: "Book is net long vol ($X/vol pt), net short gamma at the near expiry, with TSLA at 38% of BP — biggest single risk is a TSLA gap."

For hedging recommendations, recommend `delta-hedging`.

For trimming a concentrated position, recommend `position-sizing` on the offsetting trade.
