# IV Surface — Parameterizations and Checks

## SVI (Stochastic Volatility Inspired)

Gatheral 2004. Raw parameterization for total variance `w(k) = T·σ²(k)`:

```
w(k) = a + b · [ρ·(k − m) + √((k − m)² + σ²)]
```

Where `k = log(K/F)`, F = forward = S·e^((r−q)T).

**Parameter constraints (no-arb sufficient conditions, Gatheral-Jacquier 2014):**
- b ≥ 0
- |ρ| ≤ 1
- σ > 0
- a + b·σ·√(1−ρ²) ≥ 0

**Python fit:**

```python
import numpy as np
from scipy.optimize import least_squares

def svi(k, a, b, rho, m, sigma):
    return a + b*(rho*(k-m) + np.sqrt((k-m)**2 + sigma**2))

def fit_svi(k_arr, w_arr):
    def resid(p):
        a, b, rho, m, sigma = p
        return svi(k_arr, a, b, rho, m, sigma) - w_arr
    p0 = [0.02, 0.1, -0.5, 0.0, 0.1]
    bounds = ([-np.inf, 0,    -0.999, -np.inf, 1e-6],
              [ np.inf, np.inf, 0.999,  np.inf, np.inf])
    res = least_squares(resid, p0, bounds=bounds, max_nfev=2000)
    return res.x, res.cost
```

## SABR (Hagan 2002 approximation)

Four parameters (α, β, ρ, ν). For β ≠ 1:

```
σ_BS(K, F, T) ≈ (α / [(FK)^((1-β)/2) · (1 + ((1-β)²/24)·ln²(F/K) + ((1-β)⁴/1920)·ln⁴(F/K))])
              · (z / x(z))
              · (1 + [((1-β)²α²)/(24(FK)^(1-β)) + (ρβνα)/(4(FK)^((1-β)/2)) + ν²(2-3ρ²)/24] · T)
where:
  z = (ν/α) · (FK)^((1-β)/2) · ln(F/K)
  x(z) = ln([√(1-2ρz+z²) + z − ρ] / (1−ρ))
```

For ATM (K = F), z → 0; use the simplified ATM SABR:
```
σ_ATM ≈ (α / F^(1-β)) · (1 + [((1-β)²α²)/(24·F^(2(1-β))) + (ρβνα)/(4·F^(1-β)) + ν²(2-3ρ²)/24] · T)
```

β is typically pinned: β = 1 (lognormal, equities), β = 0.5 (normal-ish, rates), β = 0 (normal, some rates). Fit (α, ρ, ν).

## Newton-Raphson Edge Cases

1. **Vega ≈ 0** (deep OTM): switch to bisection in [1e-4, 5.0].
2. **Initial guess too far**: try σ₀ = √(2π/T) · |price/S| as a Brenner-Subrahmanyam approximation.
3. **Bid = ask**: NR converges but the result has no meaning; flag as "stale".
4. **Negative time value**: the option is below intrinsic — exchange data error. Drop.

## Arbitrage Tests

### Calendar (Carr-Madan condition)
```python
def calendar_arbitrage(surface):
    # surface = list of (T, [(K, iv), ...]) sorted by T
    violations = []
    for i in range(len(surface) - 1):
        T1, smile1 = surface[i]
        T2, smile2 = surface[i+1]
        K_common = set(k for k, _ in smile1) & set(k for k, _ in smile2)
        for K in K_common:
            iv1 = dict(smile1)[K]
            iv2 = dict(smile2)[K]
            if T1 * iv1**2 > T2 * iv2**2:
                violations.append((K, T1, T2, T1*iv1**2 - T2*iv2**2))
    return violations
```

### Butterfly (positive density)
```python
def butterfly_arbitrage(strikes, call_prices):
    # strikes sorted ascending
    violations = []
    for i in range(1, len(strikes) - 1):
        # second derivative approximation
        d2C = call_prices[i+1] - 2*call_prices[i] + call_prices[i-1]
        if d2C < -1e-6:
            violations.append((strikes[i], d2C))
    return violations
```

### Wing slope (Roger Lee)
Total variance asymptotics must satisfy `|dw/dk| ≤ 2` for large |k|, else the second moment of returns blows up. In SVI: `|b·(1 ± ρ)| ≤ 2`.

## When to Prefer SVI vs SABR

| Use SVI when | Use SABR when |
|---|---|
| Pricing index options (SPX, NDX, RUT) | Pricing FX or single-name equity |
| Surface has heavy left-skew | Surface is smile-shaped (more symmetric) |
| You need calendar-arb-free across expiries (use SSVI) | You need control over β-prescribed dynamics |
| Forward smile / VIX-style calculations | Calibrating to swaption / cap vol |
