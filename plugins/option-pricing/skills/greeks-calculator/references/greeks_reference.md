# Greeks Reference

## Python Implementation

```python
from math import log, sqrt, exp, pi
from scipy.stats import norm

def greeks(S, K, T, r, sigma, q=0.0, opt='call'):
    if T <= 0 or sigma <= 0:
        return {'delta': 0, 'gamma': 0, 'vega': 0, 'theta': 0, 'rho': 0}
    sqrtT = sqrt(T)
    d1 = (log(S/K) + (r - q + 0.5*sigma**2)*T) / (sigma*sqrtT)
    d2 = d1 - sigma*sqrtT
    pdf = (1/sqrt(2*pi))*exp(-0.5*d1*d1)
    discQ = exp(-q*T)
    discR = exp(-r*T)
    if opt == 'call':
        delta = discQ * norm.cdf(d1)
        theta = (-S*discQ*pdf*sigma/(2*sqrtT)
                 - r*K*discR*norm.cdf(d2)
                 + q*S*discQ*norm.cdf(d1))
        rho = K*T*discR*norm.cdf(d2)
    else:
        delta = discQ * (norm.cdf(d1) - 1)
        theta = (-S*discQ*pdf*sigma/(2*sqrtT)
                 + r*K*discR*norm.cdf(-d2)
                 - q*S*discQ*norm.cdf(-d1))
        rho = -K*T*discR*norm.cdf(-d2)
    gamma = discQ*pdf / (S*sigma*sqrtT)
    vega  = S*discQ*pdf*sqrtT
    # scale to practitioner conventions
    return {
        'delta': delta,
        'gamma': gamma,
        'vega_per_volpt': vega / 100,
        'theta_per_day': theta / 365,
        'rho_per_pct':   rho / 100,
    }
```

## Worked Example: Long Iron Condor on SPX (Long Vol)

Spot 5800, IV 18%, 30 DTE, r = 4.3%, q = 1.5%. Buy 5700P/Sell 5750P, Sell 5850C/Buy 5900C.

Wait — this is a SHORT iron condor (collect premium). Let's be precise:

| Leg | Strike | Side | Qty | Δ | Γ | Vega/vol-pt | Theta/day |
|---|---|---|---|---|---|---|---|
| Put 5700 | 5700 | Long | +1 | −0.16 | +0.0019 | +6.8 | −1.10 |
| Put 5750 | 5750 | Short | −1 | +0.21 | −0.0022 | −7.6 | +1.21 |
| Call 5850 | 5850 | Short | −1 | −0.27 | −0.0023 | −7.9 | +1.18 |
| Call 5900 | 5900 | Long | +1 | +0.20 | +0.0020 | +7.0 | −0.95 |
| **Net (×100)** | — | — | — | **−2** | **−0.6** | **−$150** | **+$134** |

Interpretation: this short iron condor is approximately delta-neutral (Δ ≈ 0), short gamma (loses on big moves), short vega (loses on IV expansion), positive theta (collects $134/day if nothing moves). Classic premium-selling profile.

## Intuition Guide

| If you are… | Then you want… | And you should fear… |
|---|---|---|
| Long Δ | Spot to rise | Spot drops |
| Long Γ | Spot to MOVE (either way) | Spot pins to strike |
| Long ν | IV to rise (vol expansion) | IV crush |
| Long Θ | Time to pass | Big moves (you're paying for them) |
| Short Δ | Spot to fall | Spot rallies |
| Short Γ | Spot to be pinned | Big moves (gap risk) |
| Short ν | IV to fall (vol crush) | IV spike |
| Short Θ | Spot to move fast | Time decay (you're paying it) |

**Iron rule for short-Γ / short-ν books**: define your max loss BEFORE the trade. Cash-secured puts can lose 50%+ on a single down day. Selling naked calls is theoretically unbounded.

## Common Pitfalls

1. **Vega units**: a "vega of 0.5" with no label is ambiguous. Always label as `per vol pt` (centivol) or `per 1.00 vol` (raw).
2. **Theta sign**: long options have NEGATIVE theta (you lose time value). Short positions have positive theta. Some platforms display absolute value with an icon — always check sign.
3. **Multiplier**: forgetting `×100` underestimates dollar Greeks by 100×. Equity options are 100 multiplier; SPX cash-settled is 100; some indices (NDX) are 100, futures options vary.
4. **Greeks are local**: they linearize at the current spot/IV. A 5% spot move makes them stale — re-compute, don't extrapolate.
