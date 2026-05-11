---
name: iv-surface
description: >
  Fit and visualize the implied volatility surface for a ticker — IV across strike (skew) and time
  (term structure). Use this skill when the user asks for an IV surface, vol surface, mispriced
  options across the chain, butterfly arbitrage check, or vol-of-vol context. Triggers: "IV surface",
  "vol surface", "show me the smile", "find dislocated options", "is the surface arb-free",
  "fit SVI / SABR", "surface for AAPL/SPY/QQQ", "scan the chain for cheap vol".
  Use even with partial input — defaults: nearest 6 expiries, ±20% strike range from spot.
---

# IV Surface Fitting and Visualization

Build the full IV surface (strike × time → IV) for a ticker, fit a smooth parameterization (SVI or SABR), and flag arbitrage violations.

---

## Step 1: Pull the Options Chain

Use the `yfinance-options` skill to pull ALL expiries (or top 6 by liquidity) and BOTH puts and calls.

For each contract: store strike K, expiry T (years), bid, ask, last, mid, IV (from chain), open interest, volume.

**Filters before fitting:**
- Drop contracts with bid = 0 or ask = 0 (stale/no quotes).
- Drop contracts with bid-ask spread > 30% of mid (illiquid).
- Drop contracts with mid < $0.05 (rounding noise dominates).

If after filtering you have < 6 contracts per expiry, **stop** and report the chain is too thin to fit.

---

## Step 2: Compute Mid-Market IV

If the chain IV is missing or stale, back out IV from mid price using Newton-Raphson on Black-Scholes:

```python
def implied_vol(price, S, K, T, r, opt_type='call', q=0.0, tol=1e-6, max_iter=50):
    sigma = 0.3  # initial guess
    for _ in range(max_iter):
        from math import log, sqrt, exp, pi
        from scipy.stats import norm
        d1 = (log(S/K) + (r - q + 0.5*sigma**2)*T) / (sigma*sqrt(T))
        d2 = d1 - sigma*sqrt(T)
        if opt_type == 'call':
            price_model = S*exp(-q*T)*norm.cdf(d1) - K*exp(-r*T)*norm.cdf(d2)
        else:
            price_model = K*exp(-r*T)*norm.cdf(-d2) - S*exp(-q*T)*norm.cdf(-d1)
        vega = S*exp(-q*T)*(1/sqrt(2*pi))*exp(-0.5*d1*d1)*sqrt(T)
        if vega < 1e-10:
            return None
        diff = price_model - price
        if abs(diff) < tol:
            return sigma
        sigma -= diff / vega
        if sigma <= 0:
            sigma = 0.01
    return None
```

If Newton-Raphson fails (returns None or oscillates) on > 20% of contracts, fall back to bisection. If still failing, **report which strikes failed** — do not silently drop them.

---

## Step 3: Fit a Parameterization (per expiry)

For each expiry T, fit one of:

### SVI (Stochastic Volatility Inspired) — recommended for index options
Five parameters (a, b, ρ, m, σ) describe total variance w(k) where k = log(K/F):
```
w(k) = a + b · [ρ·(k − m) + √((k − m)² + σ²)]
```
- Wings: w(k) → b·(1±ρ)·|k| as |k| → ∞
- ATM variance: w(0) = a + b·(σ + ρ·(−m)·... — see `references/iv_reference.md` for full formulas)

### SABR — recommended for equity / FX
Four parameters (α, β, ρ, ν). Hagan 2002 approximation gives IV(K, F, T).

Fit with `scipy.optimize.least_squares` on `(IV_market − IV_model)`. Constrain: b > 0, σ > 0, |ρ| < 1 (SVI); α, ν > 0, |ρ| < 1, β ∈ [0, 1] (SABR).

---

## Step 4: Arbitrage Checks (REQUIRED — never skip)

1. **Calendar arbitrage**: For fixed K, total variance T·σ²(K, T) must be NON-DECREASING in T.
2. **Butterfly arbitrage**: For fixed T, the second derivative of call price w.r.t. K must be ≥ 0 (Breeden-Litzenberger density). Failures show negative implied probabilities.
3. **Wing slope**: |dw/dk| ≤ 2 as |k| → ∞ (Roger Lee moment formula).

If any check fails, list the violating (K, T) pairs in the response. Do not silently smooth them away.

---

## Step 5: Visualize

Two charts:
1. **Surface heatmap**: x = log-moneyness k = log(K/F), y = T (years), color = IV.
2. **Per-expiry smile**: K on x-axis, IV on y-axis, one line per expiry.

For Claude.ai widgets, use `show_widget` with `["chart"]` modules and plot via Chart.js scatter.

---

## Step 6: Find Dislocations

After the fit, compute the residual `IV_market − IV_fitted` for each contract. Report:
- Top 5 "cheap" (most negative residual, market IV below fit)
- Top 5 "rich" (most positive residual)

Caveat: residuals can be wide near zero-bid contracts or right after a jump — report the liquidity (OI, volume, spread) alongside each, so the user can judge whether the dislocation is tradeable.

---

## Step 7: Respond to User

1. Number of contracts used in the fit (after filtering).
2. Per-expiry parameters table and one-line fit-quality metric (RMSE in vol points).
3. Arbitrage violations (if any).
4. The two charts.
5. Top dislocations table.

For trading off the dislocation, recommend `options-payoff` to size and visualize the structure. For pure skew interpretation, recommend `vol-skew`.

---

## Reference Files

- `references/iv_reference.md` — Full SVI and SABR derivations, Newton-Raphson edge cases, arbitrage-test code
